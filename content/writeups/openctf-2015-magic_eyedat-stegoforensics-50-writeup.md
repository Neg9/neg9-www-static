---
title: "OpenCTF 2015 - magic_eye.dat (stego,forensics 50) Writeup"
slug: "openctf-2015-magic_eyedat-stegoforensics-50-writeup"
date: "2015-08-12 09:06:57.492831"
author_name: "hackworth"
author_email: "hackworth@bespokebytes.com"
draft: false
toc: false
images:
---

Hint:

> magic_eye.dat 50 --- I'm pretty sure it's a schooner - magic_eye-e5a34090f3a6fe71248e30720132aaad

The file is a corrupted PNG file.

The solution is the following:

- Open the file using pypng.

- Extract the data from the IDAT chuck.

- Decompress the extracted data with gzip.zlib.

- Flag is at the end of the text.

The solution code is:

{language=python}
~~~~~~~~
 1 #!/usr/bin/env python
 2 
 3 import png
 4 import gzip
 5 
 6 image = png.Reader("./magic_eye-e5a34090f3a6fe71248e30720132aaad")
 7 
 8 preamble = image.preamble()
 9 (t, d) = image.chunk()
10 
11 print(gzip.zlib.decompress(d))
~~~~~~~~

Flag: some_flag_goes_here
