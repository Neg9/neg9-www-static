---
title: "TAMUctf 2021 Recovery Solution"
slug: "recovery-writeup"
date: "2021-04-26 09:46:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[Recovery](https://ctftime.org/task/15816)

First I tried to get it with fls and icat, no luck.

Get [Unproprietary Git](https://www.altsci.com/repo/).


{language=bash}
~~~~~~~~
python3 ~/altsci/unproprietary/unproprietary/find_compress1.py --magic -l 2069000 floppy.img
found 1116160 gif

python3 ~/altsci/unproprietary/unproprietary/skipto.py floppy.img 1116160 >flag.gif

gwenview flag.gif 

gimp flag.gif 
~~~~~~~~


    12:58 < Javantea> okay I got the gif and it's corrupt
    12:58 < Javantea> but I bet I can get the data out of it

{language=bash}
~~~~~~~~
okteta flag.gif 
~~~~~~~~

Using the file format I learned from [Wikipedia GIF](https://en.wikipedia.org/wiki/GIF#Example_GIF_file)

I swapped ba00 with 0e00

That shows us the image. I think I can use that..

![Flag vertical](https://neg9.org/resources/media/tamuctf-2021-recovery-forensics-100-writeup/tamuctf-2021-recovery-flag1.gif)

But I really want the image data to be changed..

    31A 	03 00 05 00 	(3, 5) 	Image width and height in pixels

That did it.

![Hexedit of flag.gif](https://neg9.org/resources/media/tamuctf-2021-recovery-forensics-100-writeup/tamuctf-2021-recovery-okteta1.png)

found the 0e00 ba00 and swapped them.

The image shows the solution:

    gigem{0u7_0f_516h7_0u7_0f_m1nd}

![Flag](https://neg9.org/resources/media/tamuctf-2021-recovery-forensics-100-writeup/tamuctf-2021-recovery-flag2.gif)


