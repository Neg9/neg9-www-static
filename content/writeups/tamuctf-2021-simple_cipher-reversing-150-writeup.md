---
title: "TAMUctf 2021 simple_cipher Solution"
slug: "simple_cipher-writeup"
date: "2021-04-26 12:33:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[simple_cipher](https://ctftime.org/task/15815) - 150 points

>    We have a flag encrypted using this program. Can you figure out what it is? [simple_cipher](https://shell.tamuctf.com/static/568f7285afca50e7518d99b13fcaeb6b/simple_cipher) [flag.enc](https://shell.tamuctf.com/static/568f7285afca50e7518d99b13fcaeb6b/flag.enc)

This is a very interesting cipher. By testing values you can understand how to attack it correctly. This is the tactic I used after Angr refused to give me a good answer.

```
./simple_cipher gigem{zbcdefghijklmnopqrst} |hexdump -C
00000000  61 9e df d4 f7 3d 62 31  f0 79                    |a....=b1.y|
0000000a
./simple_cipher gigem{zbcdefghijklmnopqrsz} |hexdump -C
00000000  61 9e df d4 f7 3d 62 31  f0 79 0e 38 70 81 11 2f  |a....=b1.y.8p../|
00000010  5b ff 55 3a 14 35 81 0a  85 ad d9                 |[.U:.5.....|
0000001b
```

Substantial bug in simple_cipher..

```
./simple_cipher zzzzzzzzzzzzzzzzzzzzzzzzzt |hexdump -C
00000000  71 8f c9 c3 e3 28 68 3a  f8 70                    |q....(h:.p|
0000000a
./simple_cipher zzzzzzzzzzzzzzzzzzzzzzzzzv |hexdump -C
00000000  71 8f c9 c3 e3 28 68 3a  f8 70 02 3f 6d 92 0c 30  |q....(h:.p.?m..0|
00000010  4c fe 55 22 0d 2b 9e 16  98 bf                    |L.U".+....|
0000001a

angr@suzy ~ $ ./simple_cipher at |hexdump -C
00000000  7f 94                                             |..|
00000002
angr@suzy ~ $ ./simple_cipher aat |hexdump -C
00000000  6a 94 c7                                          |j..|
00000003
angr@suzy ~ $ ./simple_cipher aaat |hexdump -C
00000000  7f 94 d2 d8                                       |....|
00000004
angr@suzy ~ $ ./simple_cipher aaaat |hexdump -C
00000000  6a 94 d2 d8 ed                                    |j....|
00000005
angr@suzy ~ $ ./simple_cipher aaaaat |hexdump -C
00000000  6a 94 c7 d8 f8 33                                 |j....3|
00000006
angr@suzy ~ $ ./simple_cipher aaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 26 73                              |j....&s|
00000007
angr@suzy ~ $ ./simple_cipher aaaaaaat |hexdump -C
00000000  7f 94 d2 d8 f8 33 73 21                           |.....3s!|
00000008
angr@suzy ~ $ ./simple_cipher aaaaaaaat |hexdump -C
00000000  6a 94 c7 d8 f8 33 73 21  e3                       |j....3s!.|
00000009
angr@suzy ~ $ ./simple_cipher aaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 ed 33 73 21  e3 6b                    |j....3s!.k|
0000000a
angr@suzy ~ $ ./simple_cipher aaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 66 21  e3 6b 15                 |j....3f!.k.|
0000000b
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 73 21  f6 6b 15 24              |j....3s!.k.$|
0000000c
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 73 21  e3 6b                    |j....3s!.k|
0000000a
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 73 21  e3 6b 15 24 63 89        |j....3s!.k.$c.|
0000000e
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 73 21  e3 6b 15 24 76 89 02     |j....3s!.k.$v..|
0000000f
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaat |hexdump -C
00000000  7f 94 d2 d8 f8 33 73 21  e3 6b 15 24 76 89 17 2b  |.....3s!.k.$v..+|
00000010
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 81 d2 d8 f8 33 73 21  e3 6b 15 24 76 89 17 2b  |j....3s!.k.$v..+|
00000010  57                                                |W|
00000011
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 c7 d8 f8 33 73 21  e3 6b 15 24 76 89 17 2b  |j....3s!.k.$v..+|
00000010  57 e5                                             |W.|
00000012
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 cd f8 33 73 21  e3 6b 15 24 76 89 17 2b  |j....3s!.k.$v..+|
00000010  57 e5 4e                                          |W.N|
00000013
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 ed 33 73 21  e3 6b 15 24 76 89 17 2b  |j....3s!.k.$v..+|
00000010  57 e5 4e 39                                       |W.N9|
00000014
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 26 73 21  e3 6b 15 24 76 89 17 2b  |j....&s!.k.$v..+|
00000010  57 e5 4e 39 16                                    |W.N9.|
00000015
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 66 21  e3 6b 15 24 76 89 17 2b  |j....3f!.k.$v..+|
00000010  57 e5 4e 39 16 30                                 |W.N9.0|
00000016
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 73 34  e3 6b 15 24 76 89 17 2b  |j....3s4.k.$v..+|
00000010  57 e5 4e 39 16 30 85                              |W.N9.0.|
00000017
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 73 21  f6 6b 15 24 76 89 17 2b  |j....3s!.k.$v..+|
00000010  57 e5 4e 39 16 30 85 0d  11 10 00                 |W.N9.0.....|
0000001b
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 73 21  e3 7e 15 24 76 89 17 2b  |j....3s!.~.$v..+|
00000010  57 e5 4e 39 16 30 85 0d  83                       |W.N9.0...|
00000019
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 73 21  e3 6b                    |j....3s!.k|
0000000a
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 73 21  e3 6b 15 31 76 89 17 2b  |j....3s!.k.1v..+|
00000010  57 e5 4e 39 16 30 85 0d  83 a4 d1                 |W.N9.0.....|
0000001b
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 73 21  e3 6b 15 24 63 89 17 2b  |j....3s!.k.$c..+|
00000010  57 e5 4e 39 16 30 85 0d  83 a4 d1 79              |W.N9.0.....y|
0000001c
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 73 21  e3 6b 15 24 76 9c 17 2b  |j....3s!.k.$v..+|
00000010  57 e5 4e 39 16 30 85 0d  83 a4 d1 79 31           |W.N9.0.....y1|
0000001d
angr@suzy ~ $ ./simple_cipher aaaaaaaaaaaaaaaaaaaaaaaaaaaaat |hexdump -C
00000000  6a 94 d2 d8 f8 33 73 21  e3 6b 15 24 76 89 02 2b  |j....3s!.k.$v..+|
00000010  57 e5 4e 39 16 30 85 0d  83 a4 d1 79 31 2e        |W.N9.0.....y1.|
0000001e

./simple_cipher aaaaaaaaaaaaaaaaaaaaaaaa$'\x0a' |hexdump -C
00000000  6a 94 d2 d8 f8 33 73 21  e3                       |j....3s!.|
00000009
```

So now we have the ability to break it in the same way.

From my IRC channel:

> 01:18 < Javantea> found an interesting bug in simple_cipher..
01:18 < Javantea> just by screwing around
01:19 < Javantea> I think it's null related
01:19 < Javantea> yeah.. cool
01:21 < Javantea> narrowed it down to a single character
01:26 < Javantea> reproed it on a different character
01:26 < Javantea> was pretty easy
01:27 < Javantea> each ...
01:27 < Javantea> will explain it in a blog post. doesn't make sense to explain it here

So what we have here is a null. It's pretty clear right? Yes. Okay, so let's just assume that won't happen in the one we want..

So now let's modify each value... until it fits.

first character modifies the second output.  
second character modifies the first output.  
third character modifies the second output. so we have a 2 to 1 problem  
fourth character modifies the third output.  
fifth character modifies the fifth output.  
sixth character modifies the fifth output. so we have a 2 to 1 problem  
seventh character modifies the sixth output.  
I have an idea now. Like right shift 4.  

Nothing influences the first so far. Wait, nevermind, it's the length.

38 aa are the first two bytes. let's try to get those first.

first is definitely not the length. doesn't change with length..

changed when I changed 
```
angr@suzy ~ $ ./simple_cipher zzzzzzzzzsccAAAAAAAAAAABazzzzzzzzz |hexdump -C
angr@suzy ~ $ ./simple_cipher zzzzzzzzzscczzzzAAAAAAABazzzzzzzzz |hexdump -C
angr@suzy ~ $ ./simple_cipher zzzzzzzzzsccAAAzAAAAAAABazzzzzzzzz |hexdump -C
```
So the first byte is controlled by the 16th value. let's set that..

```
angr@suzy ~ $ ./simple_cipher zzzzzzzzzsccAAA3AAAAAAABazzzzzzzzz |hexdump -C
00000000  38 b4 f2 f8 d8 13 53 01  c0 6b 0e 3f 6d 92 0c 30  |8.....S..k.?m..0|
00000010  4c fe 55 22 0d 2b 9e 16  98 bf ca 62 23 2c 01 e5  |L.U".+.....b#,..|
00000020  5c 7e                                             |\~|
00000022
```

Okaya so now we have our first value..

Let's get 38 aa

didn't change. cool

```
for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3"$x"AAAAAAAAAAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa '
```

Convert uppercase to lowercase with my text editor, [Kate](https://kate-editor.org/):

a b c d e f g h i j k l m n o p q r s t u v w x y z

So this for loop which I use to solve this cipher is pretty straightforward. It puts each plaintext character in the position and sees which one produces the same output as the ciphertext. We keep adding ciphertext bytes to the grep one after another and we keep moving the plaintext placeholder "$x" to the right until it stops working.

```
for x in _; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3"$x"AAAAAAAAAAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa'
_
00000000  38 aa f2 f8 d8 13 53 01  c3 4b 35 04 56 a9 37 0b  |8.....S..K5.V.7.|
angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3"$x"AAAAAAAAAAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca'
angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 0); do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3"$x"AAAAAAAAAAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca'
angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 0) \ \| '$' '!' ; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3"$x"AAAAAAAAAAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca'   
angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 0) \ \| '$' '!' '@'; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3"$x"AAAAAAAAAAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca'angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 0) \ \| '$' '!' '@'; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3_"$x"AAAAAAAAAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca'y
00000000  38 aa ca f8 d8 13 53 01  c3 4b 35 04 56 a9 37 0b  |8.....S..K5.V.7.|

angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 0) \ \| '$' '!' '@'; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3_y"$x"AAAAAAAAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca 89'
0
00000000  38 aa ca 89 d8 13 53 01  c3 4b 35 04 56 a9 37 0b  |8.....S..K5.V.7.|
angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 0) \ \| '$' '!' '@'; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3_y0"$x"AAAAAAAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca 89 ec'
u
00000000  38 aa ca 89 ec 13 53 01  c3 4b 35 04 56 a9 37 0b  |8.....S..K5.V.7.|
angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 0) \ \| '$' '!' '@'; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3_y0u_"$x"AAAAAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca 89 ec 20 4d'
angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 0) \ \| '$' '!' '@'; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3_y0u"$x"AAAAAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca 89 ec 20 '
r
00000000  38 aa ca 89 ec 20 53 01  c3 4b 35 04 56 a9 37 0b  |8.... S..K5.V.7.|
angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 0) \ \| '$' '!' '@'; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3_y0ur_"$x"AAAAAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca 89 ec 20 4d 70'
0
00000000  38 aa ca 89 ec 20 4d 70  c3 4b 35 04 56 a9 37 0b  |8.... Mp.K5.V.7.|
angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 0) \ \| '$' '!' '@'; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3_y0ur_0"$x"AAAAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca 89 ec 20 4d 70  f5'
w
00000000  38 aa ca 89 ec 20 4d 70  f5 4b 35 04 56 a9 37 0b  |8.... Mp.K5.V.7.|
angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 0) \ \| '$' '!' '@'; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3_y0ur_0wn"$x"AAAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca 89 ec 20 4d 70  f5 64 2b'
_
00000000  38 aa ca 89 ec 20 4d 70  f5 64 2b 04 56 a9 37 0b  |8.... Mp.d+.V.7.|
angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 0) \ \| '$' '!' '@'; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3_y0ur_0wn_"$x"AAAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca 89 ec 20 4d 70  f5 64 2b 26'
c
00000000  38 aa ca 89 ec 20 4d 70  f5 64 2b 26 56 a9 37 0b  |8.... Mp.d+&V.7.|
angr@suzy ~ $ for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 0) \ \| '$' '!' '@'; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3_y0ur_0wn_c"$x"AAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca 89 ec 20 4d 70  f5 64 2b 26 26'
```

minor bug =]

```
for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 9) \| '$' '!' '@'; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3_y0ur_0wn_c"$x"AAAAAAA |hexdump -C; done |grep -B1 '00000000  38 aa ca 89 ec 20 4d 70  f5 64 2b 26 26'
1
00000000  38 aa ca 89 ec 20 4d 70  f5 64 2b 26 26 a9 37 0b  |8.... Mp.d+&&.7.|

for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 9) \| '$' '!' '@'; do echo "$x"; ./simple_cipher AAAAAAAAAAAAAAA3_y0ur_0wn_c1ph3"$x" |hexdump -C; done |grep -B2 '00000010  44'
r
00000000  38 aa ca 89 ec 20 4d 70  f5 64 2b 26 26 98 1e 79  |8.... Mp.d+&&..y|
00000010  44 c5 6e 19 36 10 a5 2d  a3 84 f1 59 11 0e 23 e5  |D.n.6..-...Y..#.|
```

So now we need the rest..

We have:
3_y0ur_0wn_c1ph3r

```
for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 9) \| '$' '!' '@'; do echo "$x"; ./simple_cipher "$x"AAAAAAAAAAAAAA3_y0ur_0wn_c1ph3r |hexdump -C; done |grep -B2 '00000010  44 b1'
5
00000000  38 aa ca 89 ec 20 4d 70  f5 64 2b 26 26 98 1e 79  |8.... Mp.d+&&..y|
00000010  44 b1 6e 19 36 10 a5 2d  a3 84 f1 59 11 0e 23 e5  |D.n.6..-...Y..#.|

for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 9) \| '$' '!' '@' '}'; do echo "$x"; ./simple_cipher 5"$x"AAAAAAAAAAAAA3_y0ur_0wn_c1ph3r |hexdump -C; done |grep -B2 '00000010  44 b1 52'}
00000000  38 aa ca 89 ec 20 4d 70  f5 64 2b 26 26 98 1e 79  |8.... Mp.d+&&..y|
00000010  44 b1 52 19 36 10 a5 2d  a3 84 f1 59 11 0e 23 e5  |D.R.6..-...Y..#.|
```

...

```
for x in A B C D E F G H I J K L M N O P Q R S T U V W X Y Z a b c d e f g h i j k l m n o p q r s t u v w x y z _ $(seq 0 9) \| '$' '!' '@' '}' '{' '[' ']' '#' '$' '%' '^' '&' '*'; do echo "$x"; ./simple_cipher 5}gigem{d0n"$x"AAA3_y0ur_0wn_c1ph3r |hexdump -C; done |grep -B2 '00000010  44 b1 52 3f 1e 36 81 01  99 a1 80 76 67' 
7
00000000  38 aa ca 89 ec 20 4d 70  f5 64 2b 26 26 98 1e 79  |8.... Mp.d+&&..y|
00000010  44 b1 52 3f 1e 36 81 01  99 a1 80 76 67 0e 23 e5  |D.R?.6.....vg.#.|
```

Now I see what is going on here... I was off by 2 in the size of the plaintext. Luckily enough the shift depends on the length.

```
angr@suzy ~ $ ./simple_cipher 5}gigem{d0n7_wr3_y0ur_0wn_c1ph3r|hexdump -C
00000000  38 aa ca 89 ec 20 4d 70  f5 64 2b 26 26 98 1e 79  |8.... Mp.d+&&..y|
00000010  44 b1 52 3f 1e 36 81 01  99 a1 80 76 67 10 15 d6  |D.R?.6.....vg...|
00000020
angr@suzy ~ $ ./simple_cipher 5}gigem{d0n7_writ3_y0ur_0wn_c1ph3r|hexdump -C
00000000  62 81 80 e6 e0 62 67 32  dd 3a 03 2b 48 8b 47 3a  |b....bg2.:.+H.G:|
00000010  5e b7 5d 6d 0a 36 8d 0b  87 a8 cb 7c 60 21 55 fb  |^.]m.6.....|`!U.|
00000020  6a 4d                                             |jM|
00000022
angr@suzy ~ $ ./simple_cipher gigem{d0n7_writ3_y0ur_0wn_c1ph3r5}|hexdump -C
00000000  38 aa ca 89 ec 20 4d 70  f5 64 2b 26 26 98 1e 79  |8.... Mp.d+&&..y|
00000010  44 b1 52 3f 1e 36 81 01  99 a1 80 76 67 10 15 d6  |D.R?.6.....vg...|
00000020  74 4b                                             |tK|
00000022
angr@suzy ~ $ ./simple_cipher gigem{d0n7_writ3_y0ur_0wn_c1ph3r5}>solution.enc
angr@suzy ~ $ sha512sum solution.enc 
c643088c315ffc214d987ac1870d491ab3f7fdedb43c8476f1773c44986ceb9020416cd0485c7f3892ad6656568b7e4a682049311e7e64c31a56e4381e351c1b  solution.enc
angr@suzy ~ $ ./simple_cipher gigem{d0n7_wr173_y0ur_0wn_c1ph3r5}>solution.enc
angr@suzy ~ $ sha512sum solution.enc 
6e589c59f281cf4d597698eb69a23bec4e4db8e30fd921e22083ea6fc734de0dc5ccb01404c64fdc9943f9184786c75f09d270f412351b27f652d99053700bb8  solution.enc

```

The flag is `gigem{d0n7_wr173_y0ur_0wn_c1ph3r5}`


