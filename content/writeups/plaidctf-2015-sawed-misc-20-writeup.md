---
title: "PlaidCTF 2015 - Sawed (misc 20) Writeup"
slug: "plaidctf-2015-sawed-misc-20-writeup"
date: "2015-04-23 00:29:15.050018"
author_name: "tecknicaltom"
author_email: "tecknicaltom@neg9.org"
draft: false
toc: false
images:
---

Hint:

{language=python}
~~~~~~~~
What kind of mysterious message is this?!?

ddddddwwwwwwaaaaaasssssssssssssseddddddddddddddddddewwawwawwawwawwawwawwaedddssssssddddssssssssewwdwwdwwdwwdwwddssdssdssdssdsswwdwwdwwdwwdwwdwwdwwdassassassassassassassedddddddddddddddewwwwwwwwwwwwwwdssdssdssdssdssdssdsswwwwwwwwwwwwwwssssssssssssssedddddddddddewwwwwwwwwwwwweddddddddddessssssssssssssedddddddewwwwwwwwwwwwwweaaaaaaaedssdssdssdssdssdssdssewdwdwdddwddwdwwddwddwddwddewawwawawaaasasassasssasssdsssdsddsddddwdwwdwwwaaaessdddsddsddddsddddewdwdwdwdwdwdwdwawawawawawawaedddddddddddddddeddddddddaaaaaaaasssssssssssssswwwwwwwwdddddeddddddddeddddddwwwwwwaaaaaassssssssssssssedddddddddddddwwesdsddsdddwddwdwwwawwawawaawaawawwdwwdwdddsddsdsedddwwwdddesssssssssesssssess

Note: This flag does not have flag{}.
~~~~~~~~

I'll be honest. I stared at this one for a while not realizing what the catch was. I did notice that the whole string was made up of the characters "sawed", and that it contains many long strings of the characters except for "e", which always appears once. But I stared at it, played around with letter frequencies, run lengths, etc... stumped, thinking that maybe I was missing a pop culture reference or something.

It wasn't until my teammate coldwaterq, who plays more video games than I do, noticed that it was WASD that it clicked. The WASD characters were movement characters, and if that's the case, the 'e' character was probably toggling the pen state of the drawing "turtle".

That realization allowed me to throw together a quick and dirty Perl script which, after tweaking for canvas size and orientation looked like:

"System Message: ERROR/3 (<string>:, line 17)"
Error in "code-block" directive:
unknown option: "filename".

{language=python}
~~~~~~~~
.. code-block:: perl
   :number-lines:
   :filename: solve.pl

   #!/usr/bin/perl

   use strict;
   use warnings;
   use diagnostics;
   use feature 'say';
   use Data::Dumper;

   my $src = "ddddddwwwwwwaaaaaasssssssssssssseddddddddddddddddddewwawwawwawwawwawwawwaedddssssssddddssssssssewwdwwdwwdwwdwwddssdssdssdssdsswwdwwdwwdwwdwwdwwdwwdassassassassassassassedddddddddddddddewwwwwwwwwwwwwwdssdssdssdssdssdssdsswwwwwwwwwwwwwwssssssssssssssedddddddddddewwwwwwwwwwwwweddddddddddessssssssssssssedddddddewwwwwwwwwwwwwweaaaaaaaedssdssdssdssdssdssdssewdwdwdddwddwdwwddwddwddwddewawwawawaaasasassasssasssdsssdsddsddddwdwwdwwwaaaessdddsddsddddsddddewdwdwdwdwdwdwdwawawawawawawaedddddddddddddddeddddddddaaaaaaaasssssssssssssswwwwwwwwdddddeddddddddeddddddwwwwwwaaaaaassssssssssssssedddddddddddddwwesdsddsdddwddwdwwwawwawawaawaawawwdwwdwdddsddsdsedddwwwdddesssssssssesssssess";

   my ($x, $y) = (1,20);
   my @lines = (" "x200) x 30;

   my $pen  = 1;
   foreach my $l (split //, $src)
   {
        if($l eq 'w') {
                $y--;
        }elsif($l eq 's') {
                $y++;
        }elsif($l eq 'a') {
                $x--;
        }elsif($l eq 'd') {
                $x++;
        }elsif($l eq 'e') {
                $pen = $pen == 0;
        }
        substr($lines[$y], $x, 1) = 'X' if($pen);

   }
   print Dumper(\@lines);

~~~~~~~~

When run, the script outputs:

giving us the flag: PWNING>FPS!
