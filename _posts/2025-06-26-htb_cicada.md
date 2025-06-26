---
title: "HTB Cicada"
date: 2025-06-24
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

> **HTB Cicada - Writeup**  
> Easy

---
## Recon
First off all we begin with an NMAP Scan to see which ports are open it what we can exploit or focus on. For that we run `nmap -sV -sC 10.10.11.35`

![Nmap Scan](/assets/img/htb/cicada/nmap_Scan.png)

We can see a lot of open Ports. The most interesting here are `Kerberos (port 88)` and `LDAP/S (ports 389, 636, 3268, 3269)`.

Since we know this is a Windows host and we are dealing with an Active Directory, one first good step is to check, if we can login as an anonymous User and access the SMB shares. 

For this we can use the tool `smbclient` and login with no password.

![smbclient shares](/assets/img/htb/cicada/smb_share.png)

Here we see a handfull. Now we can try to see if we can read some of these shares. Since the shares with a `$` are default shares, we try the other ones. Like `HR`.

And we can! There is also a file that we can just download with the `get`command and analyse it on our local machine. 

![HR Notice](/assets/img/htb/cicada/HR.png)

If we view the file we can see, that there is a introduction on how to create a new account in the company context. Further we also see the initial default password which is used for that. 

![HR Notice2](/assets/img/htb/cicada/Notice.png)

## Enumeration
No that we have this default password, we can take a look and check if some users still use that password. For that we have to find all users available within the domain. This is exactly what `Impackets lookupsid` can do for us. This tool will bruteforce the `Windows Security Identifiers (SIDs)`of users in the AD. 
But to do that we need to specify a user, in which context the tool operates. Since we known that we can read the SMB shares anonymously we can use the user `guest`, which is a built in user and if the user is not specific disabled, we can use him.

![Lookup SID](/assets/img/htb/cicada/lookupsid.png)

We get an overview of all kind of information like groups, users and aliases. For now we are just intersted in usernamen. So every entry with the `SidTypeUser`. 
If we extract all that usernames and put it in a file we get the following `users.txt`.

![Users](/assets/img/htb/cicada/users.png)

## Password Spray
Since we have a list of usernames and known the default password, we can perform a password spray attack and see if one of the users is still using it. For that we use `netexec`. 

![Passsord Spray](/assets/img/htb/cicada/spray.png)

And we see that the user `michael.wrightson` is still using that password! With that we can use his account and enumerat the other users on the machine.

![User enumeration](/assets/img/htb/cicada/user_enum.png)

And there is an interesing Notice. The user `david.orelious` has his password stored there, in case he forgots it.

Now we have a user to login to and see if we can access other shares. No just some with the default user `gast`.

Since we tried the `HR`before, we can now try the `DEV`.

![Dev share](/assets/img/htb/cicada/dev.png)

There is a Powershell script which we can download again. 

![Script](/assets/img/htb/cicada/script.png)

And we have another username and password pair in there. `emily.oscars:Q!3@Lp#M6b*7t*Vt`.


So let's try to get a shell on the machine with this credentials.
We can use `evil-winrm` for that and find the first flag in the Desktop folder.

![Flag1](/assets/img/htb/cicada/flag1.png)

## Privilege escalation

The first step is mostly to check which privileges we have with our current user. On Windows we can check this with the `whomai /priv` command.

![Privielges](/assets/img/htb/cicada/prives.png)

And we see that there are several privleges enabled for that user. But there is one intersting one. `SeBackupPrivilege`. Because this is a privilege that normaly administrators get. With that privilege Users have access to sensitive files like the Windows Registry Hives. And within these Hives are all the Hashes stored. Nice!

There are two mainly interesting hives for us. The `SAM (Security Account Manager)`, where local user accounts and group memeberships are stored. Including their hashed password.

Another one is the `SYSTEM` hive, where settings and configurations are stored, such as the boot key which is required to decrypt the hashed passwords stored in `SAM`.

After some online research I found out, that we can use the `reg save` command, to perform a command in the registry and save it to another file, which we can download to our local machine.

![Registry](/assets/img/htb/cicada/hives.png)

If we copied the registries to the files and downloaded them to our local machine, we can try to dump the `NTLM hashes`. These are a cryptographic representation of the user's plaintext passsord. And if we have the hashes of other users, we can perform a `Pass-the-Hash attack` and login with just this hash.

For that we use once again the impacket tool with the module `secretsdump`. Also we provide the script the `SAM`and `SYSTEM`file, so that it can dump the hashes.

![NTLM Hashes](/assets/img/htb/cicada/hashed.png)

And we can see, that the tool extracted the NTLM hash from the `Administrator` Account. Perfect!

With that we can once again login to the machine, but now with Admin rights and find the final Flag in the Desktop Folder.


![Flag 2](/assets/img/htb/cicada/flag2.png)


