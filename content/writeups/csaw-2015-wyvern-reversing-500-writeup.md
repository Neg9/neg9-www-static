---
title: "CSAW 2015 - wyvern (reversing 500) Writeup"
slug: "csaw-2015-wyvern-reversing-500-writeup"
date: "2015-10-14 11:48:22.952414"
author_name: "tecknicaltom"
author_email: "tecknicaltom@neg9.org"
draft: false
toc: false
images:
---

Hint:

> There's a dragon afoot, we need a hero. Give us the dragon's secret and we'll give you a flag.

Running this program, it looks like it's going to be a fairly standard crackme:

{language=python}
~~~~~~~~
$ ./wyvern_c85f1be480808a9da350faaa6104a19b
+-----------------------+
|    Welcome Hero       |
+-----------------------+

[!] Quest: there is a dragon prowling the domain.
      brute strength and magic is our only hope. Test your skill.

Enter the dragon's secret: no

[-] You have failed. The dragon's power, speed and intelligence was greater.
~~~~~~~~

Taking a look at it in Hex-Rays, we see that it's a C++ program that calls `fgets` to input the string, then creates a `std::string` out of it, before calling `start_quest` to see if you've input the right thing.

{language=cpp}
~~~~~~~~
std::operator<<<std::char_traits<char>>(
  &std::cout,
  (unsigned int)"Enter the dragon's secret: ",
  "Enter the dragon's secret: ");
fgets(&s, 257, stdin);
std::allocator<char>::allocator(&v8, 257LL);
std::string::string(&v9, &s, &v8);
std::allocator<char>::~allocator(&v8);
std::string::string((std::string *)&v7, (const std::string *)&v9);
v3 = start_quest((std::string *)&v7);
std::string::~string((std::string *)&v7);
if ( v3 == 0x1337 )
{
  std::string::string((std::string *)&v6, (const std::string *)&v9);
  reward_strength((unsigned __int64)&v6);
  std::string::~string((std::string *)&v6);
}
else
{
  std::operator<<<std::char_traits<char>>(
    &std::cout,
    (unsigned int)"\n[-] You have failed. The dragon's power, speed and intelligence was greater.\n",
    v4);
}
~~~~~~~~

The `start_quest` method does all kinds of stuff with `std::vector`s, the input string, and some static buffers. It also calls another method called `sanitize_input` that looks even less fun to try to make sense of.

I hate reversing compiled C++. Let's see if we can find any shortcuts to make this less painful... Running the program through ltrace, we can see that one of the last things it does (given this input) before cleaning up is to call `std::string::length() const` which returns the length of the input string:

{language=python}
~~~~~~~~
$ echo no | ltrace -C ./wyvern_c85f1be480808a9da350faaa6104a19b 3>&1 >/dev/null 2>&3 | tail
memmove(0x8d1100, "d\0\0\0\326\0\0\0\n\001\0\0q\001\0\0\241\001\0\0\017\002\0\0n\002\0\0\335\002\0\0"..., 64) = 0x8d1100
operator delete(void*)(0x8d10b0, 0x6102f8, 0x7fff1b5eb0a0, 0x8d10b0) = 0
std::string::length() const(0x7fff1b5eb668, 1, 0xffffffff, 0) = 3
std::basic_string<char, std::char_traits<char>, std::allocator<char> >::~basic_string()(0x7fff1b5eb668, 0xffffffff, 0, 0) = 1
std::basic_ostream<char, std::char_traits<char> >& std::operator<< <std::char_traits<char> >(std::basic_ostream<char, std::char_traits<char> >&, char const*)(0x6101e0, 0x40e75c, 1, 0) = 0x6101e0
std::basic_string<char, std::char_traits<char>, std::allocator<char> >::~basic_string()(0x7fff1b5eb688, 0x40e700, 0x7fec595127d8, 0) = 0x8d1070
std::ios_base::Init::~Init()(0x610314, 0, 224, 0x7fec58d08d50) = 3
operator delete(void*)(0x8d1100, 0x6102f8, 0x7fff1b5eb510, 0x8d1100) = 1
std::ios_base::Init::~Init()(0x610310, 0, 160, 0x7fec58d08d10) = 0x7fec5952a100
+++ exited (status 0) +++
~~~~~~~~

I'm gonna bet that the crackme is doing an initial length check and bailing since the input isn't the right length. To figure out the correct length, run the program under GDB, put a breakpoint on `std::string::length() const` and finish to get back to the calling scope.

{language=python}
~~~~~~~~
   0x40469c <_Z11start_questSs+844>:  call   0x400f50 <_ZNKSs6lengthEv@plt>
=> 0x4046a1 <_Z11start_questSs+849>:  sub    rax,0x1
   0x4046a7 <_Z11start_questSs+855>:  mov    r9d,DWORD PTR ds:0x610138
   0x4046af <_Z11start_questSs+863>:  sar    r9d,0x2
   0x4046b3 <_Z11start_questSs+867>:  movsxd rcx,r9d
   0x4046b6 <_Z11start_questSs+870>:  cmp    rax,rcx
