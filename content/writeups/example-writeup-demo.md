---
title: "HipCTF: Example Writeup Demo"
date: 2020-10-12T22:10:55-07:00
draft: false
---

Imagine this were a real CTF challenge **writeup**, how wonderful that would be!

We can see here, that the code on _line 7_ is doing all the magic:

```python {linenos=table,hl_lines=[7]}
with open('qr.xor', "rb") as f1:
    with open('HACKER.TXT', "rb") as f2:
        bytes1 = f1.read()
        xor = bytearray()
        for byte in bytes1:
            ord1 = ord(byte)
            ord2 = ord(f2.read(1))
            xor.append(ord1 ^ ord2)
        fout = open('hackerqr', 'wb')
        fout.write(xor)
```
