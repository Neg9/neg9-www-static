---
title: "SECCON CTF 2014 Crypto 400 Ms.Fortune? : 4096-bit RSA"
slug: "seccon-ctf-2014-crypto-400-msfortune-4096-bit-rsa"
date: "2015-03-09 14:03:56.849848"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

## Step 1: Find out what we're working with
{language=python}
~~~~~~~~
gpg encrypted.gpg
gpg: encrypted with RSA key, ID 7DAD8D6C
gpg: decryption failed: No secret key

gpg key.pub.gpg
pub  4096R/8C2E1A7D 1970-01-02
uid                            SECCON ____Key <____key@seccon>
sub  4096R/7DAD8D6C 1970-01-02

gpg --import key.pub.gpg
gpg: key 8C2E1A7D: public key "SECCON ____Key <____key@seccon>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
~~~~~~~~

recover.py has some gcd and so forth going on.

It requires a p and q and it outputs a gpg key. Easy, right? I've done this so many times before.

{language=python}
~~~~~~~~
p=int(sys.argv[1])
q=int(sys.argv[2])
if p>=q:
        sys.exit(1)
~~~~~~~~

## Step 2: Read the file that we want to use with a custom OpenPGP parser
If you don't have a custom OpenPGP parser, I've recently learned that you can use gpg --list-packets to gain some knowledge about a OpenPGP stream. See below for what said custom OpenPGP parser output for the given file.

{language=python}
~~~~~~~~
python3 parsegpg1.py key.gpg.masked
Total length: 4775
binary offset: 0
flags: 0x95
        always: 1
        new_type: 0
        length_type: old 1
        packet_length: 1816
        packet_type: 5 Secret-Key Packet
