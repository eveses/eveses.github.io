---
title: "[TryHackMe] walkthrough: Ice (Easy, Windows, metasploit)"
date: 2025-02-01
categories: [tryhackme]
tags: [thm, windows, easy, metasploit]
---

![title](/assets/images/tryhackme/Ice/스크린샷%202025-09-19%20오후%2010.23.17.png)

## Info

- Platform: Tryhackme
- Machine Name: Ice
- Target IP/Host: 10.10.192.111
- OS / Version: Windows
- Difficulty / Category: Easy

## Overview
간단한 윈도우 머신이고 metasploit의 여러가지 기능을 사용하여 풀 수 있다.

## Recon & Enumeration
먼저 -sCV 를 활용해 기본적인 스캔을 진행하면 다음과 같은 결과가 나온다.

```
PORT     STATE  SERVICE       VERSION
135/tcp  open   msrpc         Microsoft Windows RPC
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3389/tcp closed ms-wbt-server
5357/tcp open   http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Service Unavailable
8000/tcp open   http          Icecast streaming media server
|_http-title: Site doesn't have a title (text/html).
Service Info: Host: DARK-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Dark-PC
|   NetBIOS computer name: DARK-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2025-09-19T07:28:28-05:00
| smb2-time: 
|   date: 2025-09-19T12:28:27
|_  start_date: 2025-09-19T12:19:56
|_clock-skew: mean: 1h40m00s, deviation: 2h53m14s, median: -1s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: DARK-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:a3:12:d2:5d:eb (unknown)
```

## Initial Access
8000번 포트에 Icecast라는 것이 돌아가고 있고 metasploit으로 찾아보면 익스플로잇 하나가 나온다.

![msf](/assets/images/tryhackme/Ice/스크린샷%202025-09-19%20오후%2010.29.36.png)

필요한 옵션은 다음과 같다.

![options](/assets/images/tryhackme/Ice/스크린샷%202025-09-19%20오후%2010.30.40.png)

RHOSTS와 LHOSTS를 설정해준 후 run을 하게되면 미터프리터 세션을 얻을 수 있다.

계정 uid를 확인해보면 Dark라는 일반사용자인걸 알 수 있다.

![img](/assets/images/tryhackme/Ice/스크린샷%202025-09-19%20오후%2010.42.02.png)

metasploit을 이용하여 권한상승까지 해보자.

## Privilege Escalation

미터프리터에서 background를 입력하여 msf로 돌아가고 권한상승에 많이 사용되는 post/multi/recon/local_exploit_suggester 모듈을 사용한다.

해당 모듈은 타겟을 분석하여 권한상승에 사용될 수 있는 모듈을 찾아준다.

옵션을 보면 SESSION이라는게 존재하는데 이는 전에 획득한 미터프리터 세션을 입력하면 된다.

![op](/assets/images/tryhackme/Ice/스크린샷%202025-09-19%20오후%2010.49.30.png)

획득한 세션은 `sessions` 로 확인 가능하다.

세션을 설정해주고 run을 입력하여 실행하자.

![img](/assets/images/tryhackme/Ice/스크린샷%202025-09-19%20오후%2010.50.23.png)

사용 가능한 여러 모듈을 보여주고있다. 이중 `windows/local/ms14_058_track_popup_menu` 모듈을 사용해볼것이다.

`use windows/local/ms14_058_track_popup_menu` 로 모듈을 바꿔준 후 session과 lhost 옵션을 설정해준다.

![error](/assets/images/tryhackme/Ice/스크린샷%202025-09-19%20오후%2010.54.17.png)

하지만 에러가 난다. 에러 메시지를 확인해보면 WOW64(32비트를 64비트로 변환시켜주는 계층), 즉 32비트 세션을 사용하고 있기때문에 동작을 안하는 것이다.

그러므로 세션을 64비트로 변환해야 하고, `sysinfo` 명령어로 확인해보면 타겟 머신이 64비트 이므로 그에 따라 target과 payload도 64비트로 변환해야 한다.

세션부터 변환해보자.
먼저 `sessions -i 1`로 기존 미터프리터로 돌아온다.

다음으로 `ps`로 프로세스를 확인하고 이들 중 x64를 찾아 해당 pid를 이용하여 `migrate <pid>`를 하면 세션이 64비트로 변환된다.

![img](/assets/images/tryhackme/Ice/스크린샷%202025-09-19%20오후%2010.59.42.png)

다음으로 타겟을 변환하자.
타겟은 `show targets` 후 `set target 1` 으로 x64로 설정하면 된다.

마지막으로 페이로드를 변환하자.(타겟을 먼저 변경해야 해당 타겟에 맞는 페이로드가 나온다.)

페이로드는 x64, meterpreter, reverse를 사용할것이므로 

`grep x64 grep meterpreter grep reverse show payloads` 로 찾을 수 있다.

![img](/assets/images/tryhackme/Ice/스크린샷%202025-09-19%20오후%2011.06.23.png)

익스플로잇 실행 후 `getuid`를 입력하면 권한상승한 것을 확인할 수 있다.

![img](/assets/images/tryhackme/Ice/스크린샷%202025-09-19%20오후%2011.09.44.png)
