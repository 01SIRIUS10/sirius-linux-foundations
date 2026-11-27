<div align="center">
<img src="https://raw.githubusercontent.com/01SIRIUS10/01SIRIUS10/main/Banner.png" width="100%" alt="SIRIUS Banner" />

# 🐧 sirius-linux-foundations
### From zero to comfortable in the Linux shell — and from standard user to root.
**It sounds serious, until you look up.**

![Linux](https://img.shields.io/badge/Linux-Internals-1f2937?style=for-the-badge&logo=linux&labelColor=111827)
![Bandit](https://img.shields.io/badge/Bandit-0%E2%86%9233_Done-1f2937?style=for-the-badge&logo=overthewire&labelColor=111827)
![PrivEsc](https://img.shields.io/badge/Privilege-Escalation-1f2937?style=for-the-badge&logo=tryhackme&labelColor=111827)
![Approach](https://img.shields.io/badge/Approach-Root_Cause_&_Remediation-1f2937?style=for-the-badge&logo=gitbook&labelColor=111827)

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=18&pause=1000&color=58A6FF&center=true&vCenter=true&width=500&lines=From+Zero+to+Kernel+Exploitation;OverTheWire+Bandit+0+%E2%86%92+33;Building+a+Real+Red+Team+Mindset" alt="Typing SVG" />
</div>

## 📖 About This Repo
This is my personal documentation for mastering Linux. I started with the basics of the command line and moved into deep privilege escalation vectors. Every write-up here isn't just a list of commands—it's a breakdown of **why** a vulnerability exists and **how** to fix it.

## 🛠️ Journey Structure

| Phase | Topic | Focus | Status |
| :--- | :--- | :--- | :--- |
| **01** | Linux 101 | TCM Security & THM Pre-Security Path | ✅ Done |
| **02** | Wargames | OverTheWire Bandit (Levels 0 → 33) | ✅ Done |
| **03** | PrivEsc | THM Linux Privilege Escalation Path | 🔄 In Progress |

---

## 🚀 Privilege Escalation Labs
Detailed walk-throughs for different privilege escalation vectors. 
**Active sprint: 9 / 10 labs completed**

| # | Vector | Core Concept / Tools | Status | Documentation |
|:--|:---|:---|:---|:---|
| 01 | Manual Enumeration | `hostname`, `uname -a`, `id`, `ss -tulnp`, `procfs` | ✅ Done | [Read →](./03-privilege-escalation/01-manual-enumeration.md) |
| 02 | Automated Recon | `LinPEAS`, `LinEnum`, `lse.sh` | ✅ Done | [Read →](./03-privilege-escalation/02-automated-recon.md) |
| 03 | Kernel Exploits | `CVE-2015-1328` (`OverlayFS`, `gcc`, `ld.so.preload`) | ✅ Done | [Read →](./03-privilege-escalation/03-kernel-exploits-cve-2015-1328.md) |
| 04 | Sudo Abuse | `sudo -l`, `find -exec`, Password Cracking | ✅ Done | [Read →](./03-privilege-escalation/04-sudo-abuse.md) |
| 05 | SUID Binaries | `find / -perm -u=s`, `base64` Arbitrary Read | ✅ Done | [Read →](./03-privilege-escalation/05-suid-binaries.md) |
| 06 | Capabilities | `getcap -r /`, `cap_setuid+ep`, `vim` Breakout | ✅ Done | [Read →](./03-privilege-escalation/06-capabilities.md) |
| 07 | Cron Jobs | Writable Scripts, SUID Bash Drop, `backup.sh` | ✅ Done | [Read →](./03-privilege-escalation/07-cron-jobs.md) |
| 08 | PATH Variable | `$PATH` Hijacking, `system("thm")`, Unquoted Binaries | ✅ Done | [Read →](./03-privilege-escalation/08-path-variable.md) |
| 09 | NFS Root Squashing | `no_root_squash`, Remote RPC Mount, SUID Injection | ✅ Done | [Read →](./03-privilege-escalation/09-nfs-root-squashing.md) |
| 10 | Capstone Challenge | Comprehensive PrivEsc & Master Triage Runbook | ⬜ Up Next | — |

---

## 📝 Lab Documentation Standard
Every lab write-up follows a strict structure to ensure deep understanding:
1. **Goal:** What are we trying to achieve?
2. **Concept:** How does the technology work normally?
3. **Enumeration:** How did we find the vulnerability?
4. **Exploitation:** Step-by-step commands and logic.
5. **Root Cause:** Why was this possible at the code/OS level?
6. **Remediation:** How would a professional fix this?

## 📍 Where This Fits
This repository is **Stop #2** in my cybersecurity journey.
*   **Stop #1:** [SIRIUS-Net-Zero](https://github.com/01SIRIUS10/SIRIUS-Net-Zero) (Networking Basics)
*   **Stop #2:** **sirius-linux-foundations** (Linux & PrivEsc) 📍
*   **Stop #3:** [SIRIUS-Python-Journey](https://github.com/01SIRIUS10/SIRIUS-Python-Journey) (Security Tooling)

---
<div align="center">

[![LinkedIn](https://img.shields.io/badge/Connect-01SIRIUS10-0077B5?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/01sirius10)

*Sirius is the brightest star in the night sky. Keep looking up.* ✨
</div>
