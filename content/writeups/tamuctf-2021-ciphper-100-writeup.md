---
title: "TAMUctf 2021 Ciphper Solution"
slug: "ciphper-writeup"
date: "2021-04-26 10:45:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[Ciphper](https://ctftime.org/task/15819) - 100 points

>    Background story: this code was once used on a REAL site to encrypt REAL data. Thankfully, this code is no longer being used and has not for a long time.
>
>    A long time ago, one of the sites I was building needed to store some some potentially sensitive data. I did not know how to use any proper encryption techniques, so I wrote my own symmetric cipher.
>
>    The encrypted content in [output.bin](https://shell.tamuctf.com/static/4ef9f712c915ad44fb2922dd5241c043/output.bin) is a well-known, olde English quote in lowercase ASCII alphabetic characters. No punctuation; just letters and spaces.
>
>    The flag is key to understanding this message.

First things first, analyze the PHP source code to find vulnerabilities.

```php
<?php

function secure_crypt($str, $key) {
  if (!$key) {
    return $str;
  }

  if (strlen($key) < 8) {
    exit("key error");
  }

  $n = strlen($key) < 32 ? strlen($key) : 32;

  for ($i = 0; $i < strlen($str); $i++) {
    $str[$i] = chr(ord($str[$i]) ^ (ord($key[$i % $n]) & 0x1F));
  }

  return $str;
}
```

The first vuln is the `0x1F`. That means that only the lowest 5 bits are encrypted. The upper 3 bits are plaintext.

```python
a = open('output.bin', 'rb').read()
b = bytes([x & 0xe0 for x in a])  
b
b'`` `` `` ``` `` `` ```` `` ``` ```````` ``````` ``` `````` `` ``` ```` `` `````` ``` `````` ``` `````` `` `````````` ``````` `` `` ```` ```` ``````` ` ``` `` ```````` ``` `` ```````` ``` ````'

```

It might be difficult to see here but \` indicates a letter and space indicates a space. That is what our vulnerability gives us. And we're going to ride this vulnerability into the sunset. Since we know the value of space is 0x20, we know the key at that point in the ciphertext. That allows us to learn the length of the key (32 bytes) and then a handful of key bytes.

```python
d = [(x,y) for x,y in zip(a,b)]           

v1 = d[:32]
v2 = d[32:64]
[(x, y) for x, y in zip(v1,v2) if x[1]==32 and y[1]==32] 
[((62, 32), (62, 32)), ((50, 32), (50, 32))]

e = [x[1] == 32 and x[0] or '' for x in d]
for i in range(len(e)//32):
    print(e[32*i:32*i+32])

['', '', 39, '', '', 59, '', '', 46, '', '', '', 47, '', '', 62, '', '', 53, '', '', '', '', 46, '', '', 50, '', '', '', 47, '']
['', '', '', '', '', '', '', 47, '', '', '', '', '', '', '', 62, '', '', '', 50, '', '', '', '', '', '', 50, '', '', 52, '', '']
['', 41, '', '', '', '', 36, '', '', 52, '', '', '', '', '', '', 57, '', '', '', 62, '', '', '', '', '', '', 57, '', '', '', 61]
['', '', '', '', '', '', 36, '', '', 52, '', '', '', '', '', '', '', '', '', '', 62, '', '', '', '', '', '', '', 48, '', '', 61]
['', '', 39, '', '', '', '', 47, '', '', '', '', 47, '', '', '', '', '', '', '', 62, '', 55, '', '', '', 50, '', '', 52, '', '']

key = [0, 41, 39, 0, 0, 59, 36, 47, 46, 52, 0, 0, 47, 0, 0, 62, 57, 0, 53, 50, 62, 0, 55, 46, 0, 0, 50, 57, 48, 52, 47, 61]

bytes([x^(key[i%32]&0x1f) for i, x in enumerate(a)])
b'so gh or nqf xc bj thnt wp the qresqdon wh{fhi~ tfs n`bl{q in thb mlcd to mgfjir {he |lipds and frrjzs of qgt~mge`us iorjvne or so qlke arsa mkaiast n s{b of trhubihs and>py,cpp`sinh epg them'

ord('r')^ord('u')
7
ord('q')^ord('t')
5
ord('d')^ord('i')
13

key = [7, 41, 39, 5, 13, 59, 36, 47, 46, 52, 0, 0, 47, 0, 0, 62, 57, 0, 53, 50, 62, 0, 55, 46, 0, 0, 50, 57, 48, 52, 47, 61]

bytes([x^(key[i%32]&0x1f) for i, x in enumerate(a)])
b'to be or nqf xc bj thnt wp the question wh{fhi~ tfs n`bl{q in the mind to mgfjir {he |lipds and arrows of qgt~mge`us iorjvne or to take arsa mkaiast n s{b of troubles and>py,cpp`sinh epg them'
```

Okay so now we know the quote is hamlet "to be or not to be" we can go on. From those key bytes we can find the rest just by filling in correct values.

```python
b'to be or not to be that is the question'

plaintext = b'to be or not to be that is the question'
key = [plaintext[i]^a[i] for i in range(32)]
key
[7, 9, 7, 5, 13, 27, 4, 15, 14, 20, 30, 18, 15, 12, 12, 30, 25, 15, 21, 18, 30, 15, 23, 14, 30, 3, 18, 25, 16, 20, 15, 29]
alphabet = 'abcdefghijklmnopqrstuvwxyz0123456789'
[alphabet[i] for i in key]
['h', 'j', 'h', 'f', 'n', '1', 'e', 'p', 'o', 'u', '4', 's', 'p', 'm', 'm', '4', 'z', 'p', 'v', 's', '4', 'p', 'x', 'o', '4', 'd', 's', 'z', 'q', 'u', 'p', '3']
''.join([alphabet[i] for i in key])
'hjhfn1epou4spmm4zpvs4pxo4dszqup3'
''.join([chr(96+i) for i in key])
'gigem{dont~roll~your~own~crypto}'
```

The flag is `gigem{dont~roll~your~own~crypto}`.

Why did I try alphabet before I tried ASCII? It seemed sensible to use an alphabet instead of using ASCII since ascii has some limitations in crypto where an arbitrary alphabet can be as small as you dare.




