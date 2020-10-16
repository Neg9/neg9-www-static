---
title: "OpenCTF 2015 - signfastest (crypto 100) Writeup"
slug: "openctf-2015-signfastest-crypto-100-writeup"
date: "2015-08-12 09:14:52.728771"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

Signfaster is a challenge from OpenCTF. It's an RSA challenge that forces you to prove your understanding of RSA's stranger features. We managed to solve the challenge in an unintended manner, which the contest organizers then released as a new challenge named "signfastest". The details of that solve are what follows.

Hint (from signfaster):

> We have new signature requirements from Central Licensing.
> 
> We're happy to comply, but,...it's a bit slow. Can you figure it out?
> 
> Service: 10.0.66.121:8008 Reference Client: [172.16.18.20/sign_faster_client-d57fe60ab04625921121656d3695beca](https://neg9.org/resources/media/openctf-2015-signfastest-crypto-100-writeup/sign_faster_client-d57fe60ab04625921121656d3695beca)

ReidB provided the insight that allowed this unintended solution.

The organizers had found and patched this bug during testing, but pushed a vulnerable version to the competition server without testing. This is a very realistic scenario and shows how you should always test obvious attacks.

In RSA, decryption and signing is a slow process due to the large decryption exponent. By making the decryption exponent small, you weaken encryption and signing done using that key, making it trivial for an attacker to compromise the private key. Therefore it should never be done in practice. In this CTF, though it is done to solve a challenge.

Modify the client to use an encryption exponent of 1. e = 1.

This makes the decryption exponent equal 1. d = 1.

This means the pow() in sign() is exponent 1, very easy and fast.

The complete solution script can be found [here](https://neg9.org/resources/media/openctf-2015-signfastest-crypto-100-writeup/open_ctf-2015-crypto-100-signfastest.py) but the part relevant to this math can be found in the method `do_pubkey`:

{language=python}
~~~~~~~~
p, q = Crypto.Util.number.getPrime(2048), Crypto.Util.number.getPrime(2048)
e = 1
N = p*q
d = Crypto.Util.number.inverse(e, (q-1)*(p-1))
self.rsa = RSA.construct((long(N), long(e), long(d), long(p), long(q)))
pub = self.rsa.publickey()
self.state = 'challange'
self.sendLine(b64e(pub.exportKey(format='DER')))
~~~~~~~~

The output of running the script:

{language=python}
~~~~~~~~
python client.py 10.0.66.121 8008
Connected
lineReceived preauth    Please burn some cpu cycles first: mBYA6mEDFtBru1vB 8589934
Doing proof of work...
sendLine     pubkey     1439005603000000196
lineReceived pubkey     Public key?
Generating RSA key...
sendLine     challange  MIICIDANBgkqhkiG9w0BAQEFAAOCAg0AMIICCAKCAgEAkH6/ks7QfVpyk5/phrMd0r7xDVPnDtGgJForRMUd+eRRPFzgRymcNudepcGedAsm7Ckr8ERkgd/KFv3h5g2D55N2ac/vJb2COS27oR8516z9QnWVg6XlTEBpfyR5QfK7xJjzjy2bbXqfgp0N5Rn6qqISVTclffiFBoCS0P+Q5WPLPv6YlmJZLVmzl4BjAODfWY6jExdsB94KBZfYtYrTc+TgtyfKFp/Vf2hxcZ9Y3lircEHFxH5mNwzIGIYuUL8KWZ9ESlxOWZHVt8CC0RSZy35D+wgLX9tVV0l8xQ3JUnRtOr7J2Al2wrpt8Z0sgMpzmsHhoyXv0onxjd/UEYG4Oinre2vtsdpUPPw1qj8HKUisz+v0qo2p5tHnz0aCLZceITMKjGgsYphXsSG2kOZjr2XF+1ybcaebPRBIyO/Wfip326mCt+cJpVAP4juKETM/Qqb3Nt5gQGCRn+x6U9bo4zbpeEasywdcKwT/eXRCz9DPWckM6d/FsSWhgxQYJfrjenO+ewI3Ar78UQGyXJA8eX7aGw1TfD4r1dbuX31P5IT2YzoyAb7nkl7LrFcjdgSDfLaElyZimMySldHebhK5H8wWu/K17U+wyRmU8kyOBWWadc58R5HsalSQMn1JK9KH9vuRPXICuuVTT+4bLi11dcarPpjODH7mpXweXa4Ry8ECAQE=
lineReceived challange  Okay, make it quick, your challange is: 9Yq1dnwaa+K8ha5AKt78xOVoQ+oARJJllMAWPGP5yL4=
('do challenge!', '9Yq1dnwaa+K8ha5AKt78xOVoQ+oARJJllMAWPGP5yL4=')
0.023631811142
sendLine     print      9Yq1dnwaa+K8ha5AKt78xOVoQ+oARJJllMAWPGPegg0=
lineReceived print      Solution verified sucessfully!
This is what they gave us: 'Solution verified sucessfully!'
lineReceived print      You took 0.025278 seconds.
This is what they gave us: 'You took 0.025278 seconds.'
lineReceived print      400 points. Your flag is: 12_primes_ARE_better_than_2
This is what they gave us: '12_primes_ARE_better_than_2'
lineReceived print      Goodbye!
This is what they gave us: 'Goodbye!'
~~~~~~~~

Because this method of solving the 200-point "signfaster" problem was not intended by the event organizers, we were awarded 200 points for solving the challenge in this way, and the organizers created a new challenge (the 100-point "signfastest") out of it. This solution earned Neg9 200 points out of our total 2710 (7.4%).
