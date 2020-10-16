---
title: "OpenCTF 2018 - unknown Writeup"
slug: "openctf-2018-unknown-writeup"
date: "2018-08-14 23:39:26.915487"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

by Javantea

Aug 12, 2018

unknown is an easy network programming challenge.

The hint only contained an IP address and port, 172.31.2.59 51966. When you connected, it would ask you to decode a base64 string. When you responded with the answer within a second, it would respond with a morse code string. After the morse, it would respond with a binary string. After the binary it would respond with a hex string with spaces between each byte. After the hex string, it would respond with a base64 string and so on. Since the formats were constant while the strings were variable, I was able to hardcode them, but because I didn't want any surprises, I wrote a quick function called "solve" that would detect each type and decode. Running this in a loop, I got the solution to this challenge. Instead of 12 solutions, the challenge required between 64 and 76 solutions. But since they were just a few codecs, it was pretty easy. The original code was written by my teammate forte. The code quality of this solution is far lower than most of the code I write because I don't write Python 2 code anymore. There is little to no documentation.

[interact.py](https://www.altsci.com/concepts/images/interact.py)
