---
title: "OpenCTF 2015 - absence (scripting,misc 200) Writeup"
slug: "openctf-2015-absence-scriptingmisc-200-writeup"
date: "2015-08-12 09:24:42.098023"
author_name: "hackworth"
author_email: "hackworth@bespokebytes.com"
draft: false
toc: false
images:
---

Hint:

> absence 200 --- What you see is what you get - 172.16.18.20/absence-df4268407e6b95d11388f81da1468978

The absence challenge started with a C file that on first examination is simple. The source code is:

{language=c}
~~~~~~~~
 1 #include <stdio.h>
 2 
 3 void one(char k)
 4 {
 5   unsigned int i, bytes[]={0x0a,0x5d,0x2c,0x0b,0x37,0x38,0x04,0x05,0x1f,0x4c,0x05,0x1f,0x4c,0x02,0x03,0x18,0x4c,0x18,0x04,0x09,0x4c,0x0f,0x03,0x08,0x09,0x4c,0x15,0x03,0x19,0x4b,0x1e,0x09,0x4c};
 6 
 7   for(i=0; i<32; i++)
 8       printf("%c",(char)bytes[i]^k);
 9   printf("\n");
10 }
11 
12 
13 char two(char c)
14 {
15   return (c^0x32)-7;
16 }
17 
18 
19 void main(int argc, char** argv)
20 {
21   one(two('A'));
22 }
~~~~~~~~

The provided C file has a bunch of trailing whitespace characters.

{language=c}
~~~~~~~~
 1 #include␠<stdio.h>␠␠␠␠␉␠␠␠␠␠↵
 2 ␉↵
 3 void␠one(char␠k)␠␠␠␉␉␠␉␉␠␠↵
 4 {␉↵
 5 ␠␠unsigned␠int␠i,␠bytes[]={0x0a,0x5d,0x2c,0x0b,0x37,0x38,0x04,0x05,0x1f,0x4c,0x05,0x1f,0x4c,0x02,0x03,0x18,0x4c,0x18,0x04,0x09,0x4c,0x0f,0x03,0x08,0x09,0x4c,0x15,0x03,0x19,0x4b,0x1e,0x09,0x4c};␉␉␠␉␉␉␉↵
 6 ␉↵
 7 ␠␠for(i=0;␠i<32;␠i++)␠␉␉␠␠␠␠↵
 8 ␉printf("%c",(char)bytes[i]^k);↵
 9 ␠␠printf("\n");␠␠␠␉␉␠␉␠␉␉↵
10 }␉↵
11 ␠␠␠␠␠␉␉␠␉␠␠␉↵
12 ␉↵
13 char␠two(char␠c)␠␠␠␉␉␠␉␉␉␠↵
14 {␉↵
15 ␠␠return␠(c^0x32)-7;␠␠␉␠␉␉␠␉↵
16 }␉↵
17 ␠␠␠␠␠␉␉␠␠␉␉␠↵
18 ␉↵
19 void␠main(int␠argc,␠char**␠argv)␠␠␉␉␠␉␉␉␉↵
20 {␉↵
21 ␠␠one(two('A'));␠␠␠␉␠␉␉␉␠␉↵
22 }␉↵
23 ␠␠↵
24 ↵
25 ↵
26 ↵
~~~~~~~~

Running the program gives:

"System Message: ERROR/3 (<string>:, line 67)"
Error in "code" directive:
maximum 1 argument(s) allowed, 6 supplied.

{language=python}
~~~~~~~~
.. code:: f1@g[This is not the code you're

~~~~~~~~

Extracting the trailing white space characters with:

"System Message: ERROR/3 (<string>:, line 71)"
Error in "code" directive:
maximum 1 argument(s) allowed, 8 supplied.

{language=python}
~~~~~~~~
.. code:: cat absence-df4268407e6b95d11388f81da1468978 | sed -E "s/[^[:space:]]//g" > whitespaces.txt

~~~~~~~~

Yields the following file with whitespace only.

{language=python}
~~~~~~~~
␠␠␠␠␠␉␠␠␠␠␠↵
␉↵
␠␠␠␠␠␉␉␠␉␉␠␠↵
␉↵
␠␠␠␠␠␉␉␠␉␉␉␉↵
␉↵
␠␠␠␠␠␉␉␠␠␠␠↵
␉↵
␠␠␠␠␠␉␉␠␉␠␉␉↵
␉↵
␠␠␠␠␠␉␉␠␉␠␠␉↵
␉↵
␠␠␠␠␠␉␉␠␉␉␉␠↵
␉↵
␠␠␠␠␠␉␠␉␉␠␉↵
␉↵
␠␠␠␠␠␉␉␠␠␉␉␠↵
␉↵
␠␠␠␠␠␠␉␉␠␉␉␉␉↵
␉↵
␠␠␠␠␠␉␠␉␉␉␠␉↵
␉↵
␠␠↵
↵
↵
↵
~~~~~~~~

Executing that [whitespace](https://en.wikipedia.org/wiki/Whitespace_(programming_language)) using the [WS2JS](http://ws2js.luilak.net/interpreter.html) interpreter yields:

"System Message: ERROR/3 (<string>:, line 106)"
Content block expected for the "code" directive; none found.

{language=python}
~~~~~~~~
.. code::  lo0kin-fo]


~~~~~~~~

The flag: This is not the code you're lo0kin-fo

Thanks to tecknicaltom for his insights.
