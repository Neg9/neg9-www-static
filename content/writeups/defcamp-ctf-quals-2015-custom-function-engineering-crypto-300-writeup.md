---
title: "Defcamp CTF Quals 2015 - Custom Function Engineering (crypto 300) Writeup"
slug: "defcamp-ctf-quals-2015-custom-function-engineering-crypto-300-writeup"
date: "2015-10-14 15:18:09.408334"
author_name: "tecknicaltom"
author_email: "tecknicaltom@neg9.org"
draft: false
toc: false
images:
---

Hint (with newlines added for publishing):

> You are given the following ciphertext:
> 
> 320b1c5900180a034c74441819004557415b0e0d1a316918011845524147384f5700264f48091e45
> 
> 00110e41030d1203460b1d0752150411541b455741520544111d0000131e0159110f0c16451b0f1c
> 
> 4a74120a170d460e13001e120a1106431e0c1c0a0a1017135a4e381b16530f330006411953664334
> 
> 593654114e114c09532f271c490630110e0b0b
> 
> And you are also give the code used to encrypt it: encrypt.php. Show us some magic! :-)
> 
> This challenge does not have a specific flag format.

And the linked PHP script:

{language=php}
~~~~~~~~
<?php

function xorIt($charOne, $charTwo)
{
      return $charOne ^ $charTwo;
}

function asciiValue($char)
{
      return ord($char);
}

function encrypt($plainText)
{
      $length = strlen($plainText);
      $space = 0xA;
      $cipherText = "";

      for ($i = 0; $i < $length; $i++) {
              if ($i + $space < $length - 1) {
                      $cipherText .= xorIt($plainText[$i], $plainText[$i + $space]);
              } else {
                      $cipherText .= xorIt($plainText[$i], $plainText[$space]);
              }

              $space = (asciiValue($plainText[$i]) % 2 == 0 ? $space + 1 : $space - 1);
      }

      return bin2hex($cipherText);
}

?>
~~~~~~~~

I fought with this one off and on throughout most of the weekend, attempting to break it through cryptanalysis. It was obvious that the encryption was far from secure. The bytes of the ciphertext are just a XOR of two plaintext bytes each. It's obvious which plaintext bytes go into the first ciphertext byte, but after that, it's less obvious. It also gets a bit screwy near the end of the plaintext. Like many of the other teams' writeups that I've seen, I attempted a tree-based approach, trying to determine possible values for the progression of the `space` variable, but it seemed to take forever and branch into an unwieldy number of possibilities. Personally, during the rush of a CTF isn't when I do my best cryptanalysis.

So eventually on the final day of the CTF, I thought this might actually be a good exercise for a SMT solver. So I threw together the following solver with Z3py. I had to play a lot with the assumptions made about the unknowns: over-constrained and the problem was unsat, under-constrained and I got a garbage plaintext that may encrypt to the given ciphertext but which doesn't get me any points. Below is a commented version of the solver script:

{language=python}
~~~~~~~~
#!/usr/bin/python

from z3 import *

ciphertext = [ 0x32, 0x0b, 0x1c, 0x59, 0x00, 0x18, 0x0a, 0x03, 0x4c, 0x74, 0x44, 0x18, 0x19, 0x00, 0x45, 0x57, 0x41, 0x5b, 0x0e, 0x0d, 0x1a, 0x31, 0x69, 0x18, 0x01, 0x18, 0x45, 0x52, 0x41, 0x47, 0x38, 0x4f, 0x57, 0x00, 0x26, 0x4f, 0x48, 0x09, 0x1e, 0x45, 0x00, 0x11, 0x0e, 0x41, 0x03, 0x0d, 0x12, 0x03, 0x46, 0x0b, 0x1d, 0x07, 0x52, 0x15, 0x04, 0x11, 0x54, 0x1b, 0x45, 0x57, 0x41, 0x52, 0x05, 0x44, 0x11, 0x1d, 0x00, 0x00, 0x13, 0x1e, 0x01, 0x59, 0x11, 0x0f, 0x0c, 0x16, 0x45, 0x1b, 0x0f, 0x1c, 0x4a, 0x74, 0x12, 0x0a, 0x17, 0x0d, 0x46, 0x0e, 0x13, 0x00, 0x1e, 0x12, 0x0a, 0x11, 0x06, 0x43, 0x1e, 0x0c, 0x1c, 0x0a, 0x0a, 0x10, 0x17, 0x13, 0x5a, 0x4e, 0x38, 0x1b, 0x16, 0x53, 0x0f, 0x33, 0x00, 0x06, 0x41, 0x19, 0x53, 0x66, 0x43, 0x34, 0x59, 0x36, 0x54, 0x11, 0x4e, 0x11, 0x4c, 0x09, 0x53, 0x2f, 0x27, 0x1c, 0x49, 0x06, 0x30, 0x11, 0x0e, 0x0b, 0x0b ]

length = len(ciphertext)
decrypt_len = length
plaintext = [ BitVec('p_%d'%i, 8) for i in range(0, length) ]
spaces = [ Int('sp_%d'%i) for i in range(0, length) ]

# the "space" value starts at 10, assume it doesn't deviate by more than 20
space_deviation = 20
min_space = 10-space_deviation
max_space = 10+space_deviation

s = Solver()
s.add(spaces[0] == 0xA)
for i in range(0, decrypt_len):

    # make some assumptions about the contents of the plaintext. otherwise, the solution it finds
    # is garbage that encrypts to the given ciphertext, but doesn't give me a flag
    s.add(Or(
        # plaintext is a symbol between space (0x20) and slash (0x2f)
        And(UGE(plaintext[i], ord(' ')), ULE(plaintext[i], ord('/'))),
        # plaintext is one of those symbols often in flags
        plaintext[i] == ord('{'),
        plaintext[i] == ord('}'),
        plaintext[i] == ord('_'),
        # plaintext is alphanumeric
        And(UGE(plaintext[i], ord('A')), ULE(plaintext[i], ord('Z'))),
        And(UGE(plaintext[i], ord('a')), ULE(plaintext[i], ord('z'))),
        And(UGE(plaintext[i], ord('0')), ULE(plaintext[i], ord('9')))
        ))

    # add assertions about what the next value of "space" will be, based on the current value
    # of "space" and the current plaintext
    if i != length-1:
        s.add(If(plaintext[i] & 1 == 0, spaces[i+1]==spaces[i]+1, spaces[i+1]==spaces[i]-1))

    # add assertions about our assumptions for the range of "space"
    s.add(spaces[i] <= max_space)
    s.add(spaces[i] >= min_space)

    # finally add assertion that we get the given ciphertext character based on the space value and
    # the unknown plaintext
    for sp in range(min_space, max_space+1):
        if(i + sp < length - 1):
            s.add(Implies(spaces[i] == sp, ciphertext[i] == plaintext[i] ^ plaintext[(i+sp+length)%length]))
        else:
            s.add(Implies(spaces[i] == sp, ciphertext[i] == plaintext[i] ^ plaintext[(sp+length)%length]))


print "Attempting to solve..."
print s.check()
m=s.model()

# print out the plaintext
print ''.join([chr(m[plaintext[i]].as_long()) for i in range(length)])
~~~~~~~~

The script runs for about 2.5 minutes, and produces the following output:

{language=python}
~~~~~~~~
Attempting to solve...
sat
Very well done you CTF pwner. Now I have to give you the reward for all this hard work or maybe guessing. The flag is cryptanalysis_is_hard
~~~~~~~~