0400015180011000b18852a9120ee31e8a0e863562fc0f185335a0686dd4ef223acfa30bdb688bc2
7d70d13ca6c17a1a020fb5e87589fec5a1b5eb02ac50dd1518547cbc9ef5c5c374408333c24c1824
4de210280da73c8350a86da2b047c8474b10ed109a54895322e15bf8001dd9823dd16cd644f3f9a0
a20f45dfece7237ea013f4eda6f5ffdf32110987022e5b0af256e36bd56b80453f507aa6d2932cf6
566384a06213f011b4ef8156f931077d54b6feccc2475dcf05e9c8186f4286f5a28cde8727b5c9f5
ff2144c567b75705d96101133cdb0b135f819583db1e7f97833e2f53d62fa46af58bd0346bcc95e1
4eee5dc8a5b317c5bd44661130207727dbbd8374256660cb6328a5f310cbb1d502f819a07920e8f1
228129462b6ad0c1998bd712881731310ec5a62b146808c8725673555f6a3277e0c04e72c42cf7b8
8280262488571e5400589cbff5db0dd4ca127f0612aa37200ea3ca3dfacb78262b7e6094ad2988f4
846b8f1d9030f263d7fae43c1babe347cc2e2d5d2af682265aabcc89ccf10d57f61a5b49bcc21014
343dc6e1c86fbc7e10adcf4115bfac671552bdb1c43bfce04b3970ab114407e942f0f64e8ffe345d
407e8d890287cb4c435afd60585a74061f3a831eb7ef33ad1cd87dcc9bc0d0845efa66ed174b93ce
70074d2c9c3a49c9a26f729ee8f4206e4c803a4a9a02b28586a5cc74fffe95d314ddd733c075bf27
0011010001000ffb05692606ff1e7fd5f6a46714bd0f5f9734006d02ee9469ce30aaac96782007c7
39198223d47ba7e405d542bf0c613b4e631588e718f42b926404d72dffb4a97ef4e77b3e9a65921d
fdf0e0e1946dbf6831f553607cf9b64b878f2980e63cd68b6ab6880470c4782135758459110e20ec
26977e92b12b8174091ef296aadb04eac7ee749947c8bcb2cf4f9b059ceef66d3f76f3dc6af228f8
0d0a623bd0549fa17a860afa829e2aecde598171548e7fde6d98b572b4662ede6adc0e402f1372b4
1ed8b0ed48e5582f700d3a355c7da7c28f9869438e857c7007842c457e0d0fab95430d68d532a73f
9015af54d92fff2815793e6584e8fb98b5fdfa75118df03c08de3ae1fc71f579840986b6405dbcd1
5d6d1b10671bbbef96f81e33f3e4a2c13ad92eba6ae4c41747bf64aa0151222a2ae7e93dc0b87876
2814b751a7cded6208d3b1f468d05c8d42db2f29f83c878ff6b465c530ff708fdbc0aa7b4345df15
6f001e8b9eb78267f7d72c45abd244b4f75ecb77dd948309cddd7faee7b82804cbe9b41774544d14
e664c533fd4645dbeed33202520a92c028c98e87e8a972d2e9f9f6166c8c107169f1ee6e1e830482
2202ced7bd7b710021c7b100fdd8db9008bc8da3e5ad25644eb2cfb142d70eea66031ccc26a0eaa6
1aa71bd5caf52fe7323f5ec4367b254a34106bd57418228a4c58afd28fc8d2820d4ae8d5b2e6cb81
0800d382fbdfbf63724d4ac0f137912b096c67c2d2e3e522922494bf202ccab384b0adfb085cbdc3
bebed1b2d437dea25e5f15b6cf548664c2a1e97d5f73ed73ff84473d01a3123b89c8767b98461f68
a93b33b88e9d4f7be4ae21b232ffb91e55e3384d8c14e2845b66ab997024ee57b18efc50760ce4bc
0ab7d9fa7fa7c52b1ef39955433c6f6307c986c5011fd75aa321108f3374c50cbfd3dcc96fdf2eef
16dcfa9291468f896870f5a75abcb23a32982590538e6fd1cef78d3c1f62db9ddf2436f3928786ed
bd3221f37f99bd182b2e0e26373a2af74ae86a40e54551e9236262e25e3cd4564207277de55ba747
34e9ab531a3e7147568e9d7db0fa127ceac10800d6dfb45a341069bd1976811a9ca6b19f26db1622
03ecf56a1265ac0ba3a11c5b5cf873aa31525df972ab4f5ec496add03753eab7f82221a10cf778cb
38e4a8f44cf1b931665b55e7bf2aacc5ec3846712342d687fd06a0f70ff5924de74ffaafb9e299ca
8f96f7140ed54019ba7944eab5cd4e493e9e4133ca042df168f78395829e14572cbee64d41c6f45d
37ecc94f469ed2758a3bd52e3536faf10fe8f8032944f883e0a34c41104988b27ab371f92d44175a
d5f99f148dbebe87077537974d39061db9ce22db8e0a4a65b5943235ecaa4cdc8c6003d06690971b
675cb3ed511d3d58ec60af8d3dd83ab36fd311a31300229be926f88faca585bbe7afabe707ff7355
1638cc8a287e9866d4d8a79ec5c4995d72a559395015f925dbdb6b3379c9d0a582340565002b1a2a
eafdb696900d50d5a49440c61f459fbcb15611716500edac3a9b363c51557a0e4f545c8fd24803e9
8738bfff67a1058dfb6a344e9ac88e681facd9ce32427d8aca3da5a75f71337e8a7ca54284190cc4
0ecaae9dc1d9e0bda5068c1f3062aeec1ed4afe41f95ffd6035cba8ef4a7190fca05273dd93ab058
3a680c4dab53bd894903329a34e957dc0355770de5ed8f25a8fc26cde1ee387b4db64da6e72b1376
c0efe1ce198a0dae56d0cc5edf938a2d1f1ceaaf0cc920c78ac7adb05237a7a0df56257aeaa599a4
a8e270bedc67566f84765f8a6c9784cc
binary offset: 1819
flags: 0xb4
        always: 1
        new_type: 0
        length_type: old 0
        packet_length: 31
        packet_type: 13 User ID Packet
534543434f4e205f5f5f5f4b6579203c5f5f5f5f6b657940736563636f6e3e
        SECCON ____Key <____key@seccon>
binary offset: 1852
flags: 0x89
        always: 1
        new_type: 0
        length_type: old 1
        packet_length: 555
        packet_type: 2 Signature Packet
