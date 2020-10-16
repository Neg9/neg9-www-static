---
title: "PlaidCTF 2015 - RE GEX (reversing 250) Writeup"
slug: "plaidctf-2015-re-gex-reversing-250-writeup"
date: "2015-04-22 23:30:46.611804"
author_name: "tecknicaltom"
author_email: "tecknicaltom@neg9.org"
draft: false
toc: false
images:
---

In this challenge, we are given a Python program that acts as a network server. After connecting to the game's server, we're prompted to "Enter your key". If the provided key does not match the provided regular expression, we're provided with the flag.

The file that contains the regex is enough to scare away the bravest regex novice. Breaking it down though, we see that it's a single pattern, anchored at the beginning and end of the string. The pattern contains many separate clauses joined with pipes (OR operators). Pulling apart the clauses, the first three are different and tell us a lot about the key:

- .*[^plaidctf].* - matches if the string contains any character outside of the characters "plaidctf". Since we are looking to not match, that means the key must be composed of only the characters "plaidctf"

- .{,170} - matches any string with 170 characters or fewer. Since we are looking to not match, that means the key must be 171 or greater characters long.

- .{172,} - matches any string with 172 characters or greater. Since we are looking to not match, that means the key must be 171 or fewer characters long.

Particularly, clauses two and three above tell us that the input string must be 171 characters long.

The rest of the regex is 2560 clauses that look like:

"System Message: ERROR/3 (<string>:, line 13)"
Content block expected for the "code" directive; none found.

{language=python}
~~~~~~~~
.. code:: .{15}[dctf].{132}[dt].{22}

~~~~~~~~

These clauses dictate that (because we're avoiding a match) certain characters of the string can't be certain combinations of letters. This pattern is one of the simpler ones, as most contain three character classes. It's important to remember that each clause is independent of the others, but the character classes within a clause are related. This pattern, for instance, rules out the following possibilities:

{language=python}
~~~~~~~~
.{15}d.{132}d.{22}
.{15}c.{132}d.{22}
.{15}t.{132}d.{22}
.{15}f.{132}d.{22}
.{15}d.{132}t.{22}
.{15}c.{132}t.{22}
.{15}t.{132}t.{22}
.{15}f.{132}t.{22}
~~~~~~~~

So to solve the challenge, we just need to find a 171-character long string composed of the letters "plaidctf" that does not match any of the 2560 patterns.

From an earlier attempt before fully understanding the problem, I had a file of preprocessed patterns that expanded the count quantifiers, making the previous pattern:

"System Message: ERROR/3 (<string>:, line 32)"
Error in "code-block" directive:
unknown option: "filename".

{language=python}
~~~~~~~~
.. code-block::
   :filename: patterns2.txt

   ...............[dctf]....................................................................................................................................[dt]......................

~~~~~~~~

Once I understood the problem though, it seemed like a perfect match for a SAT solver. For instance, the condition of the previous pattern basically says:

{language=python}
~~~~~~~~
Not(And(
        Or(c15=='d', c15=='c', c15=='t', c15=='f'),
        Or(c148=='d', c148=='t')
       ))
~~~~~~~~

The following program uses z3py to solve the challenge, reading the patterns in from a file named patterns2.txt listed one per line in the expanded syntax above. Due to the context switching between Perl (my language of choice) and Python (for the awesome z3py library), most of the script is sloppy non-Pythonic text processing that would make any Python programmer cringe.

"System Message: ERROR/3 (<string>:, line 48)"
Error in "code-block" directive:
unknown option: "filename".

{language=python}
~~~~~~~~
.. code-block:: python
   :number-lines:
   :filename: solve.z3.py

   #! /usr/bin/python

   from z3 import *

   chars = [Int('c_%d' % i) for i in range(171)]

   s = Solver()
   for char in chars:
       s.add(char >= 0)
       s.add(char < 8)

   lines = [line.strip() for line in open('patterns2.txt')]
   for line in lines:
       print line
       pos = 0
       clauses = []  # to hold the Or assertions, one per character class
       while line:
           if line[0] == '.':
               pos += 1
               line = line[1:]
           elif line[0] == '[':  # start of a new characer class
               line = line[1:]
               or_clause = []  # assertions per each character in the character class
               while line[0] != ']':  # while inside the character class
                   or_clause.append(chars[pos] == "plaidctf".index(line[0]))
                   line = line[1:]
               clauses.append(Or(or_clause))  # combine character assertions with Or
               line = line[1:]
               pos += 1
       s.add(Not(And(clauses)))  # combine Or clauses with Not(And(..)), add complete clause to solver



   print s
   print s.check()
   model = s.model()

   print "".join(["plaidctf"[model[char].as_long()] for char in chars])


~~~~~~~~

Despite the sloppy text processing, the real meat of the program is the creation of the 171 z3 variables (line 5), limiting these variables to the 8 possibilities for "plaidctf" (lines 9-10), building up of the pattern clauses (line 30), and running the solver and printing the results (lines 35-38).

After it had been churning for about two hours, I was thinking more about how there were no guarantees that it would finish in a reasonable amount of time. I started thinking about more specific ways of solving the problem that would likely be faster, when before long, I noticed my terminal:

{language=python}
~~~~~~~~
sat
cddliadtatddtcfidpfatdacctfcdiadplfdicdfldcltiftpdafpaddfdcddipappfdptapiptaitipccllpttpcifpdpdtapptfcppfdftccfialctdallcftaadlffaffpfpdiidltpacipdctdapfiddfdcpacdpidlpicp

real    153m55.631s
user    153m32.479s
sys     0m4.631s
~~~~~~~~

Netcat'ing back to the service and providing the key, the service provides the flag:

{language=python}
~~~~~~~~
$ nc 52.6.11.111 29285
Enter your key:
cddliadtatddtcfidpfatdacctfcdiadplfdicdfldcltiftpdafpaddfdcddipappfdptapiptaitipccllpttpcifpdpdtapptfcppfdftccfialctdallcftaadlffaffpfpdiidltpacipdctdapfiddfdcpacdpidlpicp
Congrats! You passed the test! Here's your FLAG: flag{np_hard_reg3x_ftw!!!1_ftdtatililactldtadf}
~~~~~~~~
