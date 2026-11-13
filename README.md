<div align="center">

<img src="https://raw.githubusercontent.com/01SIRIUS10/01SIRIUS10/main/Banner.png" width="100%" alt="SIRIUS Banner" />

# 🐧 sirius-linux-foundations

**From zero to comfortable in the Linux shell — and from standard user to root.**  
*Documenting internals, wargames, and real-world local privilege escalation.*

<img src="https://readme-typing-svg.demolab.com?font=Fira+Code&weight=500&size=18&pause=1200&color=58A6FF&center=true&vCenter=true&width=640&lines=Linux+Internals+%26+Shell+Fundamentals;OverTheWire+Bandit+Wargame+Completed;Local+Privilege+Escalation+Labs." alt="Typing SVG" />

<br>

![Linux](https://img.shields.io/badge/Linux-Internals-1f2937?style=for-the-badge&logo=linux&logoColor=FCC624&labelColor=111827)
![Bandit](https://img.shields.io/badge/OverTheWire-Bandit%20Done-1f2937?style=for-the-badge&logo=gnubash&logoColor=4EAA25&labelColor=111827)
![PrivEsc](https://img.shields.io/badge/Privilege-Escalation-1f2937?style=for-the-badge&logo=kalilinux&logoColor=557C94&labelColor=111827)
![Approach](https://img.shields.io/badge/Approach-Root%20Cause%20%2B%20Remediation-1f2937?style=for-the-badge&logo=hackthebox&logoColor=9FEF00&labelColor=111827)

</div>

---

## 📖 About This Repo

Before you can break a server or escalate privileges, you have to understand how the operating system actually breathes. This repository started as my ground-zero documentation for Linux command-line fundamentals and the 34 levels of OverTheWire's Bandit wargame.

Now, it serves as my active lab notebook for **Linux Local Privilege Escalation** — moving beyond basic commands to dissect kernel vulnerabilities, permission misconfigurations, and post-exploitation tradecraft.

- Every lab is documented with a pentester's mindset: **Why** the flaw exists, not just which command to run
- Exploit code is reviewed for safety and OPSEC indicators before execution
- Each vector ends with remediation steps, written the way they would appear in a real client report

---

## 🗺️ Repository Structure & Progress

| # | Phase / Focus | Status |
| :--: | :-- | :-- |
| 01 | Linux 101 & Core Internals (TCM Security + THM Pre-Security) | ✅ Done |
| 02 | OverTheWire: Bandit Wargame (Levels 0 → 33 Documented) | ✅ Done |
| 03 | TryHackMe: Linux Privilege Escalation | 🔄 In Progress |

---

## 🔥 Part Three — Linux Privilege Escalation Labs

**Active sprint:** 3 / 10 labs completed

| # | Vector | Key Concepts & Mechanics | Status | Write-up |
| :--: | :-- | :-- | :-- | :--: |
| 01 | Manual Enumeration | Host & privilege profiling (`id`, `uname -a`, `/etc/issue`, `ps`, `find`) | ✅ Done |[Read →](./03-privilege-escalation/01-manual-enumeration.md) |
| 02 | Automated Recon | Scripted enumeration (`LinPEAS`, `LinEnum`, `Linux Exploit Suggester`) | ✅ Done | [Read →](./03-privilege-escalation/02-automated-recon.md) |
| 03 | Kernel Exploits | `CVE-2015-1328` (`OverlayFS` namespace flaw, `gcc`, `ld.so.preload`) | ✅ Done | [Read →](./03-privilege-escalation/01-kernel-exploits-cve-2015-1328.md) |
| 04 | Sudo Abuse | Abusing `sudo -l`, `GTFOBins`, and `LD_PRELOAD` environment hijacking | 🔄 In Progress | — |
| 05 | SUID Binaries | `find -perm -4000`, abusing custom SUIDs, Shared Object Injection | ⬜ Up Next | — |
| 06 | Capabilities | `getcap`, exploiting `cap_setuid+ep` on system binaries | ⬜ Queued | — |
| 07 | Cron Jobs | Abusing writable scheduled scripts, `PATH` hijacking, and tar wildcards | ⬜ Queued | — |
| 08 | PATH Hijacking | Hijacking relative binary calls via writable `$PATH` directories | ⬜ Queued | — |
| 09 | NFS Root Squashing | Exploiting `no_root_squash` network shares to plant SUID shells | ⬜ Queued | — |
| 10 | Capstone Challenge | Full black-box privilege escalation methodology & final report | ⬜ Planned | — |

---

## ⚙️ Lab Documentation Standard

- **Enumeration First:** Documenting the exact commands used to identify the misconfiguration.
- **Root Cause Analysis:** Explaining the underlying Linux mechanism (`Real vs Effective UID`, namespaces, linker preloading).
- **OPSEC & Remediation:** Highlighting artifacts left on disk, crash risks, and how sysadmins patch the flaw.

---

## 🧭 Where This Fits

Networking ✅ → ⭐ **Linux (this repo)** → Python → Web Security → AI Red Teaming  
Stop #2. Foundations are done; the PrivEsc sprint runs alongside Python.

---

## 👤 Author

**Nada** — CS student · Aspiring Web App Pentester → AI Red Teamer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-1f2937?style=for-the-badge&logo=linkedin&logoColor=0A66C2&labelColor=111827)](https://www.linkedin.com/in/01sirius10)

---

<div align="center">

*Sirius is the brightest star in the night sky. Keep looking up.* ✨

</div>