041301020015050200015180021b03020b09021502021e01021780000a0910f1d5d0238c2e1a7d2b
660fff71ccc64a2b2f08ea3c6107f51f78624231e986ce5d99718e2bb23a16c3b453c57882133fed
95a13c72e9bec2a8d46cf89726047ff07797ca30849899efd04e07a355189e3e6046758728c20f2e
638e355833945f2917ffe75fe687d69ab3628ffcfae4e0b3f31ebf604c70c92711dcedbdc6c98a54
8fd3e166f8e71f06475dfa35c9aed34eae6e2874e6565541b0578943edd3e11a6f8e877ce53160a1
68e41e0036c513efd0cfb7ebdef125b363ef9b0d086e7d7114491405e1ce66608992bc7611f4ccb6
06df3a9b46fe6aaa5bc38bbf2bc2458728a2048c7748cb4834dc03c5fcdf8b00497760dc64acc5f2
52edd59be08c3a55ceae80891eff5e5028b4684d465700cb1f656aab978e810af897765587c8caa3
9244c0755d06fbb5ad4f03ee431a1e4f0c8963565257f6a34fa4e8cd484377688b8848bb9a4bb30d
baeb166a2e73cc90adc77e283a20ec2c22ecba4ab4093fd7cb86a74997be7614c5194e4e1d7bf061
fa5a2afb600c8deb13914a953df68f77590cb6ba867bca52a0f352ba7b524c72d0f39714e7488cc1
0c1afebc6f3f7fa328bfc0be32db4e64edadbe5c5857d632b6ce61b9d89df7f9c876028124db3c2f
166367727d44ba5f67abe91de2b9640641d236b504797e27eb774d70391b6c66a26c47037fb92492
2fc3b2a208e0d4f208ceef49ef35b28095ee5e13e38c9ea97d2c952f368d28dfaab4a4
binary offset: 2410
flags: 0x9d
        always: 1
        new_type: 0
        length_type: old 1
        packet_length: 1816
        packet_type: 7 Secret-Subkey Packet
0400015180011000fd7fc49e2004f3b7ee67c36f7a86c80a2efee0390c581fbd4d745d9395b598a2
ee08bd5d337e97e80984566c2c853ef63ecc82b4945081ac16ebe627c2916f9659fb09913046e435
412576ad9d0ba1de4e8d99a4b2901eb0af70d5ac99ab30cf65212c506eb126c18e06875115524fa6
a8affc15b3d99a325e22e1f354b0cf6a9ac8d7d7af2aee29c2461632e370c422c122fbf581d071ec
5b7139deb783574745b96f406bcbc9d7d8181b4ae6baee0ed3f33018efb9ddca4292dd08a9dded9e
d7ca79d138d51d5daa4aea6a8d6004885a6ceb8a6302899ce497d263e856c98edea3ab9d68726821
ede349e453badda269f20ceca5859c44c9d6c43f9c11005e3dcf355308f0881b2a80491c334dfa86
b9e5cf8d8a886c74f30bf7df8d599fd93b33a3386a0ce4999dd08926bf0d4966aa7e52bdd3ff7d1b
687bb937bdbc19f1d80258a1ffe9b8e7d4c4d3b3e8080d82989435e079bf142a076d886df8f2a7ff
7f5b4cc1f47ceea89f147d6d31f08cf3098ae6f9a6c6ee7729b360297e6ef46c926635eb945a6c4f
2e8fb7cc72c533cd1f8bed9028c263ebdd7cab19b2949f81618ec9e00083996989022a8356e86a18
5f09f359056341fa922fbf19cf48053653479e9e286030b47b8c6289c5c914960b2b2035b2daf01b
8fe53825898981e77bac85bc1e79577daf1232fee8ef4a3eba23ca854b2de4898ae05d41cb43ee53
0011010001000ffe3fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
0800ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffff0800ffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff0800ffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffff6de7
        version: 4
        timestamp: 86400
        algorithm: 1
        values: fd7fc49e2004f3b7ee67c36f7a86c80a2efee0390c581fbd4d745d9395b598a2
        ee08bd5d337e97e80984566c2c853ef63ecc82b4945081ac16ebe627c2916f9659fb0991
        3046e435412576ad9d0ba1de4e8d99a4b2901eb0af70d5ac99ab30cf65212c506eb126c1
        8e06875115524fa6a8affc15b3d99a325e22e1f354b0cf6a9ac8d7d7af2aee29c2461632
        e370c422c122fbf581d071ec5b7139deb783574745b96f406bcbc9d7d8181b4ae6baee0e
        d3f33018efb9ddca4292dd08a9dded9ed7ca79d138d51d5daa4aea6a8d6004885a6ceb8a
        6302899ce497d263e856c98edea3ab9d68726821ede349e453badda269f20ceca5859c44
        c9d6c43f9c11005e3dcf355308f0881b2a80491c334dfa86b9e5cf8d8a886c74f30bf7df
        8d599fd93b33a3386a0ce4999dd08926bf0d4966aa7e52bdd3ff7d1b687bb937bdbc19f1
        d80258a1ffe9b8e7d4c4d3b3e8080d82989435e079bf142a076d886df8f2a7ff7f5b4cc1
        f47ceea89f147d6d31f08cf3098ae6f9a6c6ee7729b360297e6ef46c926635eb945a6c4f
        2e8fb7cc72c533cd1f8bed9028c263ebdd7cab19b2949f81618ec9e00083996989022a83
        56e86a185f09f359056341fa922fbf19cf48053653479e9e286030b47b8c6289c5c91496
        0b2b2035b2daf01b8fe53825898981e77bac85bc1e79577daf1232fee8ef4a3eba23ca85
        4b2de489
