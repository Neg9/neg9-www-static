---
title: "DEF CON CTF Quals 2015 - catwestern (coding challenge 1) Writeup"
slug: "def-con-ctf-quals-2015-catwestern-coding-challenge-1-writeup"
date: "2015-05-18 15:06:13.361175"
author_name: "FalconK"
author_email: "falcon@iridiumlinux.org"
draft: false
toc: false
images:
---

This challenge asks us to solve a programming problem. The TCP service
is running on port 9999. When you connect to it, it tells you:

{language=python}
~~~~~~~~
****Initial Register State****
rax=0xef91c9bc5e06177b
rbx=0x2182d45a3d3e608c
rcx=0x6bd55e3cde9d0e55
rdx=0xdd59f3fc1c666a5e
rsi=0xbc9ec68be44252db
rdi=0x3327008906dea77c
r8=0xdca252f662c1eede
r9=0xb9b44beb597bc359
r10=0xbd6649285ef50a50
r11=0x37b751981d3582d
r12=0xfa34e739634ccb7b
r13=0x6824becdc88f4ed
r14=0xebd638495d51e14a
r15=0x6935dba2deba8f6d
****Send Solution In The Same Format****
About to send 76 bytes:
~~~~~~~~

Then, it sends that many bytes of x86_64 bytecode. It is a backwards
challenge!  You write the shellcode-running service this time.

The register state changes each time, as does the length of the bytecode. The problem,
intuitively enough, is asking us to execute the code it sends us with
the initial register state, and print the register state back to it.

This is not very hard, if you know a little assembly. All you have to
do is read the register state into memory, put the code somewhere
executable, and do some marshalling. An observation of the code
indicates that it is basically all ALU instructions, which makes it
pretty safe, but I ran it in an EC2 instance anyway so I didn't even
have to worry about running the solution as root.

Here is some very simple code that does this.

"System Message: ERROR/3 (<string>:, line 40)"
Error in "code-block" directive:
unknown option: "filename".

{language=python}
~~~~~~~~
.. code-block:: cpp
   :number-lines:
   :filename: parse.cpp

   #include <stdio.h>
   #include <stdint.h>
   #include <stdlib.h>
   #include <string.h>
   #include <sys/mman.h>

   uint64_t regstate[14]; //a, b, c, d, si, di, 9-15
   const char* regnames[] = {"rax", "rbx", "rcx", "rdx", "rsi", "rdi",
   "r8", "r9", "r10", "r11", "r12", "r13", "r14", "r15"};

   extern "C" void shellcall(void* regstate, void* shellcode);

   int main() {
           char* foo = 0;
           size_t foo_sz = 0;
           size_t ri=0;

           getline(&foo, &foo_sz, stdin); //get header
           fprintf(stderr, "Got header: %s", foo);

           while (getline(&foo, &foo_sz, stdin) > 0 && ri<14) {
                   //fprintf(stderr, "l=%d %s\n", ri, foo);
                   regstate[ri++] = strtoull(strstr(foo, "0x"), 0, 16);
                   fprintf(stderr, "%s=0x%016llx\n", regnames[ri-1], regstate[ri-1]);
           }

           getline(&foo, &foo_sz, stdin); //get header
           fprintf(stderr, "Got header: %s", foo);

           size_t n_bytes = 0;
           sscanf(foo, "About to send %u", &n_bytes);

           fprintf(stderr, "Expecting %u bytes\n", n_bytes);
           uint8_t* shellcode = (uint8_t*)mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0);

           for (int i=0;i<n_bytes;i++) shellcode[i] = fgetc(stdin);

           mprotect(shellcode, 4096, PROT_READ|PROT_EXEC);

           fprintf(stderr, "Exec shellcode: regstate at 0x%016lX  shellcode at 0x%016lX\n", (uint64_t)regstate, (uint64_t)shellcode);
           shellcall(regstate, shellcode);

           for (int i=0;i<14;i++) fprintf(stderr, "%s=0x%016llx\n", regnames[i], regstate[i]);
           for (int i=0;i<14;i++) printf("%s=0x%016llx\n", regnames[i], regstate[i]);
           printf("\n");
           fflush(stdout);

           while (getline(&foo, &foo_sz, stdin) > 0) fprintf(stderr, "%s", foo);

           return 0;
   }

~~~~~~~~

"System Message: ERROR/3 (<string>:, line 96)"
Error in "code-block" directive:
unknown option: "filename".

