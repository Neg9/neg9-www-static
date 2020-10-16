---
title: "OpenCTF 2015 - Veritable Buzz 1 (crypto 300) Writeup"
slug: "openctf-2015-veritable-buzz-1-crypto-300-writeup"
date: "2015-08-12 09:26:12.512492"
author_name: "reidb"
author_email: "reid.borsuk@gmail.com"
draft: false
toc: false
images:
---

Hint:

> Central Licensing is now hip to social media fashions! We are very trendy, with the help of Social Media Experts Group!
> [http://172.16.18.20/veritable_buzz-bbe62d344fc330ac716b8b4c955c2e68.html](http://172.16.18.20/veritable_buzz-bbe62d344fc330ac716b8b4c955c2e68.html)

You are given a website with 12 messages. In the source, each message contains a suspicious "signature" string:

{language=html}
~~~~~~~~
Students reported that students post to discussion forums more frequently and are irrevocable provided the stated conditions are met.

     <div class="signature" style="display:none;" data-sig="a0289c0fa7e87f1ab1e94b577f43691ebd70c04b0e62ca7eaaf1791983d512e7bbc843ee3a2a0430455e9f755f832ccdcd7a46d769ee43467a01453214868094ca228cb5eebc953a39fb9bbaf865f4dbe1dad9b5f9f1bed75671e0db5433f0ed" data-pubkey="pub-f4c74a1c7c00fb118e5a50c9ab966f9d.pem">
~~~~~~~~

`data-pubkey` points to a PEM file that is a Base64 encoded "public key" of some sort:

{language=python}
~~~~~~~~
-----BEGIN PUBLIC KEY-----
MHYwEAYHKoZIzj0CAQYFK4EEACIDYgAEYlsc6dc6ucsFVJavUphpKc350ISuwGUh
uD1MYO9TpbdF+KghCWkbBCDdK7lt5VKdOnYZKaIQ8n7J2kHaFQnVsk7Drh9zDL09
CDEqLYiqU9qRSd14/TCda1fAIH4vgRO1
-----END PUBLIC KEY-----
~~~~~~~~

A quick round-trip through an ASN.1 decoder (like [https://lapo.it/asn1js/](https://lapo.it/asn1js/)) reveals that the key is an ECDSA public key over the NIST P-384 curve. Simple examination of the public key doesn’t reveal anything suspicious.

Instead, the embedded signatures were examined. Through trial and error with Pure-Python ECDSA ([https://github.com/warner/python-ecdsa](https://github.com/warner/python-ecdsa)) it was determined that the signatures are valid when the supplied string is stripped of leading & trailing whitespace, hashed with SHA1, and then signed with the ECDSA private key:

{language=python}
~~~~~~~~
public_key_ec_pem = '''
   -----BEGIN PUBLIC KEY-----
   MHYwEAYHKoZIzj0CAQYFK4EEACIDYgAEYlsc6dc6ucsFVJavUphpKc350ISuwGUh
   uD1MYO9TpbdF+KghCWkbBCDdK7lt5VKdOnYZKaIQ8n7J2kHaFQnVsk7Drh9zDL09
   CDEqLYiqU9qRSd14/TCda1fAIH4vgRO1
   -----END PUBLIC KEY-----
   '''.strip()
   txt1 = "Students reported that students post to discussion forums more frequently and are irrevocable provided the stated conditions are met."
   sig1 = '''a0289c0fa7e87f1ab1e94b577f43691ebd70c04b0e62ca7eaaf1791983d512e7bbc843ee3a2a0430455e9f755f832ccdcd7a46d769ee43467a01453214868094ca228cb5eebc953a39fb9bbaf865f4dbe1dad9b5f9f1bed75671e0db5433f0ed'''.strip().decode('hex')
   public_key_ec = VerifyingKey.from_pem(public_key_ec_pem)
   print "Verify1: " + str(public_key_ec.verify(sig1, txt1))
~~~~~~~~

There are a few common errors with ECDSA, and a quick review of the signatures reveals the likely weakness…each signature supplied begins with the same 24-byte preamble:

"System Message: ERROR/3 (<string>:, line 44)"
Error in "code" directive:
maximum 1 argument(s) allowed, 2 supplied.

{language=python}
~~~~~~~~
.. code:: a0289c0fa7e87f1ab1e94b577f43691ebd70c04b0e62ca7eaaf1791983d512e7bbc843ee3a2a0430455e9f755f832ccd […]

~~~~~~~~

The first portion of any ECDSA signature is the r parameter of the computation, which is directly generated from a secret nonce created during the signing process. It is critical to the security of ECDSA that the secret nonce never be repeated, otherwise it becomes possible to calculate the private key from the duplicate signature. This is the same fatal misuse of ECDSA that caused the Playstation 3 security breach.

Antonio Bianchi of the Strange Things blog ([http://antonio-bc.blogspot.com/2013/12/mathconsole-ictf-2013-writeup.html](http://antonio-bc.blogspot.com/2013/12/mathconsole-ictf-2013-writeup.html)) provided a writeup of a challenge he created for the iCTF 2015 competition. This had exploit code for an almost identical vulnerability, with the exception of being written for the NIST P-192 curve.

Only two small modifications were necessary to adapt the code to this larger curve length. Extraction offsets of the r and s values needed to change to 0-47 and 48-95 respectively (zero-based index). Additionally, the call to Pure-Python ECDSA’s SigningKey.from_secret_exponent needed to explicitly select the new curve, otherwise an early assertion error is encountered:

{language=python}
~~~~~~~~
File "C:\Python27\lib\site-packages\ecdsa-0.13-py2.7.egg\ecdsa\keys.py", line 137, in from_secret_exponent
  assert 1 <= secexp < n
AssertionError
~~~~~~~~

Cracking the private key provided us with a candidate private key that is easily validated by performing the same attack against a second pair of signatures and verifying that they are identical:

{language=python}
~~~~~~~~
-----BEGIN EC PRIVATE KEY-----
MIGkAgEBBDAAY2gwczNuX2J5X2Y0aXJfZGljZV9yb2xsX2d1cmFudDMzZF90b19i
ZV9yQG5kMG2gBwYFK4EEACKhZANiAARiWxzp1zq5ywVUlq9SmGkpzfnQhK7AZSG4
PUxg71Olt0X4qCEJaRsEIN0ruW3lUp06dhkpohDyfsnaQdoVCdWyTsOuH3MMvT0I
MSotiKpT2pFJ3Xj9MJ1rV8Agfi+BE7U=
-----END EC PRIVATE KEY-----
~~~~~~~~

One interesting aspect of ECDSA is that private keys can be generated from arbitrary inputs. This private key has a suspicious dₐ component, as it is all within the printable character space:

"System Message: ERROR/3 (<string>:, line 71)"
Content block expected for the "code" directive; none found.

{language=python}
~~~~~~~~
.. code:: 0063683073336E5F62795F663469725F646963655F726F6C6C5F677572616E743333645F746F5F62655F72406E64306D

~~~~~~~~

ASCII decoding this value reveals the flag (a nod to [https://xkcd.com/221/](https://xkcd.com/221/)):

Flag: ch0s3n_by_f4ir_dice_roll_gurant33d_to_be_r@nd0m

Full exploit code below:

{language=python}
~~~~~~~~
  1 #! /usr/bin/env python
  2 
  3 import hashlib
  4 import binascii
  5 import sys
  6 import re
  7 import base64
  8 from socket import socket
  9 from ecdsa import SigningKey, NIST384p
 10 from ecdsa import VerifyingKey
 11 from ecdsa.numbertheory import inverse_mod
 12 
 13 def string_to_number(tstr):
 14     return int(binascii.hexlify(tstr), 16)
 15 
 16 def sha1(content):
 17     sha1_hash = hashlib.sha1()
 18     sha1_hash.update(content)
 19     hash = sha1_hash.digest()
 20     return hash
 21 
 22 def recover_key(c1,sig1,c2,sig2,pubkey):
 23       #using the same variable names as in:
 24       #http://en.wikipedia.org/wiki/Elliptic_Curve_DSA
 25 
 26       curve_order = pubkey.curve.order
 27 
 28       n = curve_order
 29       s1 = string_to_number(sig1[-48:])
 30       print "s1: " + str(s1)
 31       s2 = string_to_number(sig2[-48:])
 32       print "s2: " + str(s2)
 33       r = string_to_number(sig1[-96:--48])
 34       print "r: " + str(r)
 35       print "R values match: " + str(string_to_number(sig2[-96:--48]) == r)
 36 
 37       z1 = string_to_number(sha1(c1))
 38       z2 = string_to_number(sha1(c2))
 39 
 40       sdiff_inv = inverse_mod(((s1-s2)%n),n)
 41       k = ( ((z1-z2)%n) * sdiff_inv) % n
 42       r_inv = inverse_mod(r,n)
 43       da = (((((s1*k) %n) -z1) %n) * r_inv) % n
 44 
 45       print "Recovered Da: " + hex(da)
 46 
 47       recovered_private_key_ec = SigningKey.from_secret_exponent(da, curve=NIST384p)
 48       return recovered_private_key_ec.to_pem()
 49 
 50 
 51 def test():
 52       priv = SigningKey.generate(curve=NIST384p)
 53       pub = priv.get_verifying_key()
 54 
 55       print "Private key generated:"
 56       generatedKey = priv.to_pem()
 57       print generatedKey
 58 
 59       txt1 = "Dedication"
 60       txt2 = "Do you have it?"
 61 
 62       #K chosen by a fair roll of a 1d10000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
 63       sig1 = priv.sign(txt1, k=4444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444)
 64       print "Signature 1: " + str(sig1.encode('hex'))
 65       sig2 = priv.sign(txt2, k=4444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444)
 66       print "Signature 2: " + str(sig2.encode('hex'))
 67 
 68       print "Signature 1 verification: " + str(pub.verify(sig1, txt1))
 69       print "Signature 2 verification: " + str(pub.verify(sig2, txt2))
 70 
 71       key = recover_key(txt1, sig1, txt2, sig2, pub)
 72       print "Private key recovered:"
 73       print key
 74 
 75       print "Equality of generated & recovered keys: " + str(generatedKey == key)
 76 
 77 public_key_ec_pem = '''
 78 -----BEGIN PUBLIC KEY-----
 79 MHYwEAYHKoZIzj0CAQYFK4EEACIDYgAEYlsc6dc6ucsFVJavUphpKc350ISuwGUh
 80 uD1MYO9TpbdF+KghCWkbBCDdK7lt5VKdOnYZKaIQ8n7J2kHaFQnVsk7Drh9zDL09
 81 CDEqLYiqU9qRSd14/TCda1fAIH4vgRO1
 82 -----END PUBLIC KEY-----
 83 '''.strip()
 84 
 85 def recover():
 86       txt1 = "Students reported that students post to discussion forums more frequently and are irrevocable provided the stated conditions are met."
 87       sig1 = '''a0289c0fa7e87f1ab1e94b577f43691ebd70c04b0e62ca7eaaf1791983d512e7bbc843ee3a2a0430455e9f755f832ccdcd7a46d769ee43467a01453214868094ca228cb5eebc953a39fb9bbaf865f4dbe1dad9b5f9f1bed75671e0db5433f0ed'''.strip().decode('hex')
 88 
 89       txt2 = "But is this enough? And what new threats could be using it as a friend or fan.[2]"
 90       sig2 = '''a0289c0fa7e87f1ab1e94b577f43691ebd70c04b0e62ca7eaaf1791983d512e7bbc843ee3a2a0430455e9f755f832ccd54d4f8306fe11bd4a28a491ddf596c64cd98c93d7fa9a05acead17e42e96ed1a190a2fddd7c695b8d9bce43f221b4e1b'''.strip().decode('hex')
 91 
 92       public_key_ec = VerifyingKey.from_pem(public_key_ec_pem)
 93       print "Verify1: " + str(public_key_ec.verify(sig1, txt1))
 94       print "Verify2: " + str(public_key_ec.verify(sig2, txt2))
 95       print "curve order:", public_key_ec.curve.order
 96 
 97       key = recover_key(txt1, sig1, txt2, sig2, public_key_ec)
 98       print key
 99 
100 
101 if __name__ == "__main__":
102       print "---Performing test attack on known private key---"
103       test()
104       print
105       print "---Attempting to recover unknown key---"
106       recover()
107       print "---Done!---"
~~~~~~~~
