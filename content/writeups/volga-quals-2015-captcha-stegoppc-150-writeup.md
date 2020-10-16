---
title: "Volga Quals 2015 - captcha (stego,ppc 150) Writeup"
slug: "volga-quals-2015-captcha-stegoppc-150-writeup"
date: "2015-05-03 11:22:52.137913"
author_name: "tecknicaltom"
author_email: "tecknicaltom@neg9.org"
draft: false
toc: false
images:
---

captcha

Hint:

> We've got a rather strange png file. Very strange png. Something isn't right about it...
> 
> [png](https://neg9.org/resources/media/volga-quals-2015-captcha-stegoppc-150-writeup/capthca.png)

The provided PNG, when viewed, just looks like a single letter "i":

Running strings on the file though, it's easy to see a lot of IHDR and IEND strings, of which a PNG should have only one. Opening the file in Vim confirms that the x89PNG magic that starts the PNG is repeated over and over again. So at this point, it appears we have a bunch of PNG files concatenated together.

Running [hachoir-subfile](https://bitbucket.org/haypo/hachoir/wiki/Home) on the PNG allows us to easily split it into its individual files:

{language=python}
~~~~~~~~
$ hachoir-subfile capthca.png out/
[+] Start search on 1622884 bytes (1.5 MB)

[+] File at 0 size=792 (792 bytes): PNG picture: 256x256x24 => out/file-0001.png
[+] File at 792 size=867 (867 bytes): PNG picture: 256x256x24 => out/file-0002.png
[+] File at 1659 size=859 (859 bytes): PNG picture: 256x256x24 => out/file-0003.png
   -snip -
[+] File at 1620530 size=781 (781 bytes): PNG picture: 256x256x24 => out/file-1889.png
[+] File at 1621311 size=781 (781 bytes): PNG picture: 256x256x24 => out/file-1890.png
[+] File at 1622092 size=792 (792 bytes): PNG picture: 256x256x24 => out/file-1891.png

[+] End of search -- offset=1622884 (1.5 MB)
Total time: 1 sec 170 ms -- global rate: 1.3 MB/sec
~~~~~~~~

The first image is the "i" from above. The second is a "V":

"iV" doesn't seem like the beginning of a flag, but looking at the last image:

the "=" indicates it's probably base64.

At this point, I tried using both [GOCR](http://jocr.sourceforge.net/) and [Tesseract OCR](https://code.google.com/p/tesseract-ocr/) but due to the lack of context in the images and my bad track record with OCR software, the results were inaccurate enough to be useless to me.

I then thought to test out a hunch I had.

{language=python}
~~~~~~~~
$ md5sum out/* | awk '{print $1}' | sort | uniq -c | sort -n
      1 bdbd14948b0daa296ee4cf65e31ade15
      6 66e198735f47802c681c2151ee1090a4
     10 364c1666e2a89a5a64abe2d8e4b69580
     11 e2a7e99259104faabdfa2d26fef87586
     12 633200dd07b7e6197fc256a5fe571fc3
     12 69dc95333d32482337e7ef3c5097f4a3
     12 740d93829104827612a6a65eadb51f8e
     13 c107ea1ce43ef945145fb3f3790efeba
     14 a30d78a371704616a79c59ce20254cb6
     14 ec9e74a5f18847ddf04afe3d4064d22f
     15 20a02330b64a6ac42d84409e7867ff7c
     15 a8d6a8ac9e446affeb4261989a0baf50
     16 75471ca31da82c6f29fc5ec52dcf933f
     17 59a2345aae9ca2dd964ed1b61b0d460f
     18 609f644c2a044c364d5b9d743771de5e
     18 788e99506baca3d86eaad8c70cb84b36
     18 83cd79335ab3a759c99c84951b438838
     18 8d04f1fab0dddf0078a6d484d07f02a0
     18 d2013375b9f3ce21eb7a976e318167fb
     19 471e55d2cc16b3fe059e5ac9db0b31a1
     19 4d54b9e8c5618e1a0a027a35e7e0da3a
     20 18dc10da8f69c7f820964ca96c0831e1
     20 3faed2c01fad265ff59ab4791191b71a
     20 e9036e10c51334f8029929346dad5247
     21 49628fa64b0def5091e08cdd3bf92466
     21 c3214d0a5ab8ebc9f8f5f0416aea3b0d
     21 eb895ce9ad684a436665455f2db6e75d
     21 fde17b9751ca0fd02d103665ee806f24
     22 0757d43ab407cf09795ec741c4baca9c
     22 462b1bf11e61e1b1332e2f8e1c865489
     22 4a0648e7fc1c581fe19b90be6980110e
     22 4c518e3aa75477e98233e6b53e22a6ea
     22 54476bec9c688b7ddd1adc4dd177a30b
     23 0a8d1c7c7a1a3c2fa6570466305f0fd5
     23 2e691ba6943d759011a26c1de27fe70a
     23 644abdaaf808e4a2c75e2f2bcc347d3b
     24 28116c5219fdfe8233c26b5d9a6ba365
     24 7f7dd718aeceb6611edd34be8ee7a764
     24 b9d3f5b5889d10dae01c6f0ba8b5bb81
     25 1d06709d39b274054c8d7c778efaf9d7
     25 553cda794ad7557f8df76e1267c2bf4e
     25 75c2eb65189cbb54e437e139babf9788
     25 861dafbdd50792dc0648224639ce3963
     27 0872c31957efbd5587cdd8f0f3604b06
     28 ab8eec8c6d11e0ae0ae4848977d33c00
     28 df752942d48f765581979acae753ecf7
     29 90852d860e5058ed680ecf4fe3169c30
     29 a9684435587bc1ed84f8de134699b839
     30 0a6a19465a6a764cbea3b3276479db63
     30 8b7ce96904721935d677bc49a698d02e
     30 b33a799e16ead448c784b3fd4b6714b1
     31 52e387013dc79eb6a4f2557a099cd8a0
     32 34446e948f180db5bf8986c5f29a8599
     33 fb98f9311224624c8c36fc7ccfd558c7
     34 83175ff226605efce6dbc3878cee6418
     34 886a6a17d8c57bbc1ed119ea142600f8
     35 4ab6fe2ac42ca0f36a6bf00b291c331c
     36 b8c6e1ca952a11cd10096eff68dfdd22
     37 86817e1c3e6b700f10495c58fcc90f0f
     57 de3b40428ae92e6e76f0e4eb55bb90c4
     59 77c1999c75eacb9fee0b1e9b38a6d2b2
     89 fcb15452ffc5301b316b2ca78b9c3e72
     92 191a934656f747af98996dc86ff7c18f
    116 ca9f9515e27151a05ff3c0ca5c20a1a1
    184 5270cbd314b24e11cec0f03a86d1bf41
$ md5sum out/* | awk '{print $1}' | sort | uniq -c | wc -l
65
~~~~~~~~

So it looks like of the 1891 image files, there are only 65 unique files, telling us that the assumption about base64 is likely right, and the challenge creators used the exact same image for each occurrence of a given character. That makes our jobs easier because there's no variation between images per character.

I wrote the following Perl script to solve the challenge (detailed after the source):

"System Message: ERROR/3 (<string>:, line 122)"
Error in "code-block" directive:
unknown option: "filename".

{language=python}
~~~~~~~~
.. code-block:: perl
   :filename: solve.pl
   :number-lines:

   #!/usr/bin/perl

   use strict;
   use warnings;
   use diagnostics;
   use feature 'say';
   use Data::Dumper;
   use Digest::MD5::File qw(dir_md5_hex);
   use JSON;
   use File::Slurp;

   my $file_to_md5;
   if( -e "file_digests.txt")
   {
        $file_to_md5 = from_json(read_file('file_digests.txt'));
   }
   else
   {
        $file_to_md5 = dir_md5_hex('./out/');
        write_file('file_digests.txt', to_json($file_to_md5));
   }

   my $md5_to_letter;
   if( -e "md5_letters.txt")
   {
        $md5_to_letter = from_json(read_file('md5_letters.txt'));
   }
   else
   {
        my $count = 0;
        foreach my $file (keys %$file_to_md5)
        {
                if(!defined($md5_to_letter->{$file_to_md5->{$file}}))
                {
                        system("display out/$file");
                        $count++;
                        print "[$count]  Letter: ";
                        my $letter = <>;
                        chomp $letter;
                        $md5_to_letter->{$file_to_md5->{$file}} = $letter;
                }
        }
        write_file('md5_letters.txt', to_json($md5_to_letter));
   }

   my %temp;
   for my $md5 (keys %$md5_to_letter)
   {
        my $letter = $md5_to_letter->{$md5};
        die "$letter for $md5 and $temp{$letter}" if($temp{$letter});
        $temp{$letter} = $md5;
   }

   my $message = '';
   foreach my $file (sort keys %$file_to_md5)
   {
        $message .= $md5_to_letter->{ $file_to_md5->{$file} };
   }
   say $message;

~~~~~~~~

The code first takes the MD5 of each of the files keeping a hash of filename -> MD5, saving the intermediate results and using previous results if possible (lines 12-21). It then makes a hash mapping file MD5 values to the character in the image by displaying the image file to the user and prompting for the character, again keeping/using the intermediate results (lines 23-44). It then sanity checks the data to make sure that the same letter hasn't been given as an answer to multiple MD5s (lines 46-52). Finally, it uses all this data to spit out the concatenated contents of the images:

{language=python}
~~~~~~~~
$ ./solve.pl | base64 -d > decoded
base64: invalid input
$ file decoded
decoded: PNG image data, 272 x 131, 8-bit/color RGB, non-interlaced
~~~~~~~~

It appears as though the base64 data is missing a second final "=" resulting in the "invalid input" error, and the PNG file it decodes to is truncated, but it's enough to display in some viewers. A screenshot of the image that should display in all browsers:

Giving us the flag: {That_is_incredible_you_have_past!}
