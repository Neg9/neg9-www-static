---
title: "TAMUctf 2021 Basic RSA Solution"
slug: "basic-rsa-writeup"
date: "2021-04-26 10:35:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[Basic RSA](https://ctftime.org/task/15786)

    To: Dr Rivest
    CC: Dr Shamir
    
    We found this code and these numbers. [data.txt](https://shell.tamuctf.com/static/73d837aecbac3ac12ab50aa6dc1b9ab6/data.txt)
    Mean anything to you?
    
    Sincerely
    - Dr Adleman

```sh
cat data.txt 
N = 2095975199372471
e = 5449
gigem{ 875597732337885 1079270043421597 616489707580218 2010079518477891 1620280754358135 616660320758264 86492804386481 171830236437002 1500250422231406 234060757406619 1461569132566245 897825842938043 2010079518477891 234060757406619 1620280754358135 2010079518477891 966944159095310 1669094464917286 1532683993596672 171830236437002 1461569132566245 2010079518477891 221195854354967 1500250422231406 234060757406619 355168739080744 616660320758264 1620280754358135 }

factor 2095975199372471
2095975199372471: 21094081 99363191
```

We were able to factor the coprime N with the `coreutils` program `factor`. This gives us the ability to decrypt anything encrypted with RSA. There is another solution for if the coprime N was actually large enough that we couldn't factor it, but I won't go into that.

```python
p,q = 21094081, 99363191
N = 2095975199372471
e = 5449
a = [int(x) for x in '875597732337885 1079270043421597 616489707580218 2010079518477891 1620280754358135 616660320758264 86492804386481 171830236437002 1500250422231406 234060757406619 1461569132566245 897825842938043 2010079518477891 234060757406619 1620280754358135 2010079518477891 966944159095310 1669094464917286 1532683993596672 171830236437002 1461569132566245 2010079518477891 221195854354967 1500250422231406 234060757406619 355168739080744 616660320758264 1620280754358135'.split()]
phi = (p-1)*(q-1)
import Crypto.Util.number
Crypto.Util.number.inverse(e, phi)
384653161849
d = Crypto.Util.number.inverse(e, phi)
b = [pow(x, d, N) for x in a]
b
[82, 83, 65, 95, 115, 51, 99, 117, 114, 49, 116, 121, 95, 49, 115, 95, 52, 98, 48, 117, 116, 95, 112, 114, 49, 109, 51, 115]
bytes(b)
b'RSA_s3cur1ty_1s_4b0ut_pr1m3s'
```

As you can see, we are able to get the decryption exponent `d` using the inverse of the encryption exponent `e` with the value phi, which is just `(p - 1) * (q - 1)`. Because `p` and `q` are private, `phi` is secured by the factorization of integers. The larger the integer, thee more difficult to factor `phi`. Unfortunately, this is by no means the only known attack against RSA.

The flag is `gigem{RSA_s3cur1ty_1s_4b0ut_pr1m3s}`



