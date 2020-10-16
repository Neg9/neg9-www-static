---
title: "DEF CON CTF Quals 2015 - Access Control (Reverse Engineering 1) Writeup"
slug: "def-con-ctf-quals-2015-access-control-reverse-engineering-1-writeup"
date: "2015-10-13 16:04:16.627822"
author_name: "coldwaterq"
author_email: "coldwaterq@gmail.com"
draft: false
toc: false
images:
---

Hint:

> It's all about who you know and what you want.
> 
> access_control_server_f380fcad6e9b2cdb3c73c651824222dc.quals.shallweplayaga.me:17069

This challenge was a rather simple reversing problem. Me and Javantea worked on this.

When we connect to that site a connection id is returned to us, and we are expected to return a version. Since we didn't know the version to send we tried running the client. The client requests the user to enter something before it will connect, so we opened the client in hexrays. This showed that the client was expecting "hack the world". We ran the client again, entering the phrase and watched the communication with the server in wireshark.

With this we found that the communication worked as follows.

{language=python}
~~~~~~~~
server: connection ID: {connectionid}

***Welcome to the ACME data retrieval service***
what version is your client?

client: version 3.11.54

server: hello...who is this?

client: grumpy

server: enter user password

client: {password which changes}

server: hello grumpy, what would you like to do?

client: list users

server: grumpy
mrvito
gynophage
selir
jymbolia
sirgoon
duchess
deadwood
hello grumpy, what would you like to do?

client: print key

server: {access denied message}
~~~~~~~~

Running this a couple of times we found that the password changed every time. As such we tried to impersonate the server. Looking at this in hexrays it was clear that the password was an xor of the user-name and connection id. But with the connection id offset by 0 to 3. This offset was probably based on something; however, since the password could be tried multiple times, it was easier to try the password at each offset.

Once we had a client that could log in as grumpy, which is the same user as the client provided, we took the list of users and tried logging in with each of them. With each user we logged in and we tried the command "print key". The full list of listed users was:

{language=python}
~~~~~~~~
grumpy
mrvito
gynophage
selir
jymbolia
sirgoon
duchess
deadwood
~~~~~~~~

The user duchess was the only user that seemed to be allowed to view the key, which we took as a reference to Archer. With this user we were presented with a challenge that looked similar to the generated passwords. That part of the connection is shown bellow:

{language=python}
~~~~~~~~
server: connection ID: {connectionid}

***Welcome to the ACME data retrieval service***
what version is your client?

client: version 3.11.54

server: hello...who is this?

client: duchess

...

client: print key

server: challenge: {challenge value}
answer?
~~~~~~~~

So we looked at the client again, and found the part of the client that handled the challenge and response. This was also an xor however it was of the challenge, and an offset of the connection id. Although this time the connection id is offset by six plus the offset used before.

Then we sent the answer to the server and the server sent back the flag of:

"System Message: ERROR/3 (<string>:, line 89)"
Error in "code" directive:
maximum 1 argument(s) allowed, 7 supplied.

{language=python}
~~~~~~~~
.. code:: The only easy day was yesterday. 44564

~~~~~~~~

Bellow is the client I wrote in python.

{language=python}
~~~~~~~~
 1 import socket
 2 import os
 3 import time
 4 import binascii
 5 
 6 def send(text, find=None):
 7     print text
 8     s.send(text+'\n')
 9     time.sleep(.5)
10     resp = s.recv(9999)
11     print resp
12     if find is not None and find in resp:
13         return True
14     return False
15 
16 def get_password(username, connectionid):
17     return ''.join([chr(ord(x) > 0x1f and ord(x) or ord(x)+0x20) for x in xorbin(connectionid, username)])
18 
19 def xorbin(a, b):
20     q = 5
21     output = ''
22     for i in range(q):
23             output += chr(ord(a[i % len(a)]) ^ ord(b[i % len(b)]))
24     return output
25 
26 name = 'duchess'
27 s = socket.create_connection(('54.84.39.118',17069))
28 connectionid = s.recv(9999).partition(': ')[2].partition('\n')[0]
29 print connectionid
30 s.recv(9999)
31 send('version 3.11.54')
32 attempt = True
33 i = -1
34 while attempt:
35     i+= 1
36     send(name)
37     password = get_password(name, connectionid[i:])
38     attempt = not send(password, name)
39 
40 s.send('print key\n')
41 time.sleep(.5)
42 resp = s.recv(9999)
43 print resp
44 challenge = resp.partition(': ')[2].partition('\n')[0]
45 print challenge
46 send(get_password(challenge, connectionid[8+i-2:]))
~~~~~~~~

Cross-posted from: [http://coldwaterq.com/2015/06/02/Access_Control.html](http://coldwaterq.com/2015/06/02/Access_Control.html)
