<p align="center">
  <img src="https://raw.githubusercontent.com/01SIRIUS10/01SIRIUS10/main/Banner.png" width="100%" alt="SIRIUS Banner" />
</p>

<p align="center">
  <img src="https://readme-typing-svg.herokuapp.com?font=Fira+Code&size=18&pause=1000&color=58A6FF&center=true&vCenter=true&width=650&lines=SIRIUS+Linux+Foundations+%E2%86%92+Offensive+Security;From+POSIX+Internals+to+Kernel+Exploitation;10%2F10+PrivEsc+Sprint+%C2%B7+Production+Engineering+Notes" alt="Typing SVG" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/TARGET_OS-LINUX_%2F_UNIX-1f2937?style=for-the-badge&logo=linux&logoColor=F8FAFC&labelColor=111827" alt="Linux" />
  <img src="https://img.shields.io/badge/TRACK-OFFENSIVE_RED_TEAM-1f2937?style=for-the-badge&logo=target&logoColor=EF4444&labelColor=111827" alt="Red Team" />
  <img src="https://img.shields.io/badge/WARGAME-BANDIT_34%2F34-1f2937?style=for-the-badge&logo=gnubash&logoColor=10B981&labelColor=111827" alt="Bandit" />
  <img src="https://img.shields.io/badge/PRIVESC_SPRINT-7%20%2F%2010%20DONE-1f2937?style=for-the-badge&logo=tryhackme&logoColor=38BDF8&labelColor=111827" alt="Sprint" />
</p>

---

## 🏛️ Repository Architecture

This repository forms the tactical foundation of the **SIRIUS Offensive Security Path**, detailing the complete progression from foundational operating system internals and shell automation up to enterprise-grade Linux privilege escalation and kernel compromise.

```text
sirius-linux-foundations/
├── 01-introduction/ ... 11-scripting/  ──> [Pillar 1: System Internals & Administration]
├── bandit-wargame/ (Levels 0 → 33)     ──> [Pillar 2: Shell Mastery & Wargame Warfare]
└── 03-privilege-escalation/            ──> [Pillar 3: Active Sprint — Enterprise PrivEsc]
```

---

## 💥 Active Sprint: Linux Privilege Escalation (Task-by-Task)

A methodical breakdown of internal mechanics, security boundary collapses, remediation blueprints, and root-cause analyses across production target evaluations.

> **Current Sprint Execution:** `7 / 10 labs completed`

```text
Progress: [■■■■■■■□□□] 70% Completed
```

### 🧭 Vector Matrix & Engineering Writeups