8ae05d41cb43ee53, 010001, fe3f, ffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffff0800ffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff0800ffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffff0800ffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff
ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff6de7

binary offset: 4229
flags: 0x89
        always: 1
        new_type: 0
        length_type: old 1
        packet_length: 543
        packet_type: 2 Signature Packet
041801020009050200015180021b0c000a0910f1d5d0238c2e1a7dc6780fff5dbe35e8c3362d8dc4
6bb6099823483694e8f71600aaa4c856f404d766301e30a84c20bff06c13eff4dc3ed8ff2ba7765d
5db828c863599096e53a2b0b9bb1b5c5cefcb06e82a1c67a2a995764dc37901470ce494e7c969cb1
da4ec072253ea5a0e7555216455bae05e4ee2f3504bbf224cc98801f68850286281a209d58ae2bc2
bf9b8434317536760137c53a9e609fb8ab7f61ce465cc8f9d76368b5c8e27034c06a060bad94d2be
6f5740a8d6646441b1d46f512bf07c87cf62dabba6bb86fef2871d4037c13a11b6afff8c040996de
d85c3125780a157074d2b4a7146f9b7222deb3b5f5da8443614deabb628900563dbbdafd55628269
457b3782f28e6d6541a277807456bffe6f4d14846e11bd6aab3fdf5fc8cefd5194636b5aceaf3075
c5cc748adca87e658796558f2d818574a0a088b25823ad443ce1450c6aee596ca844ae0a3955b0f8
492507044ac800133a13617aa692639f30f1c91db755a8568df021788ec448d35aaba8e39ad98b39
3dee3511e89917e7dcd4c26d0db0e8ac09c5a4906045c73bfb845249c72aa4ba44410ecee0582e12
1fb1d93651289d89388b5fbf62c44e5340cfacdb2e7d941feeaf340cfd900a5f92d1bed5ae6cf5e9
fc9fd80a257b4a413edbb7ec3b4774a4801082afe51385d3df2554458d706313c7007092f4c8fc38
98fd618d5a77cc32e6bbca39dc5a925def4a1848004318

gpg --import key.gpg.masked
gpg: key 8C2E1A7D: secret key imported
gpg: key 8C2E1A7D: "SECCON ____Key <____key@seccon>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
~~~~~~~~

