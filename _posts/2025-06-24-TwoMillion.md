---
title : "TwoMillion"
date : 2025-06-19 
layout : post
categories : [HTB]
tags : [TwoMillion]
author : jana
---

# Machine Name: TwoMillion

## Machine Summary 

**TwoMillion** is an easy Linux machine on Hack The Box, made to celebrate 2 million users. It’s beginner-friendly and brings back the old HTB invite system.

You start by visiting a website where you need to solve a JavaScript challenge to get an invite code. Once you're in, you explore the site and find an admin API that lets you run commands on the server (thanks to a command injection bug). This gives you your first shell.

Next, you discover some saved credentials in a hidden config file, which you can use to log in via SSH.

Finally, you take advantage of a known Linux kernel vulnerability (CVE-2023-0386) to become root.

---

## Reconnaisance:

### Nmap Scan
```bash
 nmap -sV -A -oN /TwoMillion/Nmap.txt 10.101.11.221
``` 

