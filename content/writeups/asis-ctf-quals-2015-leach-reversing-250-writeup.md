---
title: " ASIS CTF Quals 2015 - Leach (reversing 250) Writeup"
slug: "asis-ctf-quals-2015-leach-reversing-250-writeup"
date: "2015-05-11 10:55:33.082715"
author_name: "tecknicaltom"
author_email: "tecknicaltom@neg9.org"
draft: false
toc: false
images:
---

The binary provided is a 64-bit Linux ELF executable. Opening it up in Hex-Rays and navigating to the main function, there is some obvious anti-debugging code:

{language=c}
~~~~~~~~
v11 = pthread_create(&newthread, 0LL, start_routine, 0LL);
if ( v11 )
{
  fprintf(stderr, "Error - pthread_create() return code: %d\n", (unsigned int)v11, a2);
  result = 0LL;
}
~~~~~~~~

Running the program, it displays an animated game of Pong that I'm too impatient to watch for more than a few seconds:

{language=python}
~~~~~~~~
this may take too long time ... :)

##############################
#|                          |#
#                            #
#                            #
#                            #
#                            #
#          o                 #
#                            #
#                            #
##############################
~~~~~~~~

So now my primary goal is to get past this slow intro. Looking back at Hex-Rays, with a minimal amount of reversing, you can see the call to sleep, but also that the amount of time elapsed during the call to sleep is detected and used in some capacity:

{language=c}
~~~~~~~~
v13 = time(0LL);
sleep(*(&seconds + v14));
v12 = (unsigned __int64)time(0LL) - v13;
sprintf(s, "%d", v12, v5);
~~~~~~~~

So to bypass the sleep, we also need to trick the elapsed time detection. I threw together this code for an LD_PRELOAD that doesn't sleep, but keeps track of sleep calls to fake out time return values:

"System Message: ERROR/3 (<string>:, line 40)"
Error in "code-block" directive:
unknown option: "filename".

{language=python}
~~~~~~~~
.. code-block:: c
   :filename: my_preload.c
   :number-lines:

   #include <time.h>

   static time_t my_time = 0;

   unsigned int sleep(unsigned int seconds)
   {
        my_time += seconds;
        return 0;
   }

   time_t time(time_t *t)
   {
        return my_time;
   }

~~~~~~~~

Running this code, it turns out that skipping the intro was sufficient:

{language=python}
~~~~~~~~
$ gcc -Wall -fPIC -shared -o my_preload.so my_preload.c
$ LD_PRELOAD=./my_preload.so ./leach
this may take too long time ... :)

##############################
ASIS{f18b0b4f1bc6c8af21a4a53ef002f9a2}
~~~~~~~~