{language=python}
~~~~~~~~
.. code-block:: gas
   :number-lines:
   :filename: shellcall.S

   .section .text
   .globl shellcall
   .type shellcall, @function

   shellcall:
   // rdi = regstate, rsi = shellcode
   push %rbp
   push %rax
   push %rbx
   push %rcx
   push %rdx
   push %rsi
   push %rdi
   push %r8
   push %r9
   push %r10
   push %r11
   push %r12
   push %r13
   push %r14
   push %r15

   push %rsi
   movq %rdi,%rbp

   movq 0(%rbp),%rax
   movq 8(%rbp),%rbx
   movq 16(%rbp),%rcx
   movq 24(%rbp),%rdx
   movq 32(%rbp),%rsi
   movq 40(%rbp),%rdi
   movq 48(%rbp),%r8
   movq 56(%rbp),%r9
   movq 64(%rbp),%r10
   movq 72(%rbp),%r11
   movq 80(%rbp),%r12
   movq 88(%rbp),%r13
   movq 96(%rbp),%r14
   movq 104(%rbp),%r15

   push %rbp
   call *8(%rsp)
   pop %rbp

   movq %r15,104(%rbp)
   movq %r14,96(%rbp)
   movq %r13,88(%rbp)
   movq %r12,80(%rbp)
   movq %r11,72(%rbp)
   movq %r10,64(%rbp)
   movq %r9,56(%rbp)
   movq %r8,48(%rbp)
   movq %rdi,40(%rbp)
   movq %rsi,32(%rbp)
   movq %rdx,24(%rbp)
   movq %rcx,16(%rbp)
   movq %rbx,8(%rbp)
   movq %rax,0(%rbp)

   pop %rax //discard
   pop %r15
   pop %r14
   pop %r13
   pop %r12
   pop %r11
   pop %r10
   pop %r9
   pop %r8
   pop %rdi
   pop %rsi
   pop %rdx
   pop %rcx
   pop %rbx
   pop %rax
   pop %rbp

   ret

~~~~~~~~

The mmap and mprotect in parse.cpp take care of putting the shellcode in
an executable place.

As you can see, I am very lazy and did not want to worry about what can
be clobbered or not. The only trick to this assembler function is the
state marshalling. We need to set a lot of registers for the initial
state, but the problem mercifully leaves RBP and RSP alone, and although
the shellcode uses the stack, it only ever does push and pop, and never
does base pointer dereferences at all. So, RBP is free for our use. I
put the location of my register state array in RBP and proceed to set up
the registers. The call address is placed on the stack (via push rsi),
and we save RBP across the shellcode call even though it isn't likely
necessary. Then, we just call the saved shellcode pointer, and run that.

The shellcode that gets sent to us (yes, it isn't strictly shellcode,
just code) always ends in a ret. So, we are very safe. Just wait for
it to ret, save off state, pop all our saved state off the stack, and
return into parse.cpp.

Then, we can print the register state back to the service, in the same
format as before, and it will tell us the flag. Here is a transcript of
my session:

{language=python}
~~~~~~~~
[root@ip-172-31-23-134 ec2-user]# socat TCP4:catwestern_631d7907670909fc4df2defc13f2057c.quals.shallweplayaga.me:9999 EXEC:./a.out
Got header: ****Initial Register State****
rax=0x9f9da7286ac5281e
rbx=0x92fbcd1edafe5bb2
rcx=0xf34f594b5cc1e0b2
rdx=0x1b1c24508b7cb519
rsi=0x902a02cae9a0e63f
rdi=0xc56beb738c5b1b11
r8=0xd13193869ca5ad2a
r9=0x41e6fc66e96f9cd6
r10=0x9741e20d00176ab2
r11=0x906cd1136b84cc21
r12=0xa0bb2b7f459a649d
r13=0x54188af1909e1eb7
r14=0x603511b2ade4e546
r15=0x87a16908441ab303
Got header: About to send 87 bytes:
Expecting 87 bytes
Exec shellcode: regstate at 0x00000000006014A0  shellcode at
0x00007FE0236EA000
rax=0x9f9da7286ac5281e
rbx=0xe562c9d8e0e6f14b
rcx=0x0cb0a6b4a33e1f4d
rdx=0x1b1c24508b7cb519
rsi=0xa0bb2b7f459a649c
rdi=0x000000001947faf8
r8=0x2ece6c797ffef2f6
r9=0x41e6fc66e96f9cd6
r10=0xaadb2fb59e405196
r11=0xf143280600000000
r12=0x000000001947804c
r13=0xa7571845f5f58250
r14=0x603511b2a44847da
r15=0x87a16907fe758657
The flag is: Cats with frickin lazer beamz on top of their heads!
[root@ip-172-31-23-134 ec2-user]#
~~~~~~~~
