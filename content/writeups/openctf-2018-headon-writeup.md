---
title: "OpenCTF 2018 - HeadOn Writeup"
slug: "openctf-2018-headon-writeup"
date: "2018-08-14 23:25:28.730574"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

by Javantea

Aug 12, 2018

HeadOn is an easy forensics challenge.

[HeadOn-ac8890852965d787f7591bc10add61bb01efb5eb](https://www.altsci.com/concepts/images/HeadOn-ac8890852965d787f7591bc10add61bb01efb5eb) contained [blob](https://www.altsci.com/concepts/images/blob) which is a zip file.

{language=python}
~~~~~~~~
file blob
blob: Zip archive data, made by v?[0x31e], extract using at least v2.0, last modified Sun Dec 12 05:18:44 2010, uncompressed size 10299, method=deflate
~~~~~~~~

{language=python}
~~~~~~~~
unzip -l blob
Archive:  blob
  Length      Date    Time    Name
---------  ---------- -----   ----
    10299  08-04-2018 11:25   flag.pdf
---------                     -------
    10299                     1 file
~~~~~~~~

{language=python}
~~~~~~~~
unzip -v blob
Archive:  blob
 Length   Method    Size  Cmpr    Date    Time   CRC-32   Name
--------  ------  ------- ---- ---------- ----- --------  ----
   10299  Defl:N     9575   7% 08-04-2018 11:25 bfeb2149  flag.pdf
--------          -------  ---                            -------
   10299             9575   7%                            1 file

unzip blob
Archive:  blob
file #1:  bad zipfile offset (local header sig):  0
~~~~~~~~

I tried pulling the deflated data out by hand using Unproprietary, but no such luck. Then I looked at the file in a hex editor. It looks kinda like this:

{language=python}
~~~~~~~~
hexdump -C blob |head
00000000  00 00 00 00 14 00 00 00  08 00 34 5b 04 4d 49 21  |..........4[.MI!|
00000010  eb bf 67 25 00 00 3b 28  00 00 08 00 1c 00 66 6c  |..g%..;(......fl|
00000020  61 67 2e 70 64 66 55 54  09 00 03 a3 ef 65 5b b0  |ag.pdfUT.....e[.|
00000030  ef 65 5b 75 78 0b 00 01  04 00 00 00 00 04 00 00  |.e[ux...........|
00000040  00 00 85 5a 75 58 54 5b  d7 bf 0a 06 83 34 32 34  |...ZuXT[.....424|
00000050  43 37 33 4c 30 8c 20 20  29 9d 82 94 e4 10 02 43  |C73L0.  )......C|
00000060  23 8d 84 80 80 a4 8a 74  4b 48 23 dd dd 21 2d 9d  |#......tKH#..!-.|
00000070  c2 48 49 89 f4 07 de fb  c6 f7 de f7 7b be f3 3c  |.HI.........{..<|
00000080  fb 9c bd 62 af b5 f6 5a  bf bd cf 1f 7b b3 aa 48  |...b...Z....{..H|
00000090  4a f3 f2 f3 21 00 ac ad  99 ad 75 ad 15 ad 29 00  |J...!.....u...).|
~~~~~~~~

I noticed that the normal PK\x03\x04 header was missing, so I looked at infozip's documents and found that the first thing would be to try adding the first 4 bytes. That turned out to be the solution.

[Documentation](https://users.cs.jmu.edu/buchhofp/forensics/formats/pkzip.html)

{language=python}
~~~~~~~~
unzip ../bloba.
Archive:  ../bloba.
  inflating: flag.pdf
okular flag.pdf
pdftotext flag.pdf
cat flag.txt
Flag{SDG7qJ734rIw6f3f90832r}
~~~~~~~~

The flag is visible in the [pdf](https://www.altsci.com/concepts/images/flag.pdf).
