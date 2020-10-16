---
title: " ASIS CTF Quals 2015 - KeyLead (reversing 150) Writeup"
slug: "asis-ctf-quals-2015-keylead-reversing-150-writeup"
date: "2015-05-11 21:54:20.243791"
author_name: "tecknicaltom"
author_email: "tecknicaltom@neg9.org"
draft: false
toc: false
images:
---

Hint:

> Find the flag. [file](http://tasks.asis-ctf.ir/keylead_068128f7cacc63375c9cbab8114e15da)

After unpacking the file, we get a 64-bit Linux ELF executable. Opening it in Hex-Rays, jumping to the main function, and renaming the variables, we end up with the following straight-forward code:

{language=c}
~~~~~~~~
 1 unsigned __int64 __fastcall main(__int64 a1, char **a2, char **a3)
 2 {
 3   char v3; // ST1F_1@1
 4   unsigned int seed; // eax@1
 5   unsigned int roll1; // ST14_4@1
 6   unsigned __int64 result; // rax@3
 7   unsigned int roll5; // [sp+4h] [bp-1Ch]@1
 8   unsigned int roll4; // [sp+8h] [bp-18h]@1
 9   unsigned int roll3; // [sp+Ch] [bp-14h]@1
10   unsigned int roll2; // [sp+10h] [bp-10h]@1
11   int last_time; // [sp+18h] [bp-8h]@1
12 
13   puts("hi all ----------------------");
14   puts("Welcome to dice game!");
15   puts("You have to roll 5 dices and get 3, 1, 3, 3, 7 in order.");
16   puts("Press enter to roll.");
17   v3 = getchar();
18   seed = time(0LL);
19   srand(seed);
20   last_time = time(0LL);
21   roll1 = rand() % 6 + 1;
22   roll2 = rand() % 6 + 1;
23   roll3 = rand() % 6 + 1;
24   roll4 = rand() % 6 + 1;
25   roll5 = rand() % 6 + 1;
26   printf("You rolled %d, %d, %d, %d, %d.\n", roll1, roll2, roll3, roll4, roll5);
27   if ( roll1 != 3 )
28     goto LABEL_20;
29   if ( time(0LL) - last_time > 2 )
30   {
31     puts("No cheat!");
32     return 0xFFFFFFFFLL;
33   }
34   if ( roll2 != 1 )
35     goto LABEL_20;
36   if ( time(0LL) - last_time > 2 )
37   {
38     puts("No cheat!");
39     return 0xFFFFFFFFLL;
40   }
41   if ( roll3 != 3 )
42     goto LABEL_20;
43   if ( time(0LL) - last_time > 2 )
44   {
45     puts("No cheat!");
46     return 0xFFFFFFFFLL;
47   }
48   if ( roll4 != 3 )
49     goto LABEL_20;
50   if ( time(0LL) - last_time > 2 )
51   {
52     puts("No cheat!");
53     return 0xFFFFFFFFLL;
54   }
55   if ( roll5 != 7 )
56   {
57 LABEL_20:
58     puts("You DID NOT roll as I said!");
59     puts("Bye bye~");
60     result = 0xFFFFFFFFLL;
61   }
62   else if ( time(0LL) - last_time <= 2 )
63   {
64     puts("You rolled as I said! I'll give you the flag.");
65     success();
66     result = 0LL;
67   }
68   else
69   {
70     puts("No cheat!");
71     result = 0xFFFFFFFFLL;
72   }
73   return result;
74 }
~~~~~~~~

My first thought was that since this is a local challenge, a quick solution would be to use LD_PRELOAD to override rand() and force the random numbers to the desired rolls. In fact, I had just about finished the following code:

"System Message: ERROR/3 (<string>:, line 87)"
Error in "code-block" directive:
unknown option: "filename".

{language=python}
~~~~~~~~
.. code-block:: c
   :number-lines:
   :filename: my_preload.c

   static int call_num = 0;

   int rand()
   {
        switch(call_num++)
        {
        case 0:
                return 2;
        case 1:
                return 0;
        case 2:
                return 2;
        case 3:
                return 2;
        case 4:
                // um...?
        }
        return 0;
   }

~~~~~~~~

It was only after I got to the final roll that I realized that I would need to return a number that mod 6 plus 1 would be equal to 7, which doesn't exist. After thinking about it more, I realized again that the entire program runs locally and the success() function is entirely responsible for outputting the flag. A casual glance at the decompilation of success() shows that it's complex, but doesn't seem to depend on anything external.

So I thought the easiest way to get it to give me the flag was to just force the program to run the solve() function. I opened the executable in [HT Editor](http://hte.sourceforge.net/), navigated to the entrypoint function, and changed the push of the main function to instead push the address of success:

{language=python}
~~~~~~~~
Before:

4005dd ! 48c7c76e0e4000                   mov         rdi, offset_400e6e

After:

4005dd ! 48c7c7b6064000                   mov         rdi, offset_4006b6
~~~~~~~~

Running the patched program, it provides the key without any of the fuss of the dice rolls:

{language=python}
~~~~~~~~
$ ./keylead
ASIS{1fc1089e328eaf737c882ca0b10fcfe6}
~~~~~~~~
