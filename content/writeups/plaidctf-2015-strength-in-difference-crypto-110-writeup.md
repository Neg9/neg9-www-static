---
title: "PlaidCTF 2015 - Strength in Difference (crypto 110) Writeup"
slug: "plaidctf-2015-strength-in-difference-crypto-110-writeup"
date: "2015-04-24 12:34:34.830504"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

Hint:

> Strength in Difference
> 
> We've [captured](http://play.plaidctf.com/files/captured_827a1815859149337d928a8a2c88f89f) the flag encrypted several times... do you think you can recover it?

## Alice's Other Birthday Suit
*by Javantea*
*April 18, 2015*

## How to do this quickly?
There are many ways of doing this, the main thing is that we need to
efficiently come up with pt ** N where N starts at min(e) and goes down to 1.
The first step is to find the smallest exponent we can make using two ciphertexts.

It turns out that e8 and e17 do this.

See this:

{language=python}
~~~~~~~~
best_a = 10000
for i in range(len(es)):
      for j in range(len(es)):
              if i == j: continue
              if es[i] % es[j] > 0 and es[i] % es[j] < best_a:
              best_a = es[i] % es[j]
              print(i, j)
      #next j
#next i
~~~~~~~~

This produces best_a == 1775 and i = 8, j = 17.

This allows us to use the inverse of c17 repeatedly to create
pt ** 1775.

Once you have pt ** 1775, you need to check for the next step down.
[e % 1775 for e in es]
[1491, 1349, 994, 1704, 994, 1349, 639, 639, 426, 284, 639, 1349, 1136, 639, 1491, 71, 994, 1136, 309, 1704]

As you can see you can produce 71 but it's a slower solution because it takes
30 minutes to compute. Instead, I found that 309 produced 14 in the next step
which makes it possible for me to crack in one minute fourteen seconds. Not bad.

{language=python}
~~~~~~~~
real    1m14.278s
user    1m14.228s
sys     0m0.006s
~~~~~~~~

## How does this work at all?
How does this work at all? Inverse numbers are a consequence of RSA modular exponentiation. Think for a moment about RSA's system.

Our ciphertext is generated using:
pt ^ e mod N

In Python, that is:
pow(pt, e, N)

When someone encrypts the same plaintext twice with two different e and the same N, they are making a mistake known as Alice's Birthday. The first step to cryptanalysis of this problem is to understand inverse numbers.

{language=python}
~~~~~~~~
N = 1777515163382723970373940383272808481655635386178766060349515720868487653330
89060790975724925254690253115039605976618801961586375738876768312663570453872422
39670056994997890314957563718692456908808241405556785438717235135398736785240289
7584121327597709142354218771564925957009027121275680622544508517589219713
pow(1234, 65537, N) = 8128206764887588592803344687472673906283845270537718229072
99742290889689503074097648238837529922917688091270550885108573661729290590639685
72749155102672159754878928872471404942773680915279208060650397269487853036954557
34573555394731297070065410768330619191503306069996810062631909406787918549746451
7907093433
~~~~~~~~

Let's call this cv1.

{language=python}
~~~~~~~~
pow(1234, 32768, N) = 8433382499942535200453485969430959424326350773016047478852
59535709652825684216595862468993682546342507356888357425768135935781117605679059
66427921808278741033227004179549246417812714008944738108748509428941409405586729
13905211106957809578217163079328781701444730824115693799242555067546824357212324
0793157842
~~~~~~~~

Let's call this cv2.

Note that 32768*2 = 65536

cv2_inv = Crypto.Util.number.inverse(cv2, N)

This gives us the inverse of that number in N, just like 1/cv2 except in modular arithmetic.

so if cv1 is pt ** 65537 and cv2 is pt ** 32768, what is cv1 * cv2_inv % N ?

It should be pt ** 32769 % N
Let's call this intermediate1.

What is intermediate1 * cv2_inv % N ?

It should be pt ** 1 % N, which is simply pt. This is the end of the decryption. To see it work, run this Python code:

{language=python}
~~~~~~~~
cv2_inv = Crypto.Util.number.inverse(cv2, N)
intermediate1 = (cv1 * cv2_inv) % N
pt = (intermediate1 * cv2_inv) % N
print(pt)
~~~~~~~~

As you can see, you get the plaintext back. This doesn't work on normal RSA because valid RSA systems never encrypt with two different e, e is part of the public key, so it doesn't make sense to change it. This attack only works when you find someone encrypting the same plaintext with two different e and two same N.

