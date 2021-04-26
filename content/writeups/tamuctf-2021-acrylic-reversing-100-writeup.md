---
title: "TAMUctf 2021 Acrylic Solution"
slug: "acrylic-writeup"
date: "2021-04-26 10:07:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[Acrylic](https://ctftime.org/task/15817)

    This is an easy challenge. There is a flag that can be printed out from somewhere in this binary. Only one problem: There's a lot of fake flags as well. [acrylic](https://shell.tamuctf.com/static/ee1752a1e370c26acbe5f3079e51173a/acrylic)

I wanted to use JavRE or another static method to solve this, but the solution that I found with JavRE was false. So this is a dynamic reverse engineering solution.

So the strategy here was to see what happens with memory. Generally there will be a pointer to the string somewhere in memory. Sensible places to put that pointer is a register or the stack.

```
gdb ./acrylic1 
GNU gdb (Gentoo 10.1 vanilla) 10.1
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-pc-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://bugs.gentoo.org/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./acrylic1...
(No debugging symbols found in ./acrylic1)
(gdb) break 0x55555566b8
Function "0x55555566b8" not defined.
Make breakpoint pending on future shared library load? (y or [n]) n
(gdb) break main
Breakpoint 1 at 0x6b1
(gdb) run
Starting program: /home/angr/acrylic1 

Breakpoint 1, 0x00005555554006b1 in main ()
(gdb) si
0x00005555554006b8 in main ()
(gdb) 
0x0000555555400510 in puts@plt ()
(gdb) c
Continuing.
look at the code
[Inferior 1 (process 10038) exited normally]
```

As you can see the program exits without printing the flag. Let's quickly look at the disassembly. If you'd like to follow along, try [JavRE](https://www.altsci.com/concepts/javre/) or the disassembler of your choice.

```asm
get_flag:
     63a: PUSH RBP
     63b: MOV RBP, RSP ; RBP = RSP; 
     63e: MOV DWORD [RBP-0x8], 0x7b ; '{'
     645: MOV DWORD [RBP-0x4], 0x0 ; 
     64c: JMP 0x68c ; goto 
     64e: MOV EAX, [RBP-0x8] ; EAX = [RBP-0x8]; 
     651: ADD EAX, 0x1 ; EAX += 0x1; 
     654: IMUL EAX, [RBP-0x8] ; EAX *= [RBP-0x8]; 
     658: MOV ECX, EAX ; ECX = EAX; 
     65a: MOV EDX, 0x81020409 ; EDX = 0x81020409; 
     65f: MOV EAX, ECX ; EAX = ECX; 
     661: IMUL EDX ; 
     663: LEA EAX, [RDX+RCX]
     666: SAR EAX, 0x6 ; EAX >>= 0x6; 
     669: MOV EDX, EAX ; EDX = EAX; 
     66b: MOV EAX, ECX ; EAX = ECX; 
     66d: SAR EAX, 0x1f ; EAX >>= 0x1f; 
     670: SUB EDX, EAX ; EDX -= EAX; 
     672: MOV EAX, EDX ; EAX = EDX; 
     674: MOV [RBP-0x8], EAX ; [RBP-0x8] = EAX; 
     677: MOV EDX, [RBP-0x8] ; EDX = [RBP-0x8]; 
     67a: MOV EAX, EDX ; EAX = EDX; 
     67c: SHL EAX, 0x7 ; EAX <<= 0x7; 
     67f: SUB EAX, EDX ; EAX -= EDX; 
     681: SUB ECX, EAX ; ECX -= EAX; 
     683: MOV EAX, ECX ; EAX = ECX; 
     685: MOV [RBP-0x8], EAX ; [RBP-0x8] = EAX; 
     688: ADD DWORD [RBP-0x4], 0x1 ; 
     68c: CMP DWORD [RBP-0x4], 0x7e3 ;
     693: JBE 0x64e ; } while(DWORD [RBP-0x4] < 0x7e3)
     695: MOV EAX, [RBP-0x8] ; EAX = [RBP-0x8]; 
     698: CDQE ; 
     69a: SHL RAX, 0x6 ; RAX <<= 0x6; 
     69e: MOV RDX, RAX ; RDX = RAX; 
     6a1: LEA RAX, [RIP+0x200978]  ; 0x201020 " ; b'gigem{cant_use_strings}'"
     6a8: ADD RAX, RDX ; RAX += RDX; 
     6ab: POP RBP ; POP RBP
     6ac: RET
```

Using the information I learned from this, I assumed that the function `get_flag` is worth our time in calling directly. GDB can do this using the `jump` command. Jumping to a random memory location is absolutely fine in software, don't let anyone tell you different. Of course in order to find the right address I had to add on `0x0000555555400000`, which is the **base address** that GDB loaded our executable to. Depending on your version of GDB, you might have a different base address. Where do you find your base address? You can find it by taking your `main` function and subtracting its address `0x6b1` from it.

```
(gdb) run
Starting program: /home/angr/acrylic1 

Breakpoint 1, 0x00005555554006b1 in main ()
(gdb) si
0x00005555554006b8 in main ()
(gdb) jump 0x000055555540063a
Function "0x000055555540063a" not defined.
(gdb) jump *0x000055555540063a
Continuing at 0x55555540063a.

Program received signal SIGSEGV, Segmentation fault.
0x0000000000000000 in ?? ()
(gdb) bt
#0  0x0000000000000000 in ?? ()
#1  0x00007ffff7df77ed in __libc_start_main () from /lib64/libc.so.6
#2  0x000055555540055a in _start ()
(gdb) i r rax
rax            0x555555602620      93824992945696
(gdb) x/s $rax
0x555555602620 <flags+5632>:    "gigem{counteradvise_orbitoides}"
```

The program crashed because it was not prepared to return at the end of `get_flag`, but that doesn't stop us from reverse engineering this program. The first register I checked is the register that is used for result in x86-64, rax.

The solution is `gigem{counteradvise_orbitoides}`.

