---
title: "Hack.lu Quals 2015 - Checkcheckcheck (crypto 150) Writeup"
slug: "hacklu-quals-2015-checkcheckcheck-crypto-150-writeup"
date: "2016-05-03 23:00:00.350484"
author_name: "tecknicaltom"
author_email: "tecknicaltom@neg9.org"
draft: false
toc: false
images:
---

Hint:

> We seem to have a small security problem with our new flag storage service. We already added a password login to it, with a fancy hash function, but somehow, these hackers still manage to log in. Something seems to be broken...

This challenge provided the C source code, mirrored [here](https://neg9.org/resources/media/hacklu-quals-2015-checkcheckcheck-crypto-150-writeup/server.c).

Studying the source code, it can be seen that the server does the following:

- Upon receiving a connection and forking, it calls the `handle()` method. Before any user interaction, this method reads a 32-byte `password_salt` and a 6-byte `password_hash` from disk.

- It loops, accepting commands from the user:

  - One command is "getflag" which checks the value of the `logged_in` variable to determine whether to give an error message or cat the flag file.

  - Another command is "login" followed by a password. The program does the following to process a login request:

    - Hashes the password with the salt mentioned above. Hashing is done with SHA-256, truncated to 6 bytes.

    - XORs the resulting hash with `password_hash` storing the value back into `password_hash`.

    - Performs a constant-time check to see if the result of the XOR is all zero (and thus the entered password hashes to the same as the hash from disk). If so, it sets the `logged_in` variable to true. Otherwise, it gives an error and usefully provides us with the value of the XOR.

The main takeaways from this are that we are provided with the access of `password_hash ^ hash(input_password)`, and that a failed login attempt overwrites `password_hash` with this value.

The first step is determining the desired password hash. If we attempt to login twice with the same password, the resulting XOR that is shown to us will be the initial desired password:

{language=python}
~~~~~~~~
password_hash = desired_hash
// login attempt with "a"
password_hash = password_hash ^ hash("a") = desired_hash ^ hash("a")
// another login attempt with "a"
password_hash =  password_hash ^ hash("a") = desired_hash ^ hash("a") ^ hash("a") = desired_hash
~~~~~~~~

That step is easy enough to do manually:

{language=python}
~~~~~~~~
$ nc school.fluxfingers.net 1513
login a
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 33d5d6b96bbd
login a
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 785e64baad24
^C
~~~~~~~~

So the desired password hash is 785e64baad24.

Next, we can derive what any password hashes to by XORing the resulting value of `password_hash` with the previous value. I wrote a script to automate this step, and gathered the hashes of all alphanumeric passwords of length one or two.

Now we need to find a subset of passwords whose hashes XORed together equals the desired hash. I tried a few unsuccessful methods until I remembered my Linear Algebra classes in college. What I was looking for was essentially a subset of the inputs that form a basis of addition in GF(2). Instead of trying to look specifically for a subset of hashes that XOR to the desired hash, look for XOR-subsets of inputs that could be combined to make any hash.

The program I wrote to finally solve this problem is:

{language=perl}
~~~~~~~~
  1 #!/usr/bin/perl
  2 
  3 use strict;
  4 use warnings;
  5 use diagnostics;
  6 use feature 'say';
  7 use Data::Dumper;
  8 
  9 my %hashes;
 10 
 11 sub numbits($)
 12 {
 13       my ($a) = @_;
 14       my $count = 0;
 15       while($a)
 16       {
 17               $count++ if($a&1);
 18               $a >>= 1;
 19       }
 20       return $count
 21 }
 22 
 23 sub remove_redundant(@)
 24 {
 25       my @values = @_;
 26       my %out_values;
 27       foreach my $value (@values)
 28       {
 29               if($out_values{$value})
 30               {
 31                       delete $out_values{$value};
 32               }
 33               else
 34               {
 35                       $out_values{$value} = 1;
 36               }
 37       }
 38       return keys %out_values;
 39 }
 40 
 41 sub reduce(@)
 42 {
 43       my @values = @_;
 44       @values = sort {$b <=> $a} @values;
 45       my @rows = map { {value=>$_, inputs=>[$_]} } @values;
 46 
 47       for my $i (0 .. $#rows - 1)
 48       {
 49               last if($rows[$i]->{value} == 0);
 50               my $i_leading0 = 48 - length(sprintf("%b", $rows[$i]->{value}));
 51               for my $j ($i + 1 .. $#rows)
 52               {
 53                       my $j_leading0 = 48 - length(sprintf("%b", $rows[$j]->{value}));
 54                       last if($j_leading0 > $i_leading0);
 55                       $rows[$j]->{value} ^= $rows[$i]->{value};
 56                       $rows[$j]->{inputs} = [ remove_redundant(@{$rows[$j]->{inputs}}, @{$rows[$i]->{inputs}}) ];
 57               }
 58               @rows = sort {
 59                       length(sprintf "%b", $b->{value}) <=> length(sprintf "%b", $a->{value}) or
 60                       index(sprintf("%b", $a->{value}), "0") <=> index(sprintf("%b", $b->{value}), "0")
 61               } @rows;
 62       }
 63       say STDERR join "\n", map { sprintf "%048b (%d)", $_->{value}, scalar(@{$_->{inputs}}) } (@rows);
 64       return @rows;
 65 }
 66 
 67 
 68 while(<>)
 69 {
 70       chomp;
 71       next unless(/^([0-9a-f]{12}) (.*)/);
 72       my ($hash, $pw) = ($1, $2);
 73       $hashes{hex($hash)} = $pw;
 74 }
 75 my @hashes_by_bits = sort {numbits($a) <=> numbits($b) } keys %hashes;
 76 my @rows = reduce(@hashes_by_bits[0 .. 99]);
 77 my $desired_hash = 0x785e64baad24;
 78 
 79 my $value = 0;
 80 my @inputs = ();
 81 while($value != $desired_hash)
 82 {
 83       printf STDERR "%012x\n%012x\n====\n", $value, $desired_hash;
 84       foreach my $row (@rows)
 85       {
 86               if(length(sprintf("%b",$desired_hash ^ $value)) == length(sprintf("%b", $row->{value})))
 87               {
 88                       $value ^= $row->{value};
 89                       @inputs = remove_redundant(@inputs, @{$row->{inputs}});
 90                       last;
 91               }
 92       }
 93 }
 94 
 95 foreach(@inputs)
 96 {
 97       say "login ".$hashes{$_};
 98       say STDERR "login ".$hashes{$_};
 99 }
100 say STDERR "getflag";
101 say "getflag";
~~~~~~~~

Starting outside of the functions, lines 68 - 75 read in the password hashes retrieved earlier, stores them in `$hashes`, and sorts them by the number of bits set in the hash. Line 76 takes the 100 hashes (even though only 48 should be necessary) with the fewest set bits and passes that to the `reduce` function.

The `reduce` function determines the basis through a process similar to finding Row Echelon Form. Each value in `@rows` keeps track of a value and a list of the inputs that XOR together to make that value, initialized with the input hashes. After sorting the rows, it then enumerates through the rows from first to last, each time:

- XOR the current row with each of the rows below it that have the same number of leading zeros.

- Add this row's inputs to the lower row's inputs, removing redundant terms (any term XORed twice cancels out).

- Sort the rows.

To illustrate this concept on smaller inputs, the method would reduce the following random 6-bit values:

{language=python}
~~~~~~~~
binary  decimal
001100  12
011101  29
011011  27
001000  8
110000  48
011010  26
001001  9
100001  33
101011  43
010111  23
001101  13
110101  53
100010  34
000001  1
110001  49
000101  5
001010  10
111111  63
001010  10
011001  25
~~~~~~~~

into:

{language=python}
~~~~~~~~
value  inputs
111111 (63)
010100 (43,63)
001111 (27,43,63)
000111 (43,8,63,27)
000011 (12,27,63,43)
000001 (27,12,33,63,8)
000001 (63,33,8,23)
000001 (43,8,23,53)
000001 (8,48,53,12)
000001 (48,12,63,10,8)
000000 ()
000000 (5,27,43,63,10)
000000 (10,8,25,27)
000000 (13,27,43,63,10,8)
000000 (27,12,34,63,10)
000000 (12,29,27,10)
000000 (10,63,43,27,12,9)
000000 (63,10,8,26,43,12)
000000 (49,8,10,63,12)
000000 (12,27,8,10,63,1,43)
~~~~~~~~

Only the first 6 of those results are needed:

{language=python}
~~~~~~~~
value  inputs
111111 (63)
010100 (43,63)
001111 (27,43,63)
000111 (43,8,63,27)
000011 (12,27,63,43)
000001 (27,12,33,63,8)
~~~~~~~~

Any 6-bit number can be easily constructed by XORing a subset of those 6 values. And the resulting values are themselves combinations of the inputs listed in their rows.

Lines 79 - 93 then take the results of the `reduce` function and determines the input hashes that when XORed together results in the desired hash. It finally outputs the commands to solve the network challenge.

{language=python}
~~~~~~~~
$ ./solve.pl hashes.remote.txt | nc school.fluxfingers.net 1513
111110000101000100000000101001010101100001000001 (1)
010000101101010100000101011001100110100001000011 (2)
001000001001011000011100110010110111100011110000 (2)
000100110100001010000001100011100000000001111100 (2)
000010111010101000111100100011100000100010101001 (2)
000001011111000000100001010010100000010010000110 (2)
000000101110110111111001001111011000000000011010 (5)
000000010101001001000010110100101010010000101101 (4)
000000001000111010100001101000110000010001010100 (6)
000000000101001110001000000001000011000100100110 (5)
000000000010011001010100110010001111110110100000 (6)
000000000001001111000110011001110100010011011001 (6)
000000000000100010000110001010100011010001100111 (9)
000000000000010111000010101110101010100110010011 (7)
000000000000001011110001111110111111110111101011 (7)
000000000000000101011000111001011100000111101101 (9)
000000000000000010000011010001111011100001010001 (6)
000000000000000001010001000000101111000011100011 (6)
000000000000000000100000100111111101000011001100 (12)
000000000000000000010000101010011101011100110100 (12)
000000000000000000001011010011000010001111010111 (10)
000000000000000000000101011100110010110001010001 (10)
000000000000000000000010011100011001111111100010 (13)
000000000000000000000001001101111100111000111110 (15)
000000000000000000000000100000100000000001011110 (12)
000000000000000000000000010100010111100110100011 (15)
000000000000000000000000001001111000101101111101 (10)
000000000000000000000000000100000000110011111101 (15)
000000000000000000000000000010100111011010101100 (17)
000000000000000000000000000001001100100000001010 (12)
000000000000000000000000000000101100111011100101 (15)
000000000000000000000000000000010110110011100100 (19)
000000000000000000000000000000001010110100000011 (12)
000000000000000000000000000000000100101110001111 (12)
000000000000000000000000000000000010001100011001 (21)
000000000000000000000000000000000001000110000100 (16)
000000000000000000000000000000000000101110111110 (20)
000000000000000000000000000000000000010110010111 (18)
000000000000000000000000000000000000001011111000 (17)
000000000000000000000000000000000000000101100111 (20)
000000000000000000000000000000000000000011111111 (23)
000000000000000000000000000000000000000001010111 (20)
000000000000000000000000000000000000000000101101 (24)
000000000000000000000000000000000000000000011111 (23)
000000000000000000000000000000000000000000001111 (20)
000000000000000000000000000000000000000000000111 (22)
000000000000000000000000000000000000000000000011 (24)
000000000000000000000000000000000000000000000001 (33)
000000000000000000000000000000000000000000000001 (26)
000000000000000000000000000000000000000000000001 (25)
000000000000000000000000000000000000000000000001 (24)
000000000000000000000000000000000000000000000001 (29)
000000000000000000000000000000000000000000000001 (25)
000000000000000000000000000000000000000000000001 (27)
000000000000000000000000000000000000000000000001 (28)
000000000000000000000000000000000000000000000001 (27)
000000000000000000000000000000000000000000000001 (29)
000000000000000000000000000000000000000000000001 (25)
000000000000000000000000000000000000000000000001 (30)
000000000000000000000000000000000000000000000001 (25)
000000000000000000000000000000000000000000000001 (24)
000000000000000000000000000000000000000000000001 (23)
000000000000000000000000000000000000000000000001 (29)
000000000000000000000000000000000000000000000001 (22)
000000000000000000000000000000000000000000000001 (28)
000000000000000000000000000000000000000000000001 (26)
000000000000000000000000000000000000000000000001 (22)
000000000000000000000000000000000000000000000001 (25)
000000000000000000000000000000000000000000000001 (23)
000000000000000000000000000000000000000000000001 (25)
000000000000000000000000000000000000000000000001 (25)
000000000000000000000000000000000000000000000001 (24)
000000000000000000000000000000000000000000000001 (32)
000000000000000000000000000000000000000000000001 (27)
000000000000000000000000000000000000000000000001 (25)
000000000000000000000000000000000000000000000001 (23)
000000000000000000000000000000000000000000000001 (27)
000000000000000000000000000000000000000000000001 (23)
000000000000000000000000000000000000000000000001 (20)
000000000000000000000000000000000000000000000001 (30)
000000000000000000000000000000000000000000000001 (24)
000000000000000000000000000000000000000000000001 (23)
000000000000000000000000000000000000000000000001 (24)
000000000000000000000000000000000000000000000001 (27)
000000000000000000000000000000000000000000000001 (23)
000000000000000000000000000000000000000000000001 (24)
000000000000000000000000000000000000000000000001 (20)
000000000000000000000000000000000000000000000001 (28)
000000000000000000000000000000000000000000000001 (27)
000000000000000000000000000000000000000000000001 (27)
000000000000000000000000000000000000000000000001 (24)
000000000000000000000000000000000000000000000001 (27)
000000000000000000000000000000000000000000000001 (27)
000000000000000000000000000000000000000000000001 (29)
000000000000000000000000000000000000000000000001 (27)
000000000000000000000000000000000000000000000001 (23)
000000000000000000000000000000000000000000000001 (22)
000000000000000000000000000000000000000000000001 (25)
000000000000000000000000000000000000000000000001 (25)
000000000000000000000000000000000000000000000000 (25)
000000000000 => 785e64baad24
42d505666843 => 785e64baad24
624319ad10b3 => 785e64baad24
7101982310cf => 785e64baad24
7aaba4ad1866 => 785e64baad24
78465d90987c => 785e64baad24
78559bf7dca5 => 785e64baad24
785d1ddde8c2 => 785e64baad24
785fec261529 => 785e64baad24
785eb4c3d4c4 => 785e64baad24
785e37846c95 => 785e64baad24
785e66869c76 => 785e64baad24
785e64f70394 => 785e64baad24
785e64a67a37 => 785e64baad24
785e64b676ca => 785e64baad24
785e64bc0066 => 785e64baad24
785e64b8c86c => 785e64baad24
785e64ba0689 => 785e64baad24
785e64baab8a => 785e64baad24
785e64baae1d => 785e64baad24
785e64baace5 => 785e64baad24
785e64baad82 => 785e64baad24
785e64baad7d => 785e64baad24
785e64baad2a => 785e64baad24
785e64baad25 => 785e64baad24
login 2W
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: c2da61799d26
login LI
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 634bf1f8b707
login Og
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 755ad1d88c94
login t1
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 0dfcb16aac84
login fi
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: a19dd8de8d84
login 33
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: b78cd3d858c4
login FZ
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 9fdcf1882861
login hz
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: ff9c7cfa1179
login Tl
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: e7caf42a1e6b
login L6
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: cddf24b81f3a
login eg
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 4dfe8c200d99
login Od
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 18ec8e4295dd
login lc
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 45ccee5371d7
login k
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 491aea7723f4
login VO
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 0b3a7ff02297
login w6
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 457ef7b4aae4
login Qq
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: e66866a622fe
login sx
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: f23930a248a6
login cf
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 6a71733050bd
login wV
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 7b694326cc14
login ZW
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: f239dba0849a
login 2T
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: 74b4df72cc1c
login IG
Whoops, that didn't work out. You probably mistyped your password, so you should try again. In case you want to debug the problem, here's the difference between the correct hash and the hash of what you entered: a7b050520c44
login dW
Login successful
getflag
Okay, sure! Let me grab that flag for you.
flag{more_MATH_than_crypto_but_thats_n0t_a_CATegory}
~~~~~~~~
