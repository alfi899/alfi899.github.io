---
title: "HTB Timelapse"
date: 2025-07-05
categories: [HTB, Writeup, Machine]
tags: [Active Directory]
description: Easy Machine from HTB
pin: false
math: false
mermaid: true
#image:
#  path: /assets/img/posts/htb-machine-name/banner.png
#  alt: "HTB Machine Banner"
#  width: 800
#  height: 400
---

> **HTB Timelapse - Writeup**  
> Easy

---

This is another machine from the `Active Directory Track` from HackTheBox. It is ranked as easy.

## Recon
So let's begin and start with an nmap scan to see which ports are available to us. 

![Nmap Scan](/assets/img/htb/timelapse/nmap.png)

We can see, that there are some interesting ones. For the begin we see, that the SMB port is open at `445`. So we can try to log in anonymous and read some shares.

We can use `smblcient`for this task and give it the Flag `-L`to list the shares and `-N` to login in with no password prompt. If you leave the `-N` flag, you can just press enter when the password prompt opens. 


![smb anonymous](/assets/img/htb/timelapse/smb1.png)

And we see a bunch of Shares here. Besides the Standard ones we can try to read the other ones. I tried every one and found a interesting file in the `Shares` Share.
There are two Directories inside. `Dev` and `HelpDesk`. In the Dev Folder we find a file called `winrm-backup.zip`. Let's download it to our local machine. 

![smb share](/assets/img/htb/timelapse/smb2.png)

There where other files also but i couldn't find any useful informations in there. So I focused on the ZIP file. 
The File is passsword protected and we dont have any password found yet. 

![zip file](/assets/img/htb/timelapse/zip1.png)

After trying some standard Passswords I decided to try and bruteforce it. For that we can use `John The Ripper`. But in order to use it, we first need to generate a special hash of that zip file so that john can work with that. This can be done with the build in utility `zip2john`.

```bash
zip2john winrm_backup.zip > hash.txt
```

Then we can fire john with this hash file and a wordlist.

```bash
john hash.txt -wordlist:/usr/share/wordlists/rockyou.txt
```

And the tool is able to give us the correct Password. `supremelegacy`.

Let's unzip the file with the password we just cracked. Inside the ZIP file is a new file called `legacyy_de_auth.pfx`.

Fot those who don't know, a `PFX (Personal Information Exchange)` file is an encrypted security format that typically stores one or more password-protected digital certificates and private keys for authenticating a person, application, or device.
The file contains an SSL Certificate in `PKCS#12` Format and a private key. With the certificate and the private key we can authenticate us to the system without a password. But we first need to extract these from the files in singel `.pem` files to use them with evil-winrm.

This can be done with `openssl`. 

```bash
openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out key.pem -nodes
```

```bash
openssl pkcs12 -in legacyy_dev_auth.pfx -nokeys -out cert.pem
```

But there is another problem. This process wants us to input the corresponding password and it is not the same as for the zip file! 

![openssl](/assets/img/htb/timelapse/openssl1.png)

But we are lucky that john can also handle this. We just need to provide the right format with `pfx2john` and can give it a go.

```bash
pfx2john legacyy_dev_auth.pfx > pfx_hash.txt
```

And crack with john.

![john pfx crack](/assets/img/htb/timelapse/john2.png)

There it is! `thuglegacy`.

Now we can recreate the steps from before and extract the certificate and the private key from the PFX file.

After that we use `evil-winrm` to login to the machine.

![login with cert](/assets/img/htb/timelapse/login_cert.png)

From there we are able to read the first flag. 

## Privilege Escalation
From there on i didn't really have a clue what to do. The HTB hint referred to an interesting article. (https://0xdf.gitlab.io/2018/11/08/powershell-history-file.html)

It tells about an exploit of the usage from the `ConsoleHost_history.txt` file, which is the equivalent to the `.bash_history` file in Linux. Every PS-Commands are stored there. So it is worth a try to investigate this exact file. 
The default location for this is  
`$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`. 

![PS History](/assets/img/htb/timelapse/ps_history.png)

And we find another password and username set! `svc_deploy:E3R$Q62^12p7PLlC%KWaxuaV`

So we login with these credentials.

```bash
evil-winrm -i 10.10.11.152 -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV' -S
```

There was an interesint hint on HTB, which includes checking which Groups the user is part of. So let's take a look.
For that we use the following command: `net users svc_deploy`

![Net User](/assets/img/htb/timelapse/net_user.png)

On the bottom we see the user is part of `Remote Management Use`, `LAPS_Readers` and `Domain Users`.
The interessting one here is `LAPS_Readers`.

In `LAPS (Local Administrator Password Solution)` are the local admin passwords stored in `ms-Mcs-AdmPwd` and the Group members of `LAPS_Readers` have the rights to read these password. Nice!

For the easier usage we can use a PS Module that helps us to read these passwords. You can find it here: https://github.com/ztrhgf/LAPS/tree/master/AdmPwd.PS

So we download the file from GitHub and upload it to our evil-winrm session. `upload AdmPwd.PS`

Once the Module is uploaded we need to import it `Import-Module .\AdmPwd.PS` and use it. 

First we check which object we can manage with the command: 

```bash
Find-AdmPwdExtendedRights -identity *
```

![find admpwd rights](/assets/img/htb/timelapse/admpwd1.png)

We see that we can manage a lot. But for us it is interesting to see the local admin from the Domain Controller. And we already know the name of that from the nmap Scan. It is `dc01`.

So we can use the following command to extract the password written in `ms-Mcs-AdmPwd`:

```bash
get-admpwdpassword -computername dc01 | Select password
```

![adm password](/assets/img/htb/timelapse/admpwd2.png)

And with that we can Login as administator and read the final flag.

```bash
evil-winrm -i 10.10.11.152 -u administrator -p '6y@fhye-7)xP%Z]eVV@7$zKe' -S
```
![root flag](/assets/img/htb/timelapse/final.png)


