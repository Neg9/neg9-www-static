---
title: "Boston Key Party 2015 - Airport (crypto 500) Writeup"
slug: "boston-key-party-2015-airport-crypto-500-writeup"
date: "2015-03-08 17:33:02.390443"
author_name: "coldwaterq"
author_email: "coldwaterq@gmail.com"
draft: false
toc: false
images:
---

by [ColdwaterQ](http://coldwaterq.com/2015/03/07/Boston-Key-Party-Airport-Crypto-500.html)

The challenge that I found the most enjoyable, and as such wanted to write about from the Boston Key Party was Airport (Crypto 500). This challenge's hint made it clear that the goal was to do some kind of timing attack. It said:

> The timing in this challenge is clearly not very realistic---but the methods you'll use here can be extended to real-world implementations of modular exponentiation.  Server at 52.1.245.61, port 1025.  My solution takes a little while.  Good luck.

Along with the hint, the code which is being run on the server was also supplied, and so the challenge was to figure out how to get the code to progress down the correct path to return the flag. The full source can bee seen at the bottom of this page if you want to view it, and I will include snippets that go with what I am talking about throughout this post.

The three methods that pop out first are exponentiate, handle, and captcha.

{language=python}
~~~~~~~~
def handle(self):
    self.captcha()
    while True:
        self.exponentiate()
    self.request.close()

def captcha(self):
    proof = base64.b64encode(os.urandom(9))
    self.request.sendall(proof)
    test = self.request.recv(20)
    ha = hashlib.sha1()
    ha.update(test)
    if test[0:12]!=proof or not ha.digest().endswith('\xFF\xFF\xFF'):
        self.fail("You're a robot!")

def exponentiate(self):
    base = int(self.request.recv(1024))
    if not group_check(SAFEPRIME, base):
        self.fail("Bad group element.")
    result = slowpower(base, self.SECRET, SAFEPRIME)
    if result != 4:
        self.request.sendall(str(result))
    else:
        self.request.sendall(FLAG)
~~~~~~~~

The exponentiate method returns the flag, so that is what I need to get called, handle calls captcah then exponentiate over and over, so I need captcha to succeed, then I can focus on exponentiate.

Captcha causes the server to send us 12 bytes, which are 9 random bytes base64 encoded. We must take the 12 bytes and determine what we can add to it to cause a sha1 hash of the whole thing to end in '\xFF\xFF\xFF'. The whole blob that generates the hash then needs to be sent back to the server. If the blob works, we continue, otherwise the connection is closed. We used the following to bypass this hurdle, there are probably better ways to do it, but this was good enough and let us move onto the real problem.

{language=python}
~~~~~~~~
sha_beginning = s.recv(12)
def hash(start, depth):
    for i in range(0,256):
        if depth == 0:
            ha = hashlib.sha1()
            ha.update(start+chr(i))
            sha1 = ha.digest()
            if sha1.endswith('\xff\xff\xff'):
                return start+chr(i)
        else:
            ret = hash(start+chr(i), depth - 1)
            if ret != None:
                return ret
    return None

answer = hash(sha_beginning, 6)
ha = hashlib.sha1()
ha.update(answer)
s.send(answer)
~~~~~~~~

Now the real work begins of getting exponentiate to return the flag. The code bellow is all of the code necessary to understand to solve the rest of the problem.

{language=python}
~~~~~~~~
SAFEPRIME = long('2732739539206515653529570898678620485107952883772378051013610'
                 '2615658941290873291366333982291142196119880072569148310240613'
                 '2945256014230863856845399875300416857467228021433971569771965'
                 '3602207834524916297731283755544484088530470449762224316003634'
                 '4118163834102383664729922544598824748665205987742128842266020'
                 '6443185353981585292316703655331307185593642395133761905803319'
                 '3832373989579164842980448941700010567781724874144618468982851'
                 '2402512984453866089594767267742663452532505964888865617589849'
                 '6838094168057269743494744279786917408337533269627601147449670'
                 '9365254180899938977334631729447374243951032681130003108058261'
                 '8145727L')

def exponentiate(self):
    base = int(self.request.recv(1024))
    if not group_check(SAFEPRIME, base):
        self.fail("Bad group element.")
    result = slowpower(base, self.SECRET, SAFEPRIME)
    print result
    if result != 4:
        self.request.sendall(str(result))
    else:
        self.request.sendall(FLAG)

def slowpower(base, power, mod):
    accum = 1
    for bit in bin(power)[2:]:
        if accum == 4:
            time.sleep(1.0)
        accum = accum*accum % mod
        if bit == '1':
            accum = accum*base % mod
    return accum

self.SECRET = rand_exponent(SAFEPRIME)

while True:
    self.exponentiate()
~~~~~~~~

Exponentiate is called over and over as long as group_check does not fail. I did not include group_check because once I figured out how to solve slowpower all my answers went past group_check as well. So The goal is to provide a number that will cause slowpower to return a 4. The self.Secret is generated for each connection so that is an unknown, but SAFEPRIME is a very large static prime number which was included in the server source.

Looking at slowpower we see that a variable accum starts at one. Then each bit of the secret is enumerated. And for each bit accum is squared, and then if the bit is one accum is also multiplied by base. Also every operation done on accum is modded by SAFEPRIME. Since only multiplication is happening in the loop, all of the mod operations can be pulled outside the loop, and the result will be the same. It is also important to realize that when accum is 4 the function sleeps for a second. After analyzing the function it becomes clear that the goal is to figure out the secret through repeated guesses that take advantage of the sleep so that eventually we have a number that will result in 4 being returned and us winning.

To solve this we considered base to be x, and thought about what various numbers as the secret would mean for x. Bellow is a table that show secrets compared to the value of accum in terms of x. Remember that since mod SAFEPRIME has been pulled outside the loop we don't have to consider that for now. We are just considering what the multiplication means.

What the pattern is, is that for every bit added to bin(secret) the exponent is doubled from the previous exponent. And if the bit is a one the exponent has one added to it on top of being doubled. So in order to take advantage of the sleep(1.0) we start with checking if the first bit is one or zero. If the first bit is one then the x where x % SAFEPRIME = 4 will cause the program to wait a second.

Since we know bit one is always 1 we can continue to bit two. If bit two is 0 then x where x² % SAFEPRIME = 4 will cause the program to wait a second, and if that bit was one then the x where x³ % SAFEPRIME = 4 will cause the program to wait. Assuming it was one, then if the x where x⁶ % SAFEPRIME = 4 causes the program to wait then the third bit is 0, otherwise it is one. This can be continued to find all the bits of the secret, and more importantly for us, the last value of x will cause the server to return the flag.

Now since there are many bits in the secret we need a programmatic way to solve for x given n such that xⁿ % SAFEPRIME = 4. Keep in mind that SAFEPRIME is a constant so there is one variable we set, and one we solve for.

This is not an easy function to solve though, and we had no idea how to slolve this programmatically but after much googling we found this post on xkcd. [http://forums.xkcd.com/viewtopic.php?f=17&t=73708#p3148049](http://forums.xkcd.com/viewtopic.php?f=17&t=73708#p3148049). It states the following:

> Let y = g^x mod m
> 
> Let ux = 1 mod φ(m), where φ() is Euler's totient function.
> 
> Then g = y^u mod m

In this case we want to solve for g, and so if we can find u we have our solution. So we just need a porgramtaic way to solve the second function for u and we have the answer. As it happens the second function is a Modular multiplicative inverse and had to be solved for a different challenge in the CTF so we took the code for that. Then a bit more Google to find that φ(m) when m is prime is m-1. Put this all together and we get the following code.

{language=python}
~~~~~~~~
def find_val(xs):
    return pow(4 ,modinv(xs, p-1), p)

def modinv(a, m):
    g, x, y = egcd(a, m)
    if g != 1:
        raise Exception('modular inverse does not exist')
    else:
        return x % m

def egcd(a, b):
    if a == 0:
        return (b, 0, 1)
    else:
        g, y, x = egcd(b % a, a)
        return (g, x - (b // a) * y, y)
~~~~~~~~

In this code p is SAFEPRIME, and calling find_val(xs) where xs is the exponent will return the value of what we called x but which was called g in xkcd, which is what we need to send to the server. One problem though, if xs is even the code fails. So we could only check odd numbers. Or in other words, we could only check for bits that are 1. Luckily if the bit isn't 1 we know it is 0 so that is no big deal.

The whole attack script works as follows. We break captcha with shear strength. Then we start with the exponent of 3, since we already know that bit 1 is a one. So we solve x³ % SAFEPRIME = 4 for x. We send that x to the server and if the server waits a second, then we know the second bit is a 1, if the server doesn't wait a second we know the second bit is a 0. If the bit was a one we multiply the exponent by two and add one, and repeat. If the bit was a 0 we subtract one, multiply the exponent by two, and add one. In this way the exponent never has to be even, and we are able to figure out what the correct exponent should be one bit at a time. The code had to request things that seemed to take one second twice because of the occasional false positive from lag, and we had to increase the size of the stack for the find_num function to work near the end. The code that is solving for the correct exponent is below, and is combined with the cpatcha smasher to make our solution.

{language=python}
~~~~~~~~
xs = 3
first = False
while cont:
    t = time.time()
    s.send(str(find_val(xs)))
    ret = s.recv(9999)
    print ret
    if '\n' in ret:
        cont = False
    a = time.time()-t
    print str(a)+' '+str(xs)
    if a>1:
        if first:
            xs = xs*2+1
            first = False
        else:
            first = True
    else:
        first = False
        xs = (xs - 1)*2+1
~~~~~~~~

After letting this code run for a decent while we had the password which was:

{language=python}
~~~~~~~~
diffie_hellman_im_awfully_fond_of_youuuuuu
~~~~~~~~

The code given to us by the CTF and the code we wrote are both bellow in full.

## Provided Source
{language=python}
~~~~~~~~
 1 #!/usr/bin/python
 2 
 3 import time
 4 import random
 5 
 6 def slowpower(base, power, mod):
 7     accum = 1
 8     for bit in bin(power)[2:]:
 9         if accum == 4:
10             time.sleep(1.0)
11         accum = accum*accum % mod
12         if bit == '1':
13             accum = accum*base % mod
14     return accum
15 
16 
17 SAFEPRIME = 27327395392065156535295708986786204851079528837723780510136102615658941290873291366333982291142196119880072569148310240613294525601423086385684539987530041685746722802143397156977196536022078345249162977312837555444840885304704497622243160036344118163834102383664729922544598824748665205987742128842266020644318535398158529231670365533130718559364239513376190580331938323739895791648429804489417000105677817248741446184689828512402512984453866089594767267742663452532505964888865617589849683809416805726974349474427978691740833753326962760114744967093652541808999389773346317294473742439510326811300031080582618145727L
18 # generated with gensafeprime.generate(2048)
19 # https://pypi.python.org/pypi/gensafeprime
20 
21 GENERATOR = 4
22 
23 FLAG = ''
24 
25 def rand_exponent(p):
26     return random.randrange(1, (p - 1)/2)
27 
28 def group_check(p, m):
29     if m <= 0:
30         return False
31     elif m >= p:
32         return False
33     elif pow(m, (p-1)/2, p) != 1:
34         return False
35     else:
36         return True
37 
38 
39 import base64, SocketServer, os, sys, hashlib
40 
41 class ServerHandler(SocketServer.BaseRequestHandler):
42 
43     def fail(self, message):
44         self.request.sendall(message + "\nGood-bye.\n")
45         self.request.close()
46         return False
47 
48     def captcha(self):
49         proof = base64.b64encode(os.urandom(9))
50         self.request.sendall(proof)
51         test = self.request.recv(20)
52         ha = hashlib.sha1()
53         ha.update(test)
54         if test[0:12]!=proof or not ha.digest().endswith('\xFF\xFF\xFF'):
55             self.fail("You're a robot!")
56 
57     def exponentiate(self):
58         base = int(self.request.recv(1024))
59         if not group_check(SAFEPRIME, base):
60             self.fail("Bad group element.")
61         result = slowpower(base, self.SECRET, SAFEPRIME)
62         if result != 4:
63             self.request.sendall(str(result))
64         else:
65             self.request.sendall(FLAG)
66 
67     def setup(self):
68         self.SECRET = rand_exponent(SAFEPRIME)
69 
70     def handle(self):
71         self.captcha()
72         while True:
73             self.exponentiate()
74         self.request.close()
75 
76 
77 class ThreadedServer(SocketServer.ForkingMixIn, SocketServer.TCPServer):
78     pass
79 
80 if __name__ == "__main__":
81     HOST = sys.argv[1]
82     PORT = int(sys.argv[2])
83 
84     FLAG = open('flag.txt', 'r').read()
85 
86     server = ThreadedServer((HOST, PORT), ServerHandler)
87     server.allow_reuse_address = True
88     server.serve_forever()
~~~~~~~~

## Personal source
{language=python}
~~~~~~~~
 1 import socket
 2 import hashlib
 3 import time
 4 import sys
 5 # p is the SAFEPRIME, but it is so big it gets anoying
 6 from test import p
 7 sys.setrecursionlimit(100000)
 8 
 9 def egcd(a, b):
10     if a == 0:
11         return (b, 0, 1)
12     else:
13         g, y, x = egcd(b % a, a)
14         return (g, x - (b // a) * y, y)
15 
16 def modinv(a, m):
17     g, x, y = egcd(a, m)
18     if g != 1:
19         raise Exception('modular inverse does not exist')
20     else:
21         return x % m
22 
23 def find_val(xs):
24     return pow(4 ,modinv(xs, p-1), p)
25 
26 cont = True
27 while cont:
28     s = socket.create_connection(('52.1.245.61',1025))
29 #    s = socket.create_connection(('localhost',8000))
30     sha_beginning = s.recv(12)
31     def hash(start, depth):
32         for i in range(0,256):
33             if depth == 0:
34                 ha = hashlib.sha1()
35                 ha.update(start+chr(i))
36                 sha1 = ha.digest()
37                 if sha1.endswith('\xff\xff\xff'):
38                     return start+chr(i)
39             else:
40                 ret = hash(start+chr(i), depth - 1)
41                 if ret != None:
42                     return ret
43         return None
44 
45     answer = hash(sha_beginning, 6)
46     print 'sent:'
47     print answer.encode('hex')
48     s.send(answer)
49     xs = 3
50     first = False
51     while cont:
52         t = time.time()
53         s.send(str(find_val(xs)))
54         ret = s.recv(9999)
55         print ret
56         if '\n' in ret:
57             cont = False
58         a = time.time()-t
59         print str(a)+' '+str(xs)
60         if a>1:
61             if first:
62                 xs = xs*2+1
63                 first = False
64             else:
65                 first = True
66         else:
67             first = False
68             xs = (xs - 1)*2+1
69     s.close()
~~~~~~~~
