---
title: "Password cracking"
date: 2025-02-21
categories: [cheatsheet]
tags: [cracking]
---

## Hashcat
기본 (워드리스트) -a 0
```
hashcat -m <mode> -a 0 hashfile.txt wordlists/rockyou.txt --status --status-timer=10
```

마스크 모드 -a 3
```
# 예시 모드 1000, 6글자(소문자4 + 숫자2)
hashcat -m 1000 -a 3 hashfile.txt ?l?l?l?l?d?d
```

룰 적용 -r
```
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r rules/best64.rule
```

`hashcat -h`로 해시 모드 확인 가능

`hashid -m <hash>`로 해시 모드 유추 가능

## John The Ripper
기본 (워드리스트)
```
john --wordlist=<path_to_wordlsit> --format=<type> hashfile.txt 
```

\*2john

암호가 걸린 여러 서비스 크래킹 가능 (ex zip, pdf, office ..)
```
zip2john secret.zip > zip.hash
john --wordlist=<path_to_wordlist> zip.hash
john --show zip.hash
```

## cewl
대상 웹페이지에서 키워드 추출해 커스텀 워드리스트 생성

```
# 페이지 깊이 2, 최소 단어길이 4, 출력파일
cewl -d 2 -m 4 -w cewl_words.txt https://example.com
```