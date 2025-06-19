---
title : "Pickle Rick"
date : 2025-06-19 
layout : post
categories : [THM]
tags : [PickleRick]
---
# Machine name: Pickle Rick

## Recon:

## Nmap Scan
```bash
 nmap -sV -A -oN nmap.txt
```
- nmap found ports 22(ssh) , 80(http)
  22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 
  80/tcp open  http    Apache httpd 2.4.41 
## Gobuster Scan

 ` gobuster dir -u http://10.10.127.115 -w /usr/share/wordlists/dirb/common.txt `

- gobuster found /login.php and /robots.txt

## Source Code Analysis

 login.php HTML source revealed the username: R1ckRul3s

 /robots.txt contained the password: Wubbalubbadubdub

## Exploitation
- Logged into login.php and redirected to portal.php, which allowed command execution.

- try to execute linux commands on the command panel

- get the first ingredient through `curl http://10.10.127.115/Sup3rS3cretPickl3Ingred.txt`
 * Returned: mr. meeseek hair

- we have the full permission PASSWORD : NO by check ` sudo -l ` command

- get the second & third incgedient by copy paste into the directory

  `sudo cp /home/rick/second\ ingredients ./second.txt`
  `sudo cp /root/3rd.txt .`
  
 * Opened second.txt: 1 jerry tear
 * Opened third.txt: fleeb juice

## Lessons Learned

- Enumeration is the keypart of pentest.
- Notedown the directory permissions are crucial while in privilege escalation.
- simple web forms also can be exploitable.

## Tools Used

- nmap  
- gobuster  
- linux commands  ( cat , cp , ls , sudo )

## Flags

- Flag 1: mr. meeseek hair
- Flag 2: 1 jerry tear
- Flag 3: fleeb juice