So why is this solution so complex when the explanation is easy? When the exponents do not easily give you pt ** 1, you have to reduce the exponent to a number which you can then use to divide another number with. With a large number of ciphertexts and exponents, we end up with a lot of choices for which direction to go. The most obvious direction for example, required a ton of unnecessary computation to get to 1. In this solution, I take an alternative route which only takes 1 minute 14 seconds on my system.

{language=python}
~~~~~~~~
import Crypto.Util.number
import binascii

import gmpy2

"""
START FROM strength_is_weakness1.py
"""

a = open('captured_827a1815859149337d928a8a2c88f89f','r').read().split('\n')

a.pop(0)
a.pop(-1)

b = [v.replace('{','').replace('}','').replace('L','').split(' : ') for v in a]

c = []
for v in b:
    r = [int(w,16) for w in v]
    c.append(r)

cts = [v[2] for v in c]

allcts = 1
for q in cts:
    allcts *= q

n = c[0][0]
sqrtn = gmpy2.iroot(n, 2)[0]

es = [v[1] for v in c]

"""
END OF FROM strength_is_weakness1.py
START FROM alicebirthday2.py

let's assume that we can subtract off whatever we want. What do we want to subtract off?
[1517356884191, 6173920479630499, 521222775331469, 52322651179, 1731619,
736938749, 44003349641539, 146189, 1804229351, 3714908233709, 263758484593339,
122944949, 5409989577508180636511, 1224741217939, 42232455791,
1073197694407494671, 156889697747737238458819, 59711, 17249876309,
438348354053765429]

# This was too slow:
so we start with 1775.
then we get to 75.
then we get to 4.
then we get to 1.

# I actually did:
We start with 1775.
then we get to 309.
then we get to 14.
then we get to 1.

>>> [e % 1775 for e in es]
[1491, 1349, 994, 1704, 994, 1349, 639, 639, 426, 284, 639, 1349, 1136, 639, 1491, 71, 994, 1136, 309, 1704]
...

Now we just have to make said thing happen.
"""
print("First round:")
e8  = es[8]
e17 = es[17]

c8  = cts[8]
c17 = cts[17]

# e2 is big
# e1 is small
# e8 is big
# e17 is small

N=n

c17_inv = Crypto.Util.number.inverse(c17, N)

x = c8
for i in range(e8 // e17):
  reduced = (x*c17_inv) % N
  x = reduced
#next i

print("pt**%i == " % (e8 % e17), x)

print("Second round:")
ev1775 = 1775
cv1775 = x

#>>> [e % 1775 for e in es].index(71)
#15

e18 = es[18]
c18 = cts[18]
# e2 is big
# e1 is small
# e1775 is small
# e18 is big

cv1775_inv = Crypto.Util.number.inverse(cv1775, N)
x = None
x = c18
for i in range(e18 // ev1775):
  reduced = (x*cv1775_inv) % N
  x = reduced
  if i & 0xfffff == 0: print(i)
#next i

print("pt**%i == " % (e18 % ev1775), x)
# pt**309 ==  77812143691613137859642574159751661524059656531077281832519514607166612396443019877291279228946029262208452737727357399060260572705324856415477500890491548105139173030286296654630429582478615912190130048364100883562180845460100956750790149256651343993188760510224743760002627483741121653831578520450860678971

print("Fourth round:")
e5 = es[5]
c5 = cts[5]
ev309 = 309
cv309 = x
cv309_inv = Crypto.Util.number.inverse(cv309, N)
x = None
x = c5
for i in range(e5 // ev309):
  reduced = (x*cv309_inv) % N
  x = reduced
  if i & 0xfffff == 0: print(i)
#next i

print("pt**%i == " % (e5 % ev309), x)
# pt**14 ==  27886551985375180119142257698198169680785819745800876267667334683899150260379015867328461457168129787983685908863283637175608702914017729024858236190216068123162447684806625206823427949151063725365869097301347014506928254536158313067042819454111737574973705118098896158360196887284334117004352987400844142343

print("Fifth round:")

e17 = es[17]
c17 = cts[17]
ev14 = 14
cv14 = x
cv14_inv = Crypto.Util.number.inverse(cv14, N)
x = None
x = c17
for i in range(e17 // ev14):
  reduced = (x*cv14_inv) % N
  x = reduced
  if i & 0xfffff == 0: print(i)
#next i

print("pt**%i == " % (e17 % ev14), x)
# pt**1 ==  11859814987468385682904193929732856121563109146807186957694593421160017639466355

#x_inv = Crypto.Util.number.inverse(x, N)
#reduced = (c1*x_inv) % N

#pt = reduced

pt = x

print("pt == ", pt)

print("solution == ", binascii.unhexlify(hex(pt)[2:]))
# solution ==  b'flag_Strength_Lies_In_Differences'
~~~~~~~~
