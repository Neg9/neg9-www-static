---
title: "yvrCTF Kicker of Elves Solution"
slug: "kicker-of-elves-solution"
date: "2015-03-22 16:16:18.973362"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

yvrctf, bsides vancouver 2015
[Kicker of Elves](https://github.com/yvrctf/2015/tree/master/kickerofelves)

aka. koe

Get [JavRE](https://www.altsci.com/concepts/javre/).

{language=bash}
~~~~~~~~
python javre/disasm1.py -v koe-118c28da8a435978a957cf3cba03ab20932d94fb >koe1.jav
~~~~~~~~

{language=asm}
~~~~~~~~
; start address is the middle of this instruction by the way.
; no worries, keep forensicing.
80482ff: XOR EBP, EBP ; EBP = 0
8048301: POP ESI ; POP ESI
8048302: MOV ECX, ESP ; ECX = ESP;
8048304: AND ESP, -0x10 ; ESP &= -0x10;
8048307: PUSH EAX
8048308: PUSH ESP
8048309: PUSH EDX
804830a: PUSH DWORD 0x80484d0 ; '\xc3'
804830f: PUSH DWORD 0x8048460 ; 'W1\xffVS\xe8\xc5\xfe\xff\xff\x81\xc3e\x12'
8048314: PUSH ECX
8048315: PUSH ESI
8048316: PUSH DWORD 0x80483fb ; 'L$\x04\x83\xe4\xf0\xffq\xfcU\x89\xe5Q\x83\xec\x14\xc7E\xf4'
804831b: CALL 80482ef <sub_80482ef>
8048320: HLT ;
8048321: NOP
8048323: NOP
~~~~~~~~

Then go down to main.

{language=asm}
~~~~~~~~
; main is in the middle of this instruction. No, really.
80483fa: LEA ECX, [ESP+0x4]
80483fe: AND ESP, -0x10 ; ESP &= -0x10;
8048401: PUSH DWORD [ECX-0x4]
8048404: PUSH EBP
8048405: MOV EBP, ESP ; EBP = ESP;
8048407: PUSH ECX
8048408: SUB ESP, 0x14 ; ESP -= 0x14;
804840b: MOV DWORD [EBP-0xc], 0x0 ;
8048412: JMP 0x8048432 ; goto
8048414: MOV EAX, [EBP-0xc] ; EAX = [EBP-0xc];
8048417: ADD EAX, 0x8049720 ; EAX += 0x8049720;
804841c: MOVZX EAX, BYTE [EAX] ; EAX = EAX[0]
; xor encryption We can do this.
804841f: XOR EAX, 0x41 ; EAX ^= 0x41;
8048422: MOV EDX, EAX ; EDX = EAX;
~~~~~~~~

Now try to decrypt the whole thing with a single-byte key.

{language=python}
~~~~~~~~
#!/usr/bin/env python3
"""
Solution for koe
by Javantea
Mar 16, 2015

in minutes using JavRE.
"""
def xorbin(a, b):
    q = max(len(a), len(b))
    output = b''
    for i in range(q):
        output += bytes([a[i % len(a)] ^ b[i % len(b)]])
    return output

a = open('koe-118c28da8a435978a957cf3cba03ab20932d94fb', 'rb').read()
q = xorbin(a, b'\x41')
open('koe_decrypted.bin','wb').write(q)
~~~~~~~~

{language=bash}
~~~~~~~~
strings -a koe_decrypted.bin |grep flag
EIAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAflag{4nything_is_b3tter_than_that_1train}
~~~~~~~~
