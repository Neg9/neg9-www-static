---
title: "Boston Key Party 2015 - Sullivan Square (reversing 350) Writeup"
slug: "boston-key-party-2015-sullivan-square-reversing-350-writeup"
date: "2015-03-04 17:57:43.680295"
author_name: "zombieCraig"
author_email: "agent.craig@gmail.com"
draft: false
toc: false
images:
---

## Boston Key Party 2015 - Sullivan Square (Reversing 350) Writeup
by zombieCraig

This challenge came with a tar file that decompressed to the following files:

- cipher.rbc

- trie_harder.rbc

- run.sh

- trie.rbc

- trie.dump

The contents of *run.sh* simply was:

{language=bash}
~~~~~~~~
rbx -I. -e "Rubinius::CodeLoader.require_compiled 'trie_harder'"
~~~~~~~~

After a bit of googling you can see that Rubinius compiles ruby code into byte code. You can launch compiled
Rubinius code (extension .rbc) with the rbx command. More info on Rubinius can be found [here](http://rubini.us/doc/en/).
After a bit more poking around on the Rubinius site you can see that you can load an interactive debugger!  All
you need to do is insert -Xdebug into the command line. Fantastic!  How hard could this possible be?
Ruby + a builtin debugger. Lulz, bring it.

### First Hurdle
After downloading Rubinius 2.5.2 and getting it compiled locally I setup my run.sh as follows:

{language=bash}
~~~~~~~~
#!/bin/sh
# Your PATHing my vary :)
PATH=$PATH:/home/zombieCraig/ctf/bkp-2015/sullivan/rubinius-2.5.2/bin/
rbx -Xdebug -I. -e "Rubinius::CodeLoader.require_compiled 'trie_harder'"
~~~~~~~~

However, when I went to run the application it aborted, tossing an exception of Rubinius::InvalidRBC.
Strangely didn't see much on google on this. So I compiled a quick "Hello World" program of my own and
compared headers:

## My header:
"System Message: WARNING/2 (<string>:, line 44)"
Cannot analyze code. No Pygments lexer found for "hex".

{language=python}
~~~~~~~~
.. code-block:: hex

  0000000: 2152 4249 580a 3431 3933 3131 3233 3735  !RBIX.4193112375
  0000010: 3834 3639 3234 3633 340a 3235 0a4d 0a31  846924634.25.M.1
  0000020: 0a6e 0a6e 0a78 0a45 0a38 0a55 532d 4153  .n.n.x.E.8.US-AS

~~~~~~~~

## Target's header:
"System Message: WARNING/2 (<string>:, line 53)"
Cannot analyze code. No Pygments lexer found for "hex".

{language=python}
~~~~~~~~
.. code-block:: hex

  0000000: 2152 4249 580a 3130 3138 3434 3635 3336  !RBIX.1018446536
  0000010: 3537 3833 3035 3536 3436 0a32 350a 4d0a  5783055646.25.M.
  0000020: 310a 6e0a 6e0a 780a 450a 380a 5553 2d41  1.n.n.x.E.8.US-A


~~~~~~~~

Ok, not terribly helpful. Looking at the source we see the header is very simple and is broken
into 3 parts separated by a '\n'.

- Magic => !RBIX

- Signature => 10184465365783055646

- Version => 25

So we have the same Magic and Version...must be the signature. Looking at the source for Rubinius
the signature is just a hash of a bunch files it used and is fairly unique to each install of
Rubinius. Seems kind of strange to require a specific compiled version of Rubinius if you plan
on distributing bytecode but oh well.

From *vm/builtin/system.cpp*:

{language=c++}
~~~~~~~~
CompiledFile* cf = CompiledFile::load(stream);
 if(cf->magic != "!RBIX") {
   delete cf;
   return Primitives::failure();
 }

 uint64_t sig = signature->to_ulong_long();
 if((sig > 0 && cf->signature != sig)
     || cf->version != version->to_native()) {
   delete cf;
   return Primitives::failure();
 }
~~~~~~~~

A quick patch to rbx to ignore signature comparisons and the application
prompted for the flag as expected, plus we got the debugger prompt!

### The Next Hurdle
So the Rubinius debugger has lots of great stuff such as setting breakpoints, step in/over, printing
variables and direct disassembly. Joy!  So stepping through some code you can see the execution
path is basically:

- trie_harder - Asks for a password (flag)

- Loads Trie class

- Loads Cipher class

- Loads trie.dump (Marshal dump of a linked list with 3 nodes: left, right and mid)

- cipher.encrypts your flag

- uses the flag to walk the link list with Trie.get_recursive

- If anything goes wrong you get a `nop`

Skipping Cipher for a moment let's look at the Trie class because that seems to be where all the
logic lives.

### Rubinius Debugger Tips
Quick aside. When it comes to Rubinius there are certain methods that work rather well. First
load the debug with **-Xdebug** of course. For this challenge a good starting point is to set a break
point on *gets()*

{language=python}
~~~~~~~~
debug> b Object#gets
Set breakpoint 1: kernel/common/kernel.rb:347 (+0)
~~~~~~~~

Hit `c` to continue. When it breaks you will need to hit `n` for Next a few times until it actually
takes your stdin. After that you should type

{language=python}
~~~~~~~~
debug> dis all
~~~~~~~~

This will disassemble all the bytecode for that method. It's not a bytecode you will recognize but it
is still really easy to read. Sample:

"System Message: WARNING/2 (<string>:, line 132)"
Cannot analyze code. No Pygments lexer found for "assembly".

{language=python}
~~~~~~~~
.. code-block:: assembly

  0073:  send_method                :gets
  0075:  send_stack                 :chomp, 0
  0078:  set_local                  0    # flag
  0080:  pop
  0081:  push_const_fast            :File
  0083:  push_literal               "trie.dump"
  0085:  string_dup
  0086:  send_stack                 :read, 1
  0089:  set_local                  1    # dumped
  0091:  pop
  0092:  push_const_fast            :Marshal
  0094:  push_local                 1    # dumped

~~~~~~~~

As you step through code you can use `p` to evaluate code in the current context. This becomes invaluable,
because you can dump an entire class, its methods...whatever you want via ruby. nbd.

### Back to the problem at hand
So if you break at Trie#get_recursive you can stop just before it starts to analyze each node in the
chain. If you print @root you can see that there is a LOT of junk nodes and bogus flags. Scanning through
the dump you can see a "good boy" message for the proper flag. Now how to get there?  Looking at the
get_recursive method you can see it's fairly simple. Each node contains a character and if the first
character of your flag (encrypted) is less it goes left, greater it goes right, equal it drops down mid
and increments the index of the flag.

So now we know how to move the chains.. The core concept of how this flags works.. we just need to
know how to determine the path to the *last* good boy message. If your index ends too soon you will
also fail. The node tree reminds me of a sorting algorithm but I don't bother trying to do
anything clever here. I notice that node "values" repeat. Most of this tree structure only branches
in the first 2 steps then it only uses "mid" to drop down and during that time that value repeats
the bad boy message. You can simply print paths via the debugger until you find the right debug node.
Use a piece of paper to keep track of bogus paths.

By doing things like:

{language=python}
~~~~~~~~
debug> p @root.right
~~~~~~~~

You can quickly scan the results to see if you see the "good boy" message. If it is nowhere in the chain
then go the other direction. Repeat. You will find the first "good boy" message here.

{language=python}
~~~~~~~~
debug> p @root.right.left.mid.mid.mid.right.value
$d0 = "good boy this is the flag"
~~~~~~~~

Now you just follow the "mid" until the end of the chain. It's a good ways down still. Each "char" value
of each node is your path. Basically what you need is the "mid" values. Which are:

{language=python}
~~~~~~~~
XcXFAp9F0Wc8FDHcveFypMWF288i
~~~~~~~~

Great!  But not the flag.

### Final Hurdle
Now we know the process, the flag location, the exact method to get the flag.. All that is left is
figuring out how the flag we enter gets "encrypted" and then determine how we can enter our flag
so the end result is this string.

So if we look at the Cipher class we can see it simply maps characters to other characters. Proper
timing in the debugger will show a detailed map of every character mapping. Formatted for readability:

"System Message: WARNING/2 (<string>:, line 201)"
Cannot analyze code. No Pygments lexer found for "assembly".

{language=python}
~~~~~~~~
.. code-block:: assembly

  debug> p cipher
  $d6 = #<Cipher:0x4be8 @map=
  {
          :encrypt=>
          {
                  "a"=>"K", "b"=>"D", "c"=>"w", "d"=>"X", "e"=>"H", "f"=>"3",
                  "g"=>"e", "h"=>"1", "i"=>"S", "j"=>"B", "k"=>"g", "l"=>"a",
                  "m"=>"y", "n"=>"v", "o"=>"I", "p"=>"6", "q"=>"u", "r"=>"W",
                  "s"=>"C", "t"=>"0", "u"=>"9", "v"=>"b", "w"=>"z", "x"=>"T",
                  "y"=>"A", "z"=>"q", "A"=>"U", "B"=>"4", "C"=>"O", "D"=>"o",
                  "E"=>"E", "F"=>"N", "G"=>"r", "H"=>"n", "I"=>"m", "J"=>"d",
                  "K"=>"k", "L"=>"x", "M"=>"P", "N"=>"t", "O"=>"R", "P"=>"s",
                  "Q"=>"J", "R"=>"L", "S"=>"f", "T"=>"h", "U"=>"Z", "V"=>"j",
                  "W"=>"Y", "X"=>"5", "Y"=>"7", "Z"=>"l", "0"=>"p", "1"=>"c",
                  "2"=>"2", "3"=>"8", "4"=>"M", "5"=>"V", "6"=>"G", "7"=>"i",
                  "8"=>" ", "9"=>"Q", " "=>"F"
          },

          :decrypt=>
          {
                  "K"=>"a", "D"=>"b", "w"=>"c", "X"=>"d", "H"=>"e", "3"=>"f",
                  "e"=>"g", "1"=>"h", "S"=>"i", "B"=>"j", "g"=>"k", "a"=>"l",
                  "y"=>"m", "v"=>"n", "I"=>"o", "6"=>"p", "u"=>"q", "W"=>"r",
                  "C"=>"s", "0"=>"t", "9"=>"u", "b"=>"v", "z"=>"w", "T"=>"x",
                  "A"=>"y", "q"=>"z", "U"=>"A", "4"=>"B", "O"=>"C", "o"=>"D",
                  "E"=>"E", "N"=>"F", "r"=>"G", "n"=>"H", "m"=>"I", "d"=>"J",
                  "k"=>"K", "x"=>"L", "P"=>"M", "t"=>"N", "R"=>"O", "s"=>"P",
                  "J"=>"Q", "L"=>"R", "f"=>"S", "h"=>"T", "Z"=>"U", "j"=>"V",
                  "Y"=>"W", "5"=>"X", "7"=>"Y", "l"=>"Z", "p"=>"0", "c"=>"1",
                  "2"=>"2", "8"=>"3", "M"=>"4", "V"=>"5", "G"=>"6", "i"=>"7",
                  " "=>"8", "Q"=>"9", "F"=>" "
          }
  }
  >

~~~~~~~~

So we *could* do this by hand but as you can see there are two maps. One for encrypt and one for decrypt.
Let's have the system prep our string for us then!  Do not put a break on get_recursive because that will
be out of context of the Cipher class. Set a breakpoint at Trie#getc and step a few times via `n`. Like 3 times.
Until your output matches:

{language=asm}
~~~~~~~~
| Breakpoint: Trie#get(key) at /home/zombieCraig/ctf/bkp-2015/sullivan/distribute/trie.rbc:60 (408)
| 60: s
debug> dis
==== Bytecode between 408 and 420 for line 60 ====
|   408: push_local cipher
|   410: push_local key
|   412: send_stack :to_s, 0
|   415: send_stack :encrypt, 1
|   418: set_local key
|   420: pop
~~~~~~~~

Now just feed it the target string:

{language=bash}
~~~~~~~~
debug> p cipher.decrypt "XcXFAp9F0Wc8FDHcveFypMWF288i"
$d2 = "d1d y0u tr13 be1ng m04r 2337"
~~~~~~~~

That's it!  If you restart and try again using "d1d y0u tr13 be1ng m04r 2337" then you will get the final
"good boy" message.
