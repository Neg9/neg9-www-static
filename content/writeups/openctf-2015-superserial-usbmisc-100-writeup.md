---
title: "OpenCTF 2015 - superserial (usb,misc 100) Writeup"
slug: "openctf-2015-superserial-usbmisc-100-writeup"
date: "2015-08-12 09:16:04.759617"
author_name: "craSH"
author_email: "crash@neg9.org"
draft: false
toc: false
images:
---

Hint:

> superserial 100 --- there are flags hidden on and about the beaglebone. The flag referring to Goatse goes here

Watching dmesg on a Linux system when plugging in the beaglebone, we see
several devices that it exposes, including the USB network interface, among
others. The reported serial number for one of these devices is the flag.

Below is exactly what can be seen in dmesg:

"System Message: ERROR/3 (<string>:, line 11)"
Error in "code" directive:
maximum 1 argument(s) allowed, 5 supplied.

{language=python}
~~~~~~~~
.. code:: 2010-01-01T02:24:50.207Z cpu0:2511)<6>usb 1-3: SerialNumber: W3_TR1ED_T0_PU7_ASC1I_GO4TSE_H3RE_W0ULDNT_FI7-WQGPD7HT4RIK

~~~~~~~~

As the hint references Goatse, the l33t-speak string of "WE TRIED TO PUT ASCII
GOATSE HERE WOULDNT FIT" seems like it's the key, followed by what appears to
be a more traditional serial number (-WQGPD7HT4RIK).

The flag is: W3_TR1ED_T0_PU7_ASC1I_GO4TSE_H3RE_W0ULDNT_FI7
