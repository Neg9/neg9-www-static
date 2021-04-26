---
title: "TAMUctf 2021 pybox Solution"
slug: "pybox-writeup"
date: "2021-04-26 10:56:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[pybox](https://ctftime.org/task/15805) - 150 points

>    We spun up a server for you to execute your python code! For security reasons, we've disabled a few syscalls, but you can do all the computation you'd like!
[restricted_python/src/main.rs](https://shell.tamuctf.com/static/4ff379df6496923e8675b07c6fd6a6ef/restricted_python/src/main.rs)
>    `openssl s_client -connect tamuctf.com:443 -servername pybox -quiet`

This challenge was remarkably easy. They use [seccomp](https://en.wikipedia.org/wiki/Seccomp) to disable reading from a file. That's not nearly enough to stop a hacker from accessing a flag. They did not disable mmap, which is a pretty standard way to access a file. So standard in fact that Python has a built-in module for it. It's that useful. Below is the exploit. Yes is was that easy. I had to determine the length of the flag by trial and error because it wouldn't give you the flag if you asked for more than the file size.

```python
a = open('flag.txt','rb')
x = mmap.mmap(a.fileno(), 26, flags=mmap.MAP_PRIVATE)
print(x[:26])
.

b'gigem{m3m0ry_m4pp3d_f1l35}'
```

The flag is `gigem{m3m0ry_m4pp3d_f1l35}`

