---
title: "TAMUctf 2021 Spectral Imaging Solution"
slug: "spectral-imaging-writeup"
date: "2021-04-26 11:54:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[Spectral Imaging](https://ctftime.org/task/15814) - 100 points

>    Some things are meant to be heard but not seen. This sounds like it's meant to be seen, not heard.
>    [audio.wav](https://shell.tamuctf.com/static/ad51ffa4ee71c1037dd2b6b5b3841a3c/audio.wav)

Spectral Imaging is just a simple spectrogram problem which I've seen many times before. Open the file in [Audacity](https://www.audacityteam.org/), switch to spectrogram. Set the settings to high top frequency and that's all. This can probably also be solved with `sox`.

![Audacity spectrogram of audio.wav](https://neg9.org/resources/media/tamuctf-2021-spectral-imaging-sigint-100-writeup/tamuctf-2021-spectral-imaging-audacity1.png)

