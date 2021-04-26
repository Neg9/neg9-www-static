---
title: "TAMUctf 2021 Ring Of Fire Solution"
slug: "ring-of-fire-writeup"
date: "2021-04-26 12:54:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[Ring Of Fire](https://ctftime.org/task/15788) - 100 points

>    For none and none, there is always none  
>    For none and one, there can be only one  
>    For one and one, there is nothing but none  
>    [codeFile.txt](https://shell.tamuctf.com/static/90911652e12ad6b09c5ff548b00b906d/codeFile.txt)  
>    
>    Sometimes, I sing to myself  
>    Love is a burning thing  
>    And it makes a firery ring  
>    Bound by wild desire  
>    I fell in to a ring of fire  

Ring Of Fire is a fairly straightforward crypto problem. It depends on your willingness to go on an instinct and understand their first poem. The first poem describes XOR (aka Exclusive OR). This is the `^` operator in python, C++, and C among other languages. It's so important in computing that... [we spend a lot of time on it](https://www.cell-game.com/sixteen-good-reasons-to-love-xor-%E2%8A%95).

Assuming that they are using a one-time pad, what one time pad could they be using? Could it be the song itself?

Step 1 is to convert the codeFile.txt into codeFile1.bin. To make codeFile1.bin, just split up the binary into groups of 8 and convert to bytes.


```python
# like...
x = open('codeFile.txt','r').read()
y = []
for i in range(len(x)//8):
    y.append(int(x[i*8:i*8+8], 2))

open('codeFile1.bin', 'wb').write(bytes(y))
```

Now we can start decrypting.

```python
a = open('codeFile1.bin','rb').read()
a
b'\x06\x00\x1e\x0b\x00;]\x00"A\x11\x1dRF\x0b\x01\x15NT"GN5$a-\x05S\x01O\x00+\x04\t\x17\x06A\x13YF[DIRH\x19A[N\x85\x8a\xd1O&\x0b\x14T\x07\x14B\x12\x1bLU\x12HEAYBV#iW\x07\x16L\rNI/M\x11\x1dI\x02A\x1cI\x1d\x0eN\x08\x03RJI\x01\nd.W\x14\x0c\x18\tREN\x19\x1aS\x08C\x0b\x14\x1cBI\x0f\tDR\x08\r\x13O\x1dH\x00+\x1c\x11\r*&FW&\x0f\x07HC\x1cW\x03YS\r\x0cW\rCN\x10\x0e\x1e\x00o%N\x10H\x11\x05\x00SF\x03\x07M\x16\x1cR\x05\n\x19X\x00\x05\x06\x15\t\tR~3\x07\x06U\x05\x15T\x0b\x1a\x1cBSMN\x06U\x00\x0b\x17IM\x12\x01\x1b\x01\x1d&t\r\x16P\x17\n\x07\x06L\x03\x1f\x00\x0f\x07R\x11b1H\tA\x06\x0c\x1cGS\x1b\x07G\x03\x1aR\nliH\x0f\x16L\x0fA\x1b\x0b\x11\x1d\x0eAG\x0b\x12\x17\x03\x12\x16WR-\x00\x1d8C_\x0bM\x13=F1;?E\nKN<ED\x18\x16\x1d\x0cK\n\x00\x00\x00\x0cF\x0b\x1dW\x06c2N\x00E\x11\x18I\x00\x05\r\r\x00E\x11A\x04\x16C\x16A\x1a\x00\x13\x07\x0b\x17*7\x01\rC\x0cX\x00\x16\x1d\x17N\x17ES\x16\x1c\x1c\r\x07EV\x07U\x01\x01\x06d0H\nFR\x01\x07\x14\x00;\x03N\x08\x0c\x01\x16o1H1H\x00\x0c\x0bGB\x0e\x05K\x0f\x07\x15Eh5\x06\x01\x00\x17\t\x12\x06\x04C\x1b\x03R\x05\x15\x13\x01\x00\x0b\n\x00\x07\x05\x04\x0c\x1a\';\x01\x0e\x0b\x00\x0b\r\x14\x15\x13\x1aN\x0bI\x0c\x10I\x1b\x14\x00SR\x05\x1c\x11\x1cg:\x0cF\x04L\x1eE\x04\n\x1eL\x10\x00\x00S\x02\x0c\x18\x16\x00\x02O\x16\x18\x05\t\x00*8\x01XHB\x14\x1a\x00\x1d\x06\x06R\x03\x08\x01\x0cN\x10\t\x17TS\x18\x04\x0e\x01xiA\x08\x01L\x04U\x04\x0cL\x11OD\x04M\x07\x14\x1c\x01\x1bBGF\x00\x0c\x0bGP\x1d\x0fS\t\x07R\x06e\'C\x12\x17\x1a\x07\x0cD\x0e\x19\n\x0cAD\x1b\x05\x0fHE\t\x0e\x05\x05* \x02\x08\r\x16\x04\x04C\rL\x12\x19\x04\x14EW\x12\x0f\x06D\x1a\x06\x05\rE\x05b(\r\x0c\x00\x0c\x15R\x0c\x10\x16N\x1bEMB\x01\x1a\x0bSBI\x01\x1e\x1c\x0f\x1eotJ1H\x17I#\x06NO\x0fNF+\x1e\x04i?JK'
b = list(a)
c = 'Love is a burning thing'
c = b'Love is a burning thing'
d = list(c) 
len(d)
23
bytes([x ^ y for x, y in zip(d, a[:23])])
b'John R. Cash (born J. R'
c = b'Love is a burning thing\nAnd it makes a firery ring\nBound by wild desire\nI fell in to a ring of fire'
d = list(c)                     
bytes([x ^ y for x, y in zip(d, a[:len(d)])])                                                                                      
b'John R. Cash (born J. R. Cash; February 26, 1932 \xe2\x80\x93 September 12, 2003) was an American singer, so'
bytes([x ^ y for x, y in zip(d, a[len(d):2*len(d)])])
b"(A!q,qzr$n{o!f*es<6!fg#XIcwot< F}zhY\x06'w@fu-1ewq0=j\x06\x15b6 t.|yOR'|,1ae /qbG_<4`fuxik&af)3^Anh2%z2+|u06"
bytes([x ^ y for x, y in zip(d, a[len(d)+1:2*len(d)+1])])
b'b8bi8`!e/9x&z-bz{b=g`*5\x02L}+=!t\x0bqpfOUf6\x06in:&n.#+:ck]O,;~j>b\x16\x059y$u%d65ju(\x1cUrcou4 lh5}fr\x1f\x13uo;b5;m:z+!('
c = b'Love is a burning thing\nAnd it makes a firery ring\nBound by wild desire\nI fell in to a ring of fire\n'
d = list(c)
```

Search for lyrics on google.  
Watch this https://www.youtube.com/watch?v=5WyLhwYFgmk

```python
bytes([x ^ y for x, y in zip(d, a[len(d)+1:2*len(d)+1])])                                                      
b"\x1b{z});6nx:1}1e}r%i{a-<o\x07R!yhi_<|lAC5wG/a!1y%zy!db0\x07\x01!e`z \r\\ng!}a 7#pn?s\x16\x1b%l|4l%o{)}=^R't<krtd|<$:?+\x0c"
c = b'Love is a burning thing\nAnd it makes a firery ring\nBound by wild desire\nI fell in to a ring of fire\nI fell into a burning ring of fire\nIt went down down down and the flames went higher'
d = list(c)                      
bytes([x ^ y for x, y in zip(d, a[:len(d)])])                                                                                        b'John R. Cash (born J. R. Cash; February 26, 1932 \xe2\x80\x93 September 12, 2003) was an American singer, songwriter, musician, and actor. Much o2wQji<cx8t7sic cc*\x7fyp \x0eK*0<y` 5*bj(e<%`dmxhla}l{'
len(b'John R. Cash (born J. R. Cash; February 26, 1932 \xe2\x80\x93 September 12, 2003) was an American singer, songwriter, musician, and actor. Much o')
136
c[:136]
b'Love is a burning thing\nAnd it makes a firery ring\nBound by wild desire\nI fell in to a ring of fire\nI fell into a burning ring of fire\nI'
c = b'Love is a burning thing\nAnd it makes a firery ring\nBound by wild desire\nI fell in to a ring of fire\nI fell into a burning ring of fire\nI went down down down and the flames went higher' 
bytes([x ^ y for x, y in zip(d, a[:len(d)])])
b'John R. Cash (born J. R. Cash; February 26, 1932 \xe2\x80\x93 September 12, 2003) was an American singer, songwriter, musician, and actor. Much o2wQji<cx8t7sic cc*\x7fyp \x0eK*0<y` 5*bj(e<%`dmxhla}l{'
d = list(c)                     
bytes([x ^ y for x, y in zip(d, a[:len(d)])])
b"John R. Cash (born J. R. Cash; February 26, 1932 \xe2\x80\x93 September 12, 2003) was an American singer, songwriter, musician, and actor. Much of Cash's my7b{9-'!g`>a\x01And t%f?'nb>6k7k~90ibnp{"
c = b'Love is a burning thing\nAnd it makes a firery ring\nBound by wild desire\nI fell in to a ring of fire\nI fell into a burning ring of fire\nI went down, down, down and the flames went higher'
d = list(c)
bytes([x ^ y for x, y in zip(d, a[:len(d)])])                                                                                        b"John R. Cash (born J. R. Cash; February 26, 1932 \xe2\x80\x93 September 12, 2003) was an American singer, songwriter, musician, and actor. Much of Cash's music containOD themes of sorrow, moral "
len(b"John R. Cash (born J. R. Cash; February 26, 1932 \xe2\x80\x93 September 12, 2003) was an American singer, songwriter, musician, and actor. Much of Cash's music contain")
158
c[:158]
b'Love is a burning thing\nAnd it makes a firery ring\nBound by wild desire\nI fell in to a ring of fire\nI fell into a burning ring of fire\nI went down, down, down'
c = b'Love is a burning thing\nAnd it makes a firery ring\nBound by wild desire\nI fell in to a ring of fire\nI fell into a burning ring of fire\nI went down, down, down And the flames went higher'
d = list(c)
bytes([x ^ y for x, y in zip(d, a[:len(d)])])b"John R. Cash (born J. R. Cash; February 26, 1932 \xe2\x80\x93 September 12, 2003) was an American singer, songwriter, musician, and actor. Much of Cash's music containOd themes of sorrow, moral "
c = b'Love is a burning thing\nAnd it makes a firery ring\nBound by wild desire\nI fell in to a ring of fire\nI fell into a burning ring of fire\nI went down, down, down And the flames went higher\nand it burns, burns, burns the ring of fire, the ring of fire'
d = list(c)
bytes([x ^ y for x, y in zip(d, a[:len(d)])])b"John R. Cash (born J. R. Cash; February 26, 1932 \xe2\x80\x93 September 12, 2003) was an American singer, songwriter, musician, and actor. Much of Cash's music containOd themes of sorrow, moral tRibulation, and redemption\x06\x00especially in tN\x11<a$&~u)4;h!#|;x\t"
len(b"John R. Cash (born J. R. Cash; February 26, 1932 \xe2\x80\x93 September 12, 2003) was an American singer, songwriter, musician, and actor. Much of Cash's music containOd themes of sorrow, moral tRibulation, and redemption\x06\x00especially in t")
229
c[:229]
b'Love is a burning thing\nAnd it makes a firery ring\nBound by wild desire\nI fell in to a ring of fire\nI fell into a burning ring of fire\nI went down, down, down And the flames went higher\nand it burns, burns, burns the ring of fire'
c = b'Love is a burning thing\nAnd it makes a firery ring\nBound by wild desire\nI fell in to a ring of fire\nI fell into a burning ring of fire\nI went down, down, down And the flames went higher\nand it burns, burns, burns the ring of fire the ring of fire'
d = list(c)
bytes([x ^ y for x, y in zip(d, a[:len(d)])])b"John R. Cash (born J. R. Cash; February 26, 1932 \xe2\x80\x93 September 12, 2003) was an American singer, songwriter, musician, and actor. Much of Cash's music containOd themes of sorrow, moral tRibulation, and redemption\x06\x00especially in tBE later stages o"
c = b'Love is a burning thing\nAnd it makes a firery ring\nBound by wild desire\nI fell in to a ring of fire\nI fell into a burning ring of fire\nI went down, down, down And the flames went higher\nand it burns, burns, burns the ring of fire The ring of fire\nI fell into a burning ring of fire\nI went down, down, down And the flames went higher\nand it burns, burns, burns the ring of fire The ring of fire\n'
d = list(c)
bytes([x ^ y for x, y in zip(d, a[:len(d)])])b"John R. Cash (born J. R. Cash; February 26, 1932 \xe2\x80\x93 September 12, 2003) was an American singer, songwriter, musician, and actor. Much of Cash's music containOd themes of sorrow, moral tRibulation, and redemption\x06\x00especially in tBe later stages of his career. gigem{x0r_is_c0mmuT4T1ve}. He was known for hCs deep, calm bass-baritone Voice, the distinctive souDD of his TennessOe Three backing b"
```

The flag is `gigem{x0r_is_c0mmuT4T1ve}`