## Step 3: Read The RFC for packets that we want to understand
[OpenPGP RFC 4880](https://tools.ietf.org/html/rfc4880)

{language=python}
~~~~~~~~
5.5.1.4.  Secret-Subkey Packet (Tag 7)

   A Secret-Subkey packet (tag 7) is the subkey analog of the Secret
   Key packet and has exactly the same format.

5.5.1.3.  Secret-Key Packet (Tag 5)

   A Secret-Key packet contains all the information that is found in a
   Public-Key packet, including the public-key material, but also
   includes the secret-key material after all the public-key fields.

5.5.1.1.  Public-Key Packet (Tag 6)

   A Public-Key packet starts a series of packets that forms an OpenPGP
   key (sometimes called an OpenPGP certificate).

5.5.2.  Public-Key Packet Formats

   There are two versions of key-material packets.  Version 3 packets
   were first generated by PGP 2.6.  Version 4 keys first appeared in
   PGP 5.0 and are the preferred key version for OpenPGP.

   OpenPGP implementations MUST create keys with version 4 format.  V3
   keys are deprecated; an implementation MUST NOT generate a V3 key,
   but MAY accept it.

...
   A version 4 packet contains:

     - A one-octet version number (4).

     - A four-octet number denoting the time that the key was created.

     - A one-octet number denoting the public-key algorithm of this key.

     - A series of multiprecision integers comprising the key material.
       This algorithm-specific portion is:

       Algorithm-Specific Fields for RSA public keys:

         - multiprecision integer (MPI) of RSA public modulus n;

         - MPI of RSA public encryption exponent e.

5.5.3.  Secret-Key Packet Formats

     - A Public-Key or Public-Subkey packet, as described above.

     - One octet indicating string-to-key usage conventions.  Zero
       indicates that the secret-key data is not encrypted.  255 or 254
       indicates that a string-to-key specifier is being given.  Any
       other value is a symmetric-key encryption algorithm identifier.

     - [Optional] If string-to-key usage octet was 255 or 254, a one-
       octet symmetric encryption algorithm.

     - [Optional] If string-to-key usage octet was 255 or 254, a
       string-to-key specifier.  The length of the string-to-key
       specifier is implied by its type, as described above.

     - [Optional] If secret data is encrypted (string-to-key usage octet
       not zero), an Initial Vector (IV) of the same length as the
       cipher's block size.

     - Plain or encrypted multiprecision integers comprising the secret
       key data.  These algorithm-specific fields are as described
       below.

       Algorithm-Specific Fields for RSA secret keys:

       - multiprecision integer (MPI) of RSA secret exponent d.

       - MPI of RSA secret prime value p.

       - MPI of RSA secret prime value q (p < q).

       - MPI of u, the multiplicative inverse of p, mod q.
~~~~~~~~

string-to-key usage conventions is 254.
symmetric encryption algorithm is 3f. Not set in the rfc. Weird.

At this point we have N, but we don't have p or q.

## Step 4: Attempt to factor N
There are only a few ways to factor N. Let's try two of the easier ones.

Shall we attempt to crack N? Sure.

{language=python}
~~~~~~~~
filename = 'ms.fortune/key.gpg.masked'
f = open(filename,'rb')
orig = open(filename,'rb').read()
pos = 2410
flags1 = orig[pos+0]
always = (flags1 & 0x80) >> 7
new_type = (flags1 & 0x40) >> 6
new_type
0
packet_type = (flags1 & 0x3c) >> 2
length_type = flags1 & 0x3
length_type
1
packet_length = (orig[pos+1] << 8) | orig[pos+2]
pos += 3
length_type
1
packet_length
1816
packet_type
7
d = parsegpg1.parseKey(orig[pos:pos+packet_length], packet_type)
import Crypto.Util.number
n = Crypto.Util.number.bytes_to_long(d[3][0])
sys.path.append('../mysterytwister')
import gnfs1
gnfs1.factor(n)
~~~~~~~~

gnfs1 is a simple python General Number Field Sieve implementation, I recommend you either write your own or get one.

gnfs1 failed to factor, but let's try a few other attacks.

{language=pycon}
~~~~~~~~
>>> fermat1.fermat2(n)
Trying: a=3215876357492628239969042742175159897482275015786600294286442763
48524373805400175864514938546617299093805187336491866243852067373363248131
09500237603304026009112696565510846849987937423619262973393969175056759821
65213886978321537816975783528358466084658320881272573383905913758094400268
61139127925696317969160697324317755993204583469378595898154975258286226226
52165709271152246464728489927670696601016248559515951932154686633599100945
31492183422732438195875118468497982424137525360686360189538365858248604536
35707554456298651940467008065420783788011363975777302476600700335171874395
37339428288763342861366560261430289 b2=-6431752714985256479938085484350319
79496455003157320058857288552697048747610800351729029877093234598187610374
67298373248770413474672649626219000475206608052018225393131021693699975874
84723852594678793835011351964330427773956643075633951567056716932169316641
76254514676781182751618880053722278255851392635938321394648635511986409166
93875719179630995051657245245304331418542304492929456979855341393202032497
11903190386430937326719820189062984366845464876391750236936995964848275050
72137272037907673171649720907271415108912597303880934016130841567576022727
95155460495320140067034374879074678856577526685722733120273757490
b=321587635749262823996904274217515989748227501578660029428644276348524373
80540017586451493854661729909380518733649186624385206737336324813109500237
60330402600911269656551084684998793742361926297339396917505675982165213886
97832153781697578352835846608465832088127257338390591375809440026861139127
92569631796916069732431775599320458346937859589815497525828622622652165709
27115224646472848992767069660101624855951595193215468663359910094531492183
42273243819587511846849798242413752536068636018953836585824860453635707554
45629865194046700806542078378801136397577730247660070033517187439537339428
288763342861366560261430289
(mpz(321587635749262823996904274217515989748227501578660029428644276348524
37380540017586451493854661729909380518733649186624385206737336324813109500
23760330402600911269656551084684998793742361926297339396917505675982165213
88697832153781697578352835846608465832088127257338390591375809440026861139
12792569631796916069732431775599320458346937859589815497525828622622652165
70927115224646472848992767069660101624855951595193215468663359910094531492
18342273243819587511846849798242413752536068636018953836585824860453635707
55445629865194046700806542078378801136397577730247660070033517187439537339
428288763342861366560261446073), mpz(3215876357492628239969042742175159897
48227501578660029428644276348524373805400175864514938546617299093805187336
49186624385206737336324813109500237603304026009112696565510846849987937423
61926297339396917505675982165213886978321537816975783528358466084658320881
27257338390591375809440026861139127925696317969160697324317755993204583469
37859589815497525828622622652165709271152246464728489927670696601016248559
51595193215468663359910094531492183422732438195875118468497982424137525360
68636018953836585824860453635707554456298651940467008065420783788011363975
77730247660070033517187439537339428288763342861366560261414507))
>>> _
(mpz(321587635749262823996904274217515989748227501578660029428644276348524
37380540017586451493854661729909380518733649186624385206737336324813109500
23760330402600911269656551084684998793742361926297339396917505675982165213
88697832153781697578352835846608465832088127257338390591375809440026861139
12792569631796916069732431775599320458346937859589815497525828622622652165
70927115224646472848992767069660101624855951595193215468663359910094531492
18342273243819587511846849798242413752536068636018953836585824860453635707
55445629865194046700806542078378801136397577730247660070033517187439537339
428288763342861366560261446073), mpz(3215876357492628239969042742175159897
48227501578660029428644276348524373805400175864514938546617299093805187336
49186624385206737336324813109500237603304026009112696565510846849987937423
61926297339396917505675982165213886978321537816975783528358466084658320881
27257338390591375809440026861139127925696317969160697324317755993204583469
37859589815497525828622622652165709271152246464728489927670696601016248559
51595193215468663359910094531492183422732438195875118468497982424137525360
68636018953836585824860453635707554456298651940467008065420783788011363975
77730247660070033517187439537339428288763342861366560261414507))
~~~~~~~~

Did fermat just...

{language=pycon}
~~~~~~~~
>>> p*q == n
True
~~~~~~~~

Yes, yes he did.

fermat1.py is a simple python implementation of fermat, very similar to the ones that can be found on Stack Exchange. Just saying.

## Step 5: Run recover.py to generate the key
quick run recover.py!

{language=python}
~~~~~~~~
python recover.py 32158763574926282399690427421751598974822750157866002942
86442763485243738054001758645149385466172990938051873364918662438520673733
63248131095002376033040260091126965655108468499879374236192629733939691750
56759821652138869783215378169757835283584660846583208812725733839059137580
94400268611391279256963179691606973243177559932045834693785958981549752582
86226226521657092711522464647284899276706966010162485595159519321546866335
99100945314921834227324381958751184684979824241375253606863601895383658582
48604536357075544562986519404670080654207837880113639757773024766007003351
7187439537339428288763342861366560261446073 321587635749262823996904274217
51598974822750157866002942864427634852437380540017586451493854661729909380
51873364918662438520673733632481310950023760330402600911269656551084684998
79374236192629733939691750567598216521388697832153781697578352835846608465
83208812725733839059137580944002686113912792569631796916069732431775599320
45834693785958981549752582862262265216570927115224646472848992767069660101
62485595159519321546866335991009453149218342273243819587511846849798242413
75253606863601895383658582486045363570755445629865194046700806542078378801
136397577730247660070033517187439537339428288763342861366560261414507
~~~~~~~~

Didn't work.

{language=python}
~~~~~~~~
python recover.py 321587635749262823996904274217515989748227501578660029428
644276348524373805400175864514938546617299093805187336491866243852067373363
248131095002376033040260091126965655108468499879374236192629733939691750567
598216521388697832153781697578352835846608465832088127257338390591375809440
026861139127925696317969160697324317755993204583469378595898154975258286226
226521657092711522464647284899276706966010162485595159519321546866335991009
453149218342273243819587511846849798242413752536068636018953836585824860453
635707554456298651940467008065420783788011363975777302476600700335171874395
37339428288763342861366560261414507 321587635749262823996904274217515989748
227501578660029428644276348524373805400175864514938546617299093805187336491
866243852067373363248131095002376033040260091126965655108468499879374236192
629733939691750567598216521388697832153781697578352835846608465832088127257
338390591375809440026861139127925696317969160697324317755993204583469378595
898154975258286226226521657092711522464647284899276706966010162485595159519
321546866335991009453149218342273243819587511846849798242413752536068636018
953836585824860453635707554456298651940467008065420783788011363975777302476
60070033517187439537339428288763342861366560261446073
~~~~~~~~

Worked because p < q.

{language=python}
~~~~~~~~
gpg --homedir=awwyeah/ --import key.gpg
gpg: WARNING: unsafe permissions on homedir `awwyeah/'
gpg: keyring `awwyeah//secring.gpg' created
gpg: keyring `awwyeah//pubring.gpg' created
gpg: key 8C2E1A7D: secret key imported
gpg: awwyeah//trustdb.gpg: trustdb created
gpg: key 8C2E1A7D: public key "SECCON ____Key <____key@seccon>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
gpg:       secret keys read: 1
gpg:   secret keys imported: 1

gpg --homedir=awwyeah/ encrypted.gpg
gpg: WARNING: unsafe permissions on homedir `awwyeah/'
gpg: encrypted with 4096-bit RSA key, ID 7DAD8D6C, created 1970-01-02
"SECCON ____Key <____key@seccon>"

cat encrypted
SECCON{g^2-f^2=(g+f)(g-f)~is~still~important~to~factor~BIG~numbers,
.025f0ddfdc463a24bf0350c15b175eee}
~~~~~~~~

## Conclusion
In general, the solution is to parse the OpenPGP secret key to get N, use Fermat's factorization method to find p and q, then run the provided recover.py with p and q. Because I had already written the OpenPGP parser and the Fermat function for another project, this only took 2-3 hours, most of it trying to be sure that I had N and not some other value. OpenPGP is a fairly flexible encryption suite with modes easy to manipulate for crypto puzzles like this one.

Many thanks to the SECCON CTF organizers who made a very fun CTF. 有り難うございます.

Since I use three custom scripts to solve this, I expect requests for source. To make it worth my while, encrypt your request with [my public key 3C68C8DBCBA783EF](https://www.altsci.com/gpg.html) and sign the request with your public key. I will be releasing my OpenPGP fuzzer in the near future, it will be announced on [Neg9's mailing list](https://neg9.org/mailman/listinfo/hackers).

Thanks for reading. It turns out that parsing OpenPGP and learning how to run Fermat's factorization method is actually worthwhile to the tune of 400 points.
