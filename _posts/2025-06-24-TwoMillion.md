---
title : "TwoMillion"
date : 2025-06-25 
layout : post
categories : [HTB]
tags : [TwoMillion]
author : jana
---

# Machine Name: TwoMillion

## 🎯 Objective

The goal of this challenge is to gain initial access through web vulnerabilities, perform lateral movement if necessary, and escalate privileges to obtain the root flag.

---

## 🔎 1. Reconnaissance

### 🧪 Nmap Scan

```bash
nmap -sC -sV -oN nmap/twomillion.txt 10.10.11.215
```

**Results:**

```
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu
80/tcp   open  http     nginx 1.18.0 (Ubuntu)
```

### 🌐 Web Recon

Visited `http://10.10.11.221/`:

- Static page showing a celebration for 2 million users.
- Find the invite page and look at source through browser
![image](/assets/img/CTFS/TwoMillion/web_source_network_tab.png)

## Found
```
inviteapi.min.js
```
- Extracted two functions : `makeInviteCode()` and `verifyInviteCode()`.

---

### 🚀 Invite Code Generation

```bash
curl -X POST http://2million.htb/api/v1/invite/how/to/generate
```

Returned ROT13-encoded hint → decode to get next path → POST to `/api/v1/invite/generate` returns Base64 token.

```bash
curl -s -X POST http://2million.htb/api/v1/invit/generate | jq -r '.data.code' | base64 -d
```

Resulting invite code grants registration access to the site.

---

## 🛠 3. API Enumeration & Admin Access

While logged in as a user, enumerated API via `/api/v1`:

- Found `/api/v1/admin/auth` (GET)  
- `/api/v1/admin/vpn/generate` (POST)  
- `/api/v1/admin/settings/update` (PUT)

Upgraded user to admin by sending JSON payload to `/api/v1/admin/settings/update` with `"is_admin":1`.

![image](/assets/img/CTFS/TwoMillion/admin_pri_update.png)

---

## 💥 4. Command Injection via VPN API

Sent payload to `/api/v1/admin/vpn/generate`, injecting `;pwd;` allowed command execution before VPN file generation.

![image](/assets/img/CTFS/TwoMillion/cmd_inj.png)

Gained shell as `www-data` via netcat.

![image](/assets/img/CTFS/TwoMillion/rev_shell.png)

---

## 🧬 5. Privilege Escalation Path

1. Found `.env` credentials:

```bash
DB_USERNAME=admin  
DB_PASSWORD=SuperDuperPass123
```

Used credentials to SSH as `admin` and capture user flag.

2. Noticed kernel vulnerability (OverlayFS CVE‑2023‑0386) based on email in `~/mail` and kernel version (`5.15`).

![image](/assets/img/CTFS/TwoMillion/mail.png)

3. Ran publicly available exploit → gained root privileges → captured root flag.
![image](/assets/img/CTFS/TwoMillion/Root-access.png)
---

## ✅ 6. Summary of Attack Flow

| Stage              | Technique / Tool             | Outcome                             |
|--------------------|------------------------------|--------------------------------------|
| Recon              | Nmap, Hosts file, Browser    | Mapped target, services discovered   |
| Code Analysis      | inviteapi.js, ROT13, Base64  | Created valid invite                 |
| API Enumeration    | Burp/curl                    | Found admin API endpoints            |
| Privilege Escalation | JSON API, VPN payload      | Elevated to admin, command injection |
| Lateral Movement   | .env creds, SSH              | Logged in as admin                   |
| Root Escalation    | OverlayFS CVE exploit        | Gained root shell                    |

---

## ⚙️ Tools Used

- Nmap  
- Gobuster  
- curl  
- Burp Suite  
- Base64, ROT13 decoding  
- OverlayFS exploit (CVE‑2023‑0386)  

---

## 🧠 Lessons Learned

- Obfuscated JavaScript often guards crucial functionality (e.g., invite code)
- Inspecting APIs can reveal hidden admin controls
- JSON arguments like `is_admin` may be exploitable
- Monitor kernel vulnerabilities like OverlayFS for privilege escalation opportunities

---

> ✍️ *Enjoyed this walkthrough? Subscribe for more HTB lab write-ups and real-world pentesting content.*

---