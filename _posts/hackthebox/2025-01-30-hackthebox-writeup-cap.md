---
title: "[HackTheBox] walkthrough: Cap (Easy, Linux)"
date: 2025-01-30
categories: [hackthebox]
tags: [htb, linux, easy]
---

![head](/assets/images/wrieups/cap/스크린샷%202025-08-23%20오후%206.36.20.png)

## Info

- Platform: Hackthebox
- Machine Name: Cap
- Target IP/Host: 10.10.10.245
- OS / Version: Linux
- Difficulty / Category: Easy

## Overview

Cap is an easy Linux retired machine from HackTheBox. You can get initial access by finding credential in a pcap file, and escalate privileges using Linux capabilities.

Let’s start !

## Recon & Enumeration

First I did basic nmap scanning to find open ports. And I got 3 open ports FTP(21), SSH(22), HTTP(80).

![nmap](/assets/images/wrieups/cap/스크린샷%202025-08-23%20오후%206.40.14.png)

Let’s check port 80 first.

![web](/assets/images/wrieups/cap/스크린샷%202025-08-23%20오후%206.40.59.png)

The website shows I’m logged in as ‘Nathan’ but I can’t log out or access any user settings.

I only can access the menu and download packet capture files of my traffic.

![web](/assets/images/wrieups/cap/스크린샷%202025-08-23%20오후%206.41.59.png)

While looking around, I found that the URL index number increases when I refresh the page. Since the index number started at 1, so I decided to check index 0.

![web](/assets/images/wrieups/cap/스크린샷%202025-08-23%20오후%206.42.41.png)

I was able to download the packet capture file from index 0, and I opened this packet file with wireshark.

## Initial Access

![wireshark](/assets/images/wrieups/cap/스크린샷%202025-08-23%20오후%206.43.48.png)

I found user credential in the packet file and tried them on SSH since users often reuse their credentials. It worked !

![linpeas](/assets/images/wrieups/cap/스크린샷%202025-08-23%20오후%206.44.53.png)

In the user home directory, I found the user flag.

## Privilege Escalation

While the standard approach would be using the ‘getcap’ command, I found a linpeas script already downloaded by someone else(probably due to a machine reset issue)

So I executed the linpeas script and noticed that the Python binary file had ‘cap_setuid’ capabilities. This means that with this binary, I can use the setuid system call to obtain root privilege.

Press enter or click to view image in full size

![linpeas](/assets/images/wrieups/cap/스크린샷%202025-08-23%20오후%206.45.47.png)

I looked through GTFOBins, which is collection of Unix binaries that can be used to bypass local security. There I found how to escalate privileges using Python capabilities.

![gtfo](/assets/images/wrieups/cap/스크린샷%202025-08-23%20오후%206.46.28.png)

Finally, I got the root flag !

![flag](/assets/images/wrieups/cap/스크린샷%202025-08-23%20오후%206.47.12.png)

## Conclusion

Through this machine, I learned that I should try various values in URL parameters and directories as they might reveal unexpected information. I also discovered that Linux capabilities can be used for privilege escalation, not just setuid.