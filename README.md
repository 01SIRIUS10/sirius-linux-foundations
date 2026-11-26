<img src="https://raw.githubusercontent.com/01SIRIUS10/01SIRIUS10/main/Banner.png" width="100%" alt="SIRIUS Banner" />

# Linux Privilege Escalation — Production Lab Notes

A methodical repository documenting enterprise-grade Linux privilege escalation attack vectors, internal mechanics, remediation blueprints, and root-cause analyses across standardized evaluation targets.

> **Current Sprint Progress:** `7 / 10 labs completed`

---

## 🧭 Privilege Escalation Vector Matrix

| # | Attack Vector | Severity | Core Mechanics & Primitives | Status | Lab Documentation |
| :-: | :--- | :-: | :--- | :-: | :--- |
| `01` | **Manual Enumeration** | `Info` | Environment triage, user mapping, network sockets, unquoted path analysis | `Completed` | [01-manual-enumeration.md](01-manual-enumeration.md) |
| `02` | **Automated Reconnaissance** | `Info` | LinPEAS, LinEnum, session artifact inspection, false-positive elimination | `Completed` | [02-automated-recon.md](02-automated-recon.md) |
| `03` | **Kernel Exploits** | `Critical` | CVE-2015-1328 (OverlayFS), dirty copy-on-write, kernel memory corruptions | `Completed` | [03-kernel-exploits-cve-2015-1328.md](03-kernel-exploits-cve-2015-1328.md) |
| `04` | **Sudo Rights Abuse** | `High` | `NOPASSWD` delegation, GTFOBins, shell escapes, security boundary collapse | `Completed` | [04-sudo-abuse.md](04-sudo-abuse.md) |
| `05` | **SUID / SGID Binaries** | `High` | SUID bit mechanics, arbitrary file read primitives, offline cryptanalysis | `Completed` | [05-suid-binaries.md](05-suid-binaries.md) |
| `06` | **POSIX Capabilities** | `High` | Capability splitting, `cap_setuid+ep`, embedded language execution engines | `Completed` | [06-capabilities.md](06-capabilities.md) |
| `07` | **Cron Jobs** | `Medium` | Insecure DAC permissions, SUID root drops, relative path injection, John | `Completed` | [07-cron-jobs.md](07-cron-jobs.md) |
| `08` | **PATH Variable Abuse** | `Medium` | Binary hijacking, search order manipulation, custom executable compilation | `Pending` | `08-path-variable.md` |
| `09` | **NFS Root Squashing** | `High` | `no_root_squash` exports, remote SUID payload injection, NFS mounting | `Pending` | `09-nfs-root-squash.md` |
| `10` | **PrivEsc Capstone Challenge** | `High` | Multi-vector exploitation, defense-in-depth compromise, final flag harvesting | `Pending` | `10-capstone-challenge.md` |

---

## 🛠️ Methodological Triage Protocol

1. **Phase 0: Session Stabilization:** Ensure full TTY interactive control (`python3 -c 'import pty; pty.spawn("/bin/bash")'`).
2. **Phase 1: Identity & Context:** Interrogate effective UID, group memberships (`id`), kernel parameters (`uname -a`), and container indicators (`/proc/1/cgroup`).
3. **Phase 2: Administrative Delegations:** Audit sudo permissions via `sudo -l`.
4. **Phase 3: Privilege Anomalies:** Check filesystem SUIDs (`find / -perm -04000`) and POSIX capabilities (`getcap -r / 2>/dev/null`).
5. **Phase 4: Scheduled Operations:** Parse system-wide crontabs (`/etc/crontab`, `/var/spool/cron/crontabs`).
6. **Phase 5: Process & Internal Bindings:** Map listening local ports (`ss -tulnp`) and executing daemon identities (`ps aux`).

---

<p align="center">
  <b>Nada (01SIRIUS10)</b> — CS Student · Aspiring Web App Pentester → AI Red Teamer<br>
  <i>"It sounds serious, until you look up."</i><br>
  <a href="https://www.linkedin.com/in/01sirius10">LinkedIn</a> • <a href="https://github.com/01SIRIUS10">GitHub</a>
</p>