~~~~~~~~

The call to the length method is at 0x4046a1. Then it subtracts one, presumably to remove the newline from the count, readies another value in `rcx` before comparing the string length to the value in `rcx` at 0x4046b6. Stepping down to the `cmp` and looking at the registers, we find the desired input length:

{language=python}
~~~~~~~~
RAX: 0x2  # input length
RCX: 0x1c # desired input length
~~~~~~~~

So the input to pass the crackme is 28 (0x1c) characters long.

At this point, I had a hunch that the program was checking the input character-by-character and I may be able to use a sidechannel to derive the password incrementally. I had never really done much with instruction counting, but a quick Google search showed that the callgrind tool in Valgrind would do the trick. A quick test confirmed my hunch and showed that the password began with "d":

{language=python}
~~~~~~~~
$ for a in {a..z} {A..Z} ; do echo $a $(echo ${a}Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8 | valgrind --tool=callgrind ./wyvern_c85f1be480808a9da350faaa6104a19b 2>&1 | grep Collected) ; done
a ==13480== Collected : 1511690
b ==13484== Collected : 1511690
c ==13488== Collected : 1511690
d ==13492== Collected : 1522020
e ==13499== Collected : 1511690
f ==13503== Collected : 1511690
g ==13507== Collected : 1511690
h ==13511== Collected : 1511690
i ==13518== Collected : 1511690
j ==13522== Collected : 1511690
k ==13526== Collected : 1511690
l ==13530== Collected : 1511690
m ==13534== Collected : 1511690
n ==13538== Collected : 1511690
o ==13542== Collected : 1511690
p ==13548== Collected : 1511690
q ==13552== Collected : 1511690
r ==13556== Collected : 1511690
s ==13560== Collected : 1511690
t ==13564== Collected : 1511690
u ==13568== Collected : 1511690
v ==13572== Collected : 1511690
w ==13576== Collected : 1511690
x ==13580== Collected : 1511690
y ==13584== Collected : 1511690
z ==13588== Collected : 1511690
A ==13592== Collected : 1511690
B ==13596== Collected : 1511690
C ==13600== Collected : 1511690
D ==13604== Collected : 1511690
E ==13608== Collected : 1511690
F ==13614== Collected : 1511690
G ==13618== Collected : 1511690
H ==13622== Collected : 1511690
I ==13626== Collected : 1511690
J ==13630== Collected : 1511690
K ==13634== Collected : 1511690
L ==13638== Collected : 1511690
M ==13642== Collected : 1511690
N ==13646== Collected : 1511690
O ==13650== Collected : 1511690
P ==13654== Collected : 1511690
Q ==13658== Collected : 1511690
R ==13662== Collected : 1511690
S ==13666== Collected : 1511690
T ==13670== Collected : 1511690
U ==13674== Collected : 1511690
V ==13680== Collected : 1511690
W ==13684== Collected : 1511690
X ==13688== Collected : 1511690
Y ==13692== Collected : 1511690
Z ==13696== Collected : 1511690
~~~~~~~~

So I threw together the following solver script. For some unknown reason, callgrind occasionally gives a different value than expected. If the solver sees more than one outlier for a given character of the password, it throws away that loop's values and tries again.

"System Message: ERROR/3 (<string>:, line 148)"
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

  my $len = 28;
  my $prefix = '';

  while(length($prefix) < $len)
  {
        my %values = ();
        for my $char_val ( 0 .. 255 )
        {
                my $char = chr($char_val);
                next unless($char =~ /[[:print:]]/);
                my $pw = $prefix . $char;
                $pw .= '-' x ($len - length($pw));
                $pw =~ s/'/'"'"'/g;
                my $result = `echo '$pw' | valgrind --tool=callgrind ./wyvern_c85f1be480808a9da350faaa6104a19b 2>&1 | grep Collected`;
                say $pw;
                if($result =~ /Collected : (\d+)/)
                {
                        push @{$values{$1}}, $char;
                }
        }
        my @possibilities = map { $_->[0] } grep { scalar(@$_) == 1 } values %values;
        if(scalar(@possibilities) != 1)
        {
                print Dumper(\@possibilities);
                say "not a single outlier";
        }
        else
        {
                $prefix .= $possibilities[0];
        }
  }
  say "Password: $prefix";

~~~~~~~~

The script runs for a while, printing out all the passwords it attempts. As an aside, I absolutely love TV-style hacking scripts that find the answer character-by-character. The script eventually settles on the flag:

"System Message: ERROR/3 (<string>:, line 195)"
Error in "code" directive:
maximum 1 argument(s) allowed, 2 supplied.

{language=python}
~~~~~~~~
.. code:: Password: dr4g0n_or_p4tric1an_it5_LLVM
~~~~~~~~
