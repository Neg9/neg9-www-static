---
title: "TAMUctf 2021 Unzip Solution"
slug: "tamuctf-2021-unzip-writeup"
date: "2021-04-26 12:02:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[Unzip](https://ctftime.org/task/15809) - 100 points

>    Hey, can you unzip this for me? [chall.zip](https://shell.tamuctf.com/static/1684c20ce801117c39547386f30bb7f8/chall.zip)

Step 1: Convert the zip file to a file that [John](https://www.openwall.com/john/) can crack.

Note that this is a pretty standard tool...

```sh

zip2john ~/Downloads/chall.zip >~/altsci/tamuctf/chall.txt

```

chall.txt:
```
chall.zip/flag.txt:$pkzip2$1*2*2*0*30*24*75c0f8c7*0*42*0*30*75c0*b004*e980ad8b1ffd804291d329b24794613bf3484fa6292fd97a57836440dfce9ce753a89d0ad9a8b16b042ecee459ed1274*$/pkzip2$:flag.txt:chall.zip::/home/jvoss/Downloads/chall.zip
```

Step 2: Crack the password.

```sh
john --format=raw-sha256 --rules --wordlist=crack/ai3words_order.txt ~/altsci/tamuctf/chall.txt
```

John cracks it pretty quickly with a simple wordlist but I chose to use the AI3 wordlist which you can download with [my DNSSEC research](https://www.altsci.com/concepts/enumerating-dnssec-nsec-and-nsec3-records).

The password is hunter2, a common IRC joke.

```sh
unzip chall.zip 
Archive:  chall.zip
[chall.zip] flag.txt password: 
cat flag.txt 
gigem{d0esnt_looK_lik3_5t4rs_t0_M3}

```

The flag is `gigem{d0esnt_looK_lik3_5t4rs_t0_M3}`