| # | Attack Vector | Severity | Core Primitives & Mechanics | Status | Tactical Writeup |
| :-: | :--- | :-: | :--- | :-: | :---: |
| `01` | **Manual Enumeration** | ![Info](https://img.shields.io/badge/INFO-3B82F6?style=flat-square) | Environment triage, user mapping, network sockets, unquoted path analysis | ![Done](https://img.shields.io/badge/COMPLETED-10B981?style=flat-square) | [Read Analysis ↗](03-privilege-escalation/01-manual-enumeration.md) |
| `02` | **Automated Recon** | ![Info](https://img.shields.io/badge/INFO-3B82F6?style=flat-square) | LinPEAS, LinEnum, session artifact inspection, false-positive triage | ![Done](https://img.shields.io/badge/COMPLETED-10B981?style=flat-square) | [Read Analysis ↗](03-privilege-escalation/02-automated-recon.md) |
| `03` | **Kernel Exploits** | ![Critical](https://img.shields.io/badge/CRITICAL-E11D48?style=flat-square) | CVE-2015-1328 (OverlayFS), copy-on-write race, kernel memory corruption | ![Done](https://img.shields.io/badge/COMPLETED-10B981?style=flat-square) | [Read Analysis ↗](03-privilege-escalation/03-kernel-exploits-cve-2015-1328.md) |
| `04` | **Sudo Rights Abuse** | ![High](https://img.shields.io/badge/HIGH-EA580C?style=flat-square) | `NOPASSWD` delegation, GTFOBins, shell escapes, boundary collapse | ![Done](https://img.shields.io/badge/COMPLETED-10B981?style=flat-square) | [Read Analysis ↗](03-privilege-escalation/04-sudo-abuse.md) |
| `05` | **SUID / SGID Binaries** | ![High](https://img.shields.io/badge/HIGH-EA580C?style=flat-square) | SUID bit mechanics, arbitrary file read primitives, offline John cracking | ![Done](https://img.shields.io/badge/COMPLETED-10B981?style=flat-square) | [Read Analysis ↗](03-privilege-escalation/05-suid-binaries.md) |
| `06` | **POSIX Capabilities** | ![High](https://img.shields.io/badge/HIGH-EA580C?style=flat-square) | Kernel capability splitting, `cap_setuid+ep`, embedded language escapes | ![Done](https://img.shields.io/badge/COMPLETED-10B981?style=flat-square) | [Read Analysis ↗](03-privilege-escalation/06-capabilities.md) |
| `07` | **Cron Jobs** | ![Medium](https://img.shields.io/badge/MEDIUM-EAB308?style=flat-square) | Insecure DAC permissions, SUID root drops, relative path injection, John | ![Done](https://img.shields.io/badge/COMPLETED-10B981?style=flat-square) | [Read Analysis ↗](03-privilege-escalation/07-cron-jobs.md) |
| `08` | **PATH Variable Abuse** | ![Medium](https://img.shields.io/badge/MEDIUM-EAB308?style=flat-square) | Binary hijacking, search order manipulation, custom executable compilation | ![Pending](https://img.shields.io/badge/IN_QUEUE-6B7280?style=flat-square) | `08-path-variable.md` |
| `09` | **NFS Root Squashing** | ![High](https://img.shields.io/badge/HIGH-EA580C?style=flat-square) | `no_root_squash` exports, remote SUID payload injection, NFS mounting | ![Pending](https://img.shields.io/badge/IN_QUEUE-6B7280?style=flat-square) | `09-nfs-root-squash.md` |
| `10` | **Capstone Challenge** | ![High](https://img.shields.io/badge/HIGH-EA580C?style=flat-square) | Multi-vector exploitation, defense-in-depth compromise, flag harvesting | ![Pending](https://img.shields.io/badge/IN_QUEUE-6B7280?style=flat-square) | `10-capstone-challenge.md` |

---

## 🛠️ Methodological Triage Runbook

Every target in this repository is assessed through a repeatable 6-phase triage standard:

1. **Phase 0: TTY Stabilization:** Ensure full terminal session control (`python3 -c 'import pty; pty.spawn("/bin/bash")'`).
2. **Phase 1: Identity & Context:** Interrogate effective UID, group memberships (`id`), kernel architecture (`uname -a`), and virtualization boundaries (`/proc/1/cgroup`).
3. **Phase 2: Administrative Delegations:** Audit sudo execution vectors without passwords (`sudo -l`).
4. **Phase 3: Privilege Anomalies:** Audit filesystem SUID/SGID bits (`find / -perm -04000 2>/dev/null`) and extended POSIX capabilities (`getcap -r / 2>/dev/null`).
5. **Phase 4: Scheduled Automation:** Parse system-wide tables (`/etc/crontab`, `/var/spool/cron/crontabs`) for writable scripts or unanchored relative paths.
6. **Phase 5: Process & Internal Bindings:** Map listening local sockets (`ss -tulnp`) and daemon context ownership (`ps aux`).

---

<p align="center">
  <b>Nada (01SIRIUS10)</b> — CS Student · Aspiring Web App Pentester → AI Red Teamer<br>
  <i>"It sounds serious, until you look up."</i><br>
  <a href="https://www.linkedin.com/in/01sirius10">LinkedIn</a> • <a href="https://github.com/01SIRIUS10">GitHub</a>
</p>
