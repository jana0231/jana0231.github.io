---
title : "Linux Backdoors Part IV"
date : 2025-08-18
layout : post
categories : [BACKDOOR]
tags : [PAM, BACKDOOR, PERSISTENCE, LINUX]
author : jana
---

# PAM Backdoors: Unlocking Persistence Through `pam_unix.so`

Hey hackers & defenders! 👋  

Today we’re exploring a persistence technique that’s both **surgical and terrifyingly effective**:  
backdooring the **PAM (Pluggable Authentication Module) system**, specifically the **`pam_unix.so`** module.  

If `.bashrc` was a magician’s sleight of hand, and SSH keys were a hidden spare key, then **PAM backdoors are like secretly rewriting the lock itself** 🔐.  

---

## 🔍 What Is PAM?

**PAM (Pluggable Authentication Modules)** is the authentication framework used by Linux and UNIX-like systems.  
Every time you log in, sudo, or authenticate in any way, PAM modules decide whether you’re legit.  

- Configs live in `/etc/pam.d/`  
- Core modules include things like `pam_unix.so` (handles username/password auth)  
- PAM decides who gets in — which makes it a *perfect* backdoor target  

---

## 🧠 Why `pam_unix.so`?

The **`pam_unix.so`** module validates user credentials against `/etc/shadow`.  
If an attacker can **patch or replace this file**, they can silently introduce *master passwords* or hidden login bypasses.  

In other words: they can make PAM *always say “yes”* for their secret password, no matter the user. 🤯  

---

## 🧰 The Attack: Step-by-Step  

### ✅ Step 1: Root Access  
PAM manipulation requires **root privileges**. This isn’t your average user-level persistence — the attacker is already deep inside.  

### ✅ Step 2: Backdooring `pam_unix.so`  
An attacker could:  
- Replace `/lib/security/pam_unix.so` (or `/lib/x86_64-linux-gnu/security/pam_unix.so`) with a trojanized version  
- Inject custom code that accepts a **“magic password”** for *any account*  

For example, a modified check might look like:  

```c
if(strcmp(password, "SuperSecretBackdoor123!") == 0) {
    return PAM_SUCCESS; // Bypass authentication
}
```

This tiny change means the attacker can log in as *any user* with their secret master password.  

### ✅ Step 3: Silent Persistence  
- The system behaves normally for everyone else  
- Legit users still log in with their real passwords  
- Logs don’t raise suspicion — unless defenders dig into PAM binaries  

---

## 🔁 Attack Flow Recap  

| Step              | Action                                       |
|-------------------|---------------------------------------------|
| 🎯 Initial Access | Attacker gets root on the system            |
| 📝 Modify PAM     | Injects backdoor into `pam_unix.so`         |
| 🔓 Secret Login   | Attacker logs in with a magic backdoor pwd  |
| 🕵️ Stealth Mode   | Users + admins see no difference            |

---

## 🕵️ Why It’s Scary  

✅ Runs at the **authentication layer** (the deepest trust point)  
✅ Works for **any user account** (including root)  
✅ Survives reboots and config resets  
✅ Invisible to casual file inspection — the binary looks legit  

This is the **Ferrari of backdoors**: rare, fast, and built for attackers who already have elite access.  

---

## 🧪 Lab Example (Educational Use Only!)  

⚠️ For safety: **don’t try this on production systems**.  

Instead, spin up a test VM and recompile PAM with a small patch:  

```bash
# Download PAM source
apt-get source libpam-modules

# Edit pam_unix/passverify.c to accept a hardcoded password
nano modules/pam_unix/passverify.c

# Recompile and replace pam_unix.so
make && cp pam_unix.so /lib/x86_64-linux-gnu/security/pam_unix.so
```

Now, logging in with your “magic password” works on any account. 😈  

---

## ⚠️ Limitations  

- ❌ Requires **root privileges** upfront  
- ❌ Changes can be detected with integrity monitoring (hash mismatches)  
- ❌ Advanced defenders may compare PAM binaries against known-good packages  
- ❌ Updates/upgrades of PAM overwrite the backdoor  

---

## 🛡️ Defense Strategies  

- **File Integrity Monitoring:**  
  Track cryptographic hashes of `/lib/security/pam_unix.so` and compare against vendor packages.  

- **Package Verification:**  
  Use tools like `debsums`, `rpm -V`, or Tripwire/AIDE to detect tampering.  

- **Central Authentication:**  
  Offload auth to Kerberos/LDAP where PAM modules are less directly trusted.  

- **Golden Rule:**  
  If PAM is backdoored, the system should be considered **fully compromised**. Rebuild, don’t patch.  

---

## 🧠 Final Thoughts  

`pam_unix.so` backdoors are the **nuclear option of persistence**.  
They’re subtle, powerful, and almost invisible without careful monitoring.  

- Red teamers: it’s a persistence trick to use sparingly, but it shows the importance of defense in depth.  
- Blue teamers: remember that **authentication is the crown jewel** — protect your PAM stack at all costs.  
- Learners: this is where offense meets defense in its purest form.  

---

⚠️ **Disclaimer:** This post is for **educational purposes only**. Do not attempt on any system you don’t own or have permission to test.  

Stay safe, respect authentication, and never forget: if you control PAM, you control the kingdom. 👑  
