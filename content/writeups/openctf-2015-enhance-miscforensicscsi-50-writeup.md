---
title: "OpenCTF 2015 - Enhance (misc,forensics,CSI 50) Writeup"
slug: "openctf-2015-enhance-miscforensicscsi-50-writeup"
date: "2015-08-12 01:02:13.692269"
author_name: "craSH"
author_email: "crash@neg9.org"
draft: false
toc: false
images:
---

Hint:

> Enhance 50 --- We think this woman might be a license dodger - can you find out where she's hiding? 172.16.18.20/enhance-50e7818433c4e832e0e96e91dc4cb64c

The asset file for this challenge is a JPEG photo of a woman's face, as seen below (resized):

[(Original (12mb) file)](https://neg9.org/resources/media/openctf-2015-enhance-miscforensicscsi-50-writeup/open_ctf-2015-misc,forensics,csi-50-enhance_orig.jpg).

Open the graphic file in an editor such as Gimp, note that there is a "reflection" of a QR Code in the subject's left eye (on the right side of the graphic).

Zoom in on this, crop the QR code out.

There's some reflection present diagonally across the QR code, as well as some background color that makes it impossible to scan, this can be seen below:

With the cropped QR code, use Gimp's "Threshold" tool (Color -> Threshold), slide the threshold down until the QR code looks more reasonable (Values 112 / 255):

And the final "enhanced" QR code looks like so:

The "enhanced" version of the QR code can be scanned with any QR code reader, such as "Barcode Scanner" on Android. It then decodes as: Ju5tPr1ntTheDAmNTh1n6 (Just Print The Damn Thing)

The flag is:
Ju5tPr1ntTheDAmNTh1n6
