---
title: "OpenCTF 2018 - stegoxor Writeup"
slug: "openctf-2018-stegoxor-writeup"
date: "2018-08-20 10:32:35.384529"
author_name: "banAnna"
author_email: "astenwick@gmail.com"
draft: false
toc: false
images:
---

This challenge came with a single jpg file (renamed stego.jpg) that was an unremarkable picture of a computer screen. To this file I ran:

"System Message: ERROR/3 (<string>:, line 3)"
Error in "code" directive:
maximum 1 argument(s) allowed, 5 supplied.

{language=python}
~~~~~~~~
.. code:: $ steghide –extract -sf stego.jpg

~~~~~~~~

using an empty passphrase when prompted. This produced a files.tar archive, which I then extracted 2 files from:

{language=python}
~~~~~~~~
HACKER.TXT: text file

qr.xor. data file
~~~~~~~~

At first I wasn’t sure what to do with HACKER.TXT, when I opened it with a text editor, some characters could not be displayed. Was this a clue, a diversion, or by design?

The qr.xor file seemed like I should do something to it in order to get a QR code and the obvious choice was an xor operation. So I scripted up something to read in all the bytes of the file, xor them with the key and then write them all back out. Since I didn’t know what the key was, I looped thru 0x00 – 0xFF, however this did not produce any results. Next, I decided to xor the 2 files (qr.xor and HACKER.TXT) against each other, byte by byte (at least up to the last byte of the smallest file) and then write out the results to another file. The following is the python script that I used:

{language=python}
~~~~~~~~
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
~~~~~~~~

This produced a QR code in jpg format. I pulled up a QR reader on the web and read in my new file to decode. The decoding was a long string of base64. With this string I opened up a python terminal and ran the following in python:

{language=python}
~~~~~~~~
import base64

s = '<insert base64>'
s_out = base64.b64decode(s)
f = open('new', "wb")
f.write(s_out)
f.close()
~~~~~~~~

This produced another QR code in a png file format. So again I went back to the QR online code reader and read in this new.png file, which again gave me a base64 string. With this string I followed the same method as above and produced a new file with the output.

This new file (file3), was simply a text file, so I ‘cat’ted it to the screen. This produced another QR code, this one done as ascii art. Now this produced a problem, because you couldn’t exactly send ascii art to the online QR code readers. Hoping that this would be the final flag, we attempted to read the code on my screen with a cell phone app, but none that we tried could read it in. Instead, I took a screenshot of my screen, cropped it in gimp and then read this into the online QR code reader. However, this time it could not decode the image, even after trying several different ones. After further inspection, it turns out that the QR code was actually a micro QR code, which is much less supported, but even those utilities that said it could read one, still could not. We tried another cell phone app that said it could read micro QR and we also tried making the background of the image white rather than grey, but nothing produced any results.

After hitting a dead end trying to scan in the image, we started looking for software that could decode it. I found one in particular that looked promising, called libQRCode. I downloaded their demo version for Linux (license costs $200) and ran their demo decoding program with our file. It successfully read it and decoded it, but...it * out part of the results since this was only the free version. We were able to see half the flag but we were still missing 7 characters. After some unsuccessful attempts to see the full un-obfuscated string in memory with gdb, another team member downloaded the Windows demo version of their software. This version had a demo program that did not ask for a file to decode, it simply decoded sample images that were included. All we had to do was nuke one of the sample images and rename our image to that sample’s name, rerun the program and voila, we had the flag.
