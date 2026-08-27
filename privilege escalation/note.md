# Linux Privilege Escalation - Basic Enumeration

## 1. System Enumeration

After gaining initial access, start by gathering basic information about the target system.

```bash
hostname
uname -a
uname
cat /proc/version
cat /etc/issue
ps aux
dpkg -l
```

Also check for scheduled tasks (cron jobs), as they are a common privilege escalation vector.

```bash
crontab -l
cat /etc/crontab
ls -la /etc/cron.*
```

---

## 2. User Enumeration

Gather information about the current user and other users on the system.

```bash
id
env
history
sudo -l
cat /etc/passwd
```

### List All Usernames

Extract all usernames from `/etc/passwd`:

```bash
cat /etc/passwd | cut -d ":" -f 1
```

If passwords are weak or reused, valid usernames may be useful for password attacks.

### List Local Users

Show users with home directories:

```bash
cat /etc/passwd | grep /home
```

This is often useful for identifying real local users while ignoring service accounts.

---

## 3. Network Enumeration

After user enumeration, collect networking information to identify internal services, interfaces, and connections.

Common commands:

```bash
ip a
ip route
ss -tulpn
netstat -tulpn
```

---

## 4. File Enumeration

Start with a quick directory listing:

```bash
ls -la
```

### Useful `find` Commands

Find files by name:

```bash
find / -iname "*tryhackme*" 2>/dev/null
```

Find `flag1.txt` in the current directory and all subdirectories:

```bash
find . -name flag1.txt
```

Find `flag1.txt` under `/home`:

```bash
find /home -name flag1.txt
```

Find a directory named `config`:

```bash
find / -type d -name config
```

Find files with `777` permissions:

```bash
find / -type f -perm 0777
```

Find executable files:

```bash
find / -perm -a=x
```

Find all files owned by user `frank`:

```bash
find /home -user frank
```

Find files modified within the last 10 days:

```bash
find / -mtime -10
```

Find files accessed within the last 10 days:

```bash
find / -atime -10
```

Find files changed within the last 60 minutes:

```bash
find / -cmin -60
```

Find files accessed within the last 60 minutes:

```bash
find / -amin -60
```

Find files larger than 50 MB:

```bash
find / -size +50M
```

---

### Enumeration Workflow

```text
1. System Enumeration
2. User Enumeration
3. Network Enumeration
4. File Enumeration
```

> Good enumeration solves more boxes than exploits. Always enumerate thoroughly before searching for privilege escalation techniques.
