---
title: "[TryHackMe] walkthrough: Pickle Rick (Linux)"
date: 2025-02-23
categories: [tryhackme]
tags: [thm, linux, easy]
---

![head](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%207.53.03.png)

## Info

- Platform: Tryhackme
- Machine Name: Pickle Rick
- Target IP/Host: 10.10.22.197
- OS / Version: Linux
- Difficulty / Category: Easy

## Overview

As a beginner for pentest, I decided to try solving some popular and easy machines on tryhackme. Today I chose ‘Pickle Rick’.

This is a simple and straightforward machine where the main concept appears to be reverse shell exploitation.

## Recon & Enumeration

![nmap](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%207.54.01.png)

I scanned machine and found two open ports: 22(SSH), 80(HTTP), I decided to check out the webpage first.

![web](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%207.54.30.png)

Looking at the website, it seems I need to logon Rick’s computer and find three ingredients (flag).

I inspected the page source code and found what appears to be Rick’s username.

![web](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%207.54.57.png)

And I checked robots.txt and found an interesting string.

![txt](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%207.55.41.png)

Since I couldn’t found any other useful information on this website, I started directory brute forcing and discovered login page.

![dir](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%207.56.17.png)

![login](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%207.56.47.png)

I entered the username from the page source code and the password form robots.txt, and successfully logged in.

There was a command panel where I could execute system command.

![web](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%207.57.18.png)

However, I was unable to view any files using this command panel.

![web](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%207.57.58.png)

## Initial Access

I tried to use reverse shell script written in sh (opened listening port on my system first), but it doesn’t work.

![sh](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%207.58.58.png)

Since sh wasn’t working, I checked if python3 was available by running ‘which python’ and found python3.

![rev](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%207.59.19.png)

I found python reverse shell script online and successfully executed it.

![rev](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%207.59.50.png)

To get more stable shell environment, I upgrade it to a fully interactive TTY.

And I found the first flag in the /var/www/html directory.

![flag](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%208.00.20.png)

Checked Rick’s home directory and discovered second flag.

![flag](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%208.00.57.png)

## Privilege Escalation

I suspected that the last flag would be in the root directory, so I checked sudo permissions using ‘sudo -l’.

![priv](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%208.01.34.png)

I found ‘NOPASSWD: ALL’ in the sudo configuration, which allowed me to easily escalate privileges and obtained last flag from the root directory.

![flag](/assets/images/tryhackme/pickle_rick/스크린샷%202025-08-23%20오후%208.01.59.png)

## Conclusion

I learned that there are various types of reverse shells available, and when one doesn’t work, it’s important to try different scripts rather than giving up after the first attempt.