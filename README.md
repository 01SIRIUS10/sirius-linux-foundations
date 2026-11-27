# Linux Privilege Escalation Master Runbook & Lab Portfolio

A systematic, production-grade security engineering repository documenting Linux Privilege Escalation attack vectors, internal OS security boundaries, architectural root causes, and hardening remediation.

---

## 🎯 Sprint Progress: 9 / 10 Labs Completed

| Lab # | Lab Writeup Document | Attack Vector / Focus Area | Target Binary / Artifact | Difficulty | Status |
|:---:|---|---|---|:---:|:---:|
| **01** | [01-manual-enumeration.md](01-manual-enumeration.md) | Manual Linux OS Reconnaissance | `procfs`, `/etc/passwd`, Network Sockets | Easy | Completed |
| **02** | [02-automated-recon.md](02-automated-recon.md) | Automated Scripts & Tooling | `LinPeas`, `LinEnum`, `lse.sh` | Easy | Completed |
| **03** | [03-kernel-exploits-cve-2015-1328.md](03-kernel-exploits-cve-2015-1328.md) | Kernel Vulnerability Exploitation | `overlayfs` (CVE-2015-1328) | Medium | Completed |
| **04** | [04-sudo-abuse.md](04-sudo-abuse.md) | Sudo Rights & Token Abuse | `sudo find -exec`, Password Cracking | Easy | Completed |
| **05** | [05-suid-binaries.md](05-suid-binaries.md) | SUID / SGID Bit Exploitation | `/usr/bin/base64`, `/etc/shadow` Read | Easy | Completed |
| **06** | [06-capabilities.md](06-capabilities.md) | Linux POSIX Capabilities | `/home/karen/vim` (`cap_setuid+ep`) | Easy | Completed |
| **07** | [07-cron-jobs.md](07-cron-jobs.md) | Cron Job Insecure File Overwrite | `/home/karen/backup.sh`, SUID Drop | Easy | Completed |
| **08** | [08-path-variable.md](08-path-variable.md) | PATH Hijacking & Unquoted Binaries | `system("thm")`, `$PATH` Prepending | Easy | Completed |
| **09** | [09-nfs-root-squash.md](09-nfs-root-squash.md) | Insecure NFS `no_root_squash` | Remote Mount, glibc SUID Injection | Medium | Completed |
| **10** | [10-capstone-challenge.md](10-capstone-challenge.md) | Comprehensive Privilege Escalation | Full Triage Runbook (Phases 0–5) | Hard | Next Up |

---

## 🛠️ The 6-Phase Master Triage Runbook

Every lab in this series adheres to the systematic triage methodology:

```text
+---------------------------------------------------------------------------+
|                  SYSTEMATIC PRIVILEGE ESCALATION RUNBOOK                  |
+---------------------------------------------------------------------------+
| Phase 0: TTY Stabilization & Environmental Hygiene                        |
|          python3 -c 'import pty;pty.spawn("/bin/bash")'                   |
|                                                                           |
| Phase 1: Identity & Execution Context                                     |
|          id, groups, whoami, uname -a                                     |
|                                                                           |
| Phase 2: Administrative Delegation (Sudo)                                 |
|          sudo -l -l                                                       |
|                                                                           |
| Phase 3: File Perms, SUID/SGID & POSIX Capabilities                       |
|          find / -perm -u=s -type f 2>/dev/null                            |
|          getcap -r / 2>/dev/null                                          |
|                                                                           |
| Phase 4: Scheduled Tasks & Network Storage Exports                        |
|          cat /etc/crontab /etc/cron.*/, cat /etc/exports                  |
|                                                                           |
| Phase 5: Inter-Process Communication, Sockets & Credentials               |
|          ss -tuln, ps aux, history, unshadow + john                       |
+---------------------------------------------------------------------------+
```

---

## 📂 Repository Structure

```text
sirius-linux-foundations/
└── 03-privilege-escalation/
    ├── README.md
    ├── 01-manual-enumeration.md
    ├── 02-automated-recon.md
    ├── 03-kernel-exploits-cve-2015-1328.md
    ├── 04-sudo-abuse.md
    ├── 05-suid-binaries.md
    ├── 06-capabilities.md
    ├── 07-cron-jobs.md
    ├── 08-path-variable.md
    ├── 09-nfs-root-squash.md
    └── 10-capstone-challenge.md
```

---

## 🛡️ Engineering & Hardening Principles

1. **Principle of Least Privilege (PoLP):** Restrict sudo commands to exact arguments; never grant wildcards on interpreters or utilities supporting shell breakouts (`find`, `vim`, `less`).
2. **Granular POSIX Capabilities:** Replace coarse-grained SUID binaries with fine-grained capabilities, auditing for elevated capabilities like `cap_setuid` that allow full credential synthesis.
3. **Defensive Pathing:** Always invoke executables inside scripts and binaries using absolute, immutable paths (`/usr/bin/id` instead of `id`).
4. **Secure Export Policies:** Never export NFS filesystems with `no_root_squash` to untrusted networks; enforce `nosuid` mount flags on all external media.
5. **Deterministic Scheduling:** Lock down cron job scripts with strict ownership (`root:root`) and immutable permissions (`0700` or `0755`), preventing low-privilege script modification.
