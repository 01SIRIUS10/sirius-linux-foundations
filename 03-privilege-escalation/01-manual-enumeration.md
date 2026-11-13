# 🔍 Manual Enumeration

| | |
| :-- | :-- |
| **Platform** | TryHackMe, Linux Privilege Escalation |
| **Category** | Linux Privilege Escalation |
| **Difficulty** | Easy |
| **Status** | ✅ Done |

---

## 🎯 Goal

I land on a machine with a weak account. I want to know exactly **what this machine is, who I am on it, what's running, and where the admin made a mistake**, before I touch anything.

> Low-privileged shell → full picture of the target.

---

## 🧠 Concept

### Terms I use in every lab

| Term | What it means to me |
| :-- | :-- |
| **Shell** | The black screen where I type commands and talk to the machine |
| **User** | An account on the machine, like your Twitter account |
| **root** | The big boss. Does whatever it wants on the machine. King of the game |
| **Kernel** | The heart of the OS. The middleman that talks to the hardware. One flaw in it and the whole machine falls |
| **Process** | A program that's running right now |
| **Permissions** | Who is allowed to read, write, or run a file |

**Privilege Escalation** = I get in with a weak account and I want to reach the king (root).

### Why enumerate first?

Imagine a doctor. A patient walks in. Does the doctor hand over medicine right away, before asking the name, the age, what hurts, or the medical history? Of course not.

Same in hacking. First I look around:
- What is this machine?
- Who am I on it?
- What's running?
- Where's the mistake?

### The script kiddie trap

A script kiddie finds a ready-made exploit and just throws it. If it fails, they don't know why. They memorize commands and run them.

> **Enumeration is not a step before the attack. It is MOST of the attack.**

In real work, a huge part of the time (people say around 80%) goes into collecting information. The exploit itself takes minutes.

**My rule:** before every command I ask myself *"What am I asking the machine, and what will I do with the answer?"* If I can't answer, I'm just copy-pasting.

So: collect facts first, understand every command so I can read the output myself, build the full picture, then rank what I found.

### My order (always the same)

1. System
2. Users
3. Processes
4. Network
5. Files
6. Cron
7. Software

---

## 🔍 Enumeration

> The outputs below are **example outputs** to show the format. They are not from a real target.

### 🖥️ 1. System: "What am I standing on?"

Old systems have known, public vulnerabilities.

```bash
hostname
uname -a
cat /etc/issue
cat /etc/os-release
cat /proc/version
```

| Command | What it tells me |
| :-- | :-- |
| `hostname` | The machine name. If it says `db-prod`, it's probably a database server, so there's important data |
| `uname -a` | **The most important one here.** Shows the kernel version and architecture, so I can search for exploits for that exact version |
| `cat /etc/issue` | Like a welcome message. Has the distro name and version, e.g. Ubuntu 14.04 |
| `cat /etc/os-release` | Same idea with more detail, and more reliable on modern systems |
| `cat /proc/version` | Kernel version plus some info about how it was built |

Example output:
```text
Linux web01 3.13.0-24-generic #46-Ubuntu SMP x86_64 GNU/Linux
```

**How I read it:** kernel version + distro version together, then I search for exploits for that exact combo. Version numbers are evidence.

> Note: `/proc/version` shows the compiler used to build the **kernel**, not necessarily what's installed on the target. To check if `gcc` exists, I use `which gcc` (Section 7).

---

### 👤 2. Users & Privileges: "Who am I and what can I do?"

#### `id`
Tells me who I am and what I can do.

```text
uid=1001(student) gid=1001(student) groups=1001(student),27(sudo),999(docker)
```

- **uid:** my ID number. root is always `0`.
- **groups:** the groups I'm in. **This is the most important part.**
- **sudo:** I can run commands as root.
- **docker:** I can create a container, mount the whole disk inside it, and be root in practice.

#### `whoami`
Just my name. I need it when I jump between users and forget who I am right now.

#### `sudo -l`
One of the most important commands. `sudo` means "run as root". The sysadmin lets some users run specific commands as root, and this command tells me **exactly what I'm allowed to run**.

```text
(root) NOPASSWD: /usr/bin/find
```

This means I can run `find` as root with no password. `find` can execute commands, so I can spawn a root shell with it. This is the **GTFOBins** idea: normal programs that become dangerous when they're given sudo rights (Lab 04).

#### `cat /etc/passwd`
The name is misleading. There are no passwords in it. It's the **list of users**.

```text
student:x:1001:1001:Student:/home/student:/bin/bash
```

| Part | Meaning |
| :-- | :-- |
| `student` | Username |
| `x` | Password (stored in another file) |
| `1001` | User ID (UID) |
| `1001` | Group ID (GID) |
| `Student` | Description |
| `/home/student` | Home folder |
| `/bin/bash` | The shell, so this account can log in |

If the shell is `/usr/sbin/nologin` or `/bin/false`, it's a service account and nobody logs in with it.

**My filter:**
```bash
cat /etc/passwd | grep -v nologin | grep -v false
```

- `|` (pipe): take the output of the command before me and give it to the command after me.
- `grep -v`: **exclude** lines that contain this word.
- Meaning: show me the users that have a real login shell.

#### `cat /etc/group`
The groups and their members. Shows me who else is in the `sudo` group, so who else might be a target.

#### `ls -la /etc/shadow`
This file has the **real password hashes**. I don't read it, I look at its **permissions**.

```text
-rw-r----- 1 root shadow 1234 Jan 1 12:00 /etc/shadow
```

- `rw-` owner (root)
- `r--` group (shadow)
- `---` everyone else: not allowed at all

If I see `-rw-r--r--`, anyone can read it. I can take the hashes and crack them on my own machine. That's a critical finding.

#### `history`
Shows commands typed before in this shell. People type things like `mysql -u root -pMyPassword123` and forget about it. **It only shows the current user's history.**

#### `env`
Environment variables. Sometimes they hold `DB_PASSWORD` or `API_KEY`. I also check `PATH` here (Lab 08).

---

### ⚙️ 3. Processes & Services: "What's running, and who runs it?"

If something is running as root and it has a weakness, that's my target.

```bash
ps aux
```

| Letter | Meaning |
| :-- | :-- |
| `a` | Processes of all users |
| `u` | Show the owner and resource usage |
| `x` | Even processes with no terminal (background services) |

(Writing `aux` without a dash is the old style. It works fine.)

`ps -ef` gives the same idea in a different format. It's useful because it shows the **PPID** (who the parent of the process is).

`ps aux | grep root` = show me only what root is running.

**What I look for:**
- Root services I might be able to influence.
- Scripts or programs that are **not part of the system**. Someone wrote them by hand, so they probably have mistakes.
- Commands with a password in the arguments, like `mysql -u root -pSecret`. It shows up in `ps` and anyone can see it.
- Old software versions.

---

### 🌐 4. Network: "Who does this machine talk to, and what's open from the inside?"

```bash
ip a
ip route
ss -tulnp
arp -a
cat /etc/hosts
```

| Command | What it tells me |
| :-- | :-- |
| `ip a` | Network cards and IPs. **If I see two cards**, the machine is connected to two networks, and through it I might reach another internal network. That's called **pivoting** |
| `ip route` | Which networks the machine can reach |
| `ss -tulnp` | The open ports |
| `arp -a` | Nearby devices this machine talked to (its neighbors) |
| `cat /etc/hosts` | Internal names. Sometimes it reveals other servers |

**`ss -tulnp` letter by letter:**
- `t` = TCP, `u` = UDP
- `l` = only what's **listening**
- `n` = numbers, don't resolve names (faster)
- `p` = show me the program that opened the port

**The golden point:**
```text
127.0.0.1:3306   ← open from inside the machine only
0.0.0.0:80       ← open to the whole world
```

A service on `127.0.0.1` won't show up in an nmap scan from outside. But I'm inside, so I can reach it. And people usually neglect the security of these services because they think nobody can reach them.

---

### 📁 5. Files & Permissions

This is where most vulnerabilities live. First I need to understand **permissions**.

#### Reading permissions

```text
-rwxr-xr-x
```

Split it like this:

```text
-    rwx     r-x     r-x
type owner   group   others
```

- `r` = read (4), `w` = write (2), `x` = execute (1)
- First character: `-` is a normal file, `d` is a folder

#### SUID: the most important idea here

Normally when I run a program, it runs with **my** privileges. But if the program has the **SUID bit**, it runs with the privileges of **its owner**, no matter who launched it.

**Why does it even exist?** Real example: the `passwd` command that changes your password. It has to edit `/etc/shadow`, which only root can touch. So `passwd` is SUID root: any user can run it, it runs as root, and edits the file **under locked-down conditions**.

It looks like this:

```text
-rwsr-xr-x 1 root root ... /usr/bin/passwd
   ↑
   an "s" instead of "x"
```

**The danger:** if I find a SUID root program that can run commands or read files (like `find`, `vim`, or `bash`), then I'm running commands as root.

**The command in detail:**
```bash
find / -perm -4000 -type f 2>/dev/null
```

| Part | Meaning |
| :-- | :-- |
| `find /` | Search from the very top of the filesystem |
| `-perm -4000` | Files that have **SUID set**. The `-` before the number means "at least these bits are set" |
| `-type f` | Files only, not folders |
| `2>/dev/null` | Get rid of error messages |

**Why 4000?** `4` = SUID, `2` = SGID, `1` = sticky bit. So `4000` is SUID, and `2000` is SGID (same idea but with the **group's** privileges).

Example output:
```text
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/find        ← this one is unusual, look closer
```

#### The `2>/dev/null` trick

Linux has 3 channels:
- `0` = input
- `1` = normal output
- `2` = **error messages**

`2>` means "send channel 2 somewhere else". `/dev/null` is a **trash can**: anything thrown in it disappears. When I search the whole machine I get hundreds of `Permission denied` lines. `2>/dev/null` keeps the output clean.

#### Capabilities

Before, there were two kinds of programs: normal or root. **Capabilities split root's power into small pieces.** A program can get only the power it needs, not everything.

```bash
getcap -r / 2>/dev/null
```

(`-r` = recursive, so it checks all folders.)

If I find `python3 = cap_setuid+ep`, that python can **change its own UID**. So I run it and make it call `setuid(0)` (0 = root). That's basically being root.

#### Writable locations

```bash
find / -writable -type d 2>/dev/null
find / -perm -222 -type d 2>/dev/null
find / -writable -type f 2>/dev/null | grep -v proc
```

Find the folders and files **I can write to**.

**Why it matters:** if root runs something from a place I can write to, I can put my own thing there, and root runs it without noticing. This is the base of **Cron** and **PATH Hijacking**.

- `-perm -222` = folders that **anyone** can write to (world-writable).
- `grep -v proc` removes `/proc`, because it fills the screen with useless stuff.

#### Sensitive files

```bash
ls -la ~ /home /root 2>/dev/null
find / -name "id_rsa*" 2>/dev/null
find / -name "*.bak" -o -name "*.old" -o -name "*.conf" 2>/dev/null
grep -ri "password" /etc /var/www /opt 2>/dev/null
```

- `ls -la ~ /home /root`: look inside user folders. If I can read `/root`, game over.
- `find / -name "id_rsa*"`: private SSH keys. If I find someone's key, I can log in as them with no password.
- `*.bak`, `*.old`, `*.conf`: backups and config files. Sometimes they hold passwords.
- `grep -ri "password" ...`: search for the word "password" inside files. `r` = recursive, `i` = ignore upper/lower case.

**Why it matters:** people **reuse passwords**. The database password might be the same as the root password.

#### NFS

```bash
cat /etc/exports
```

`/etc/exports` is the file that says which folders the machine shares over the network.

```text
/tmp *(rw,no_root_squash)
```

Normally, if root from another machine connects to the share, the system **downgrades** it to `nobody`. That's **root squashing**. But `no_root_squash` means "leave them as root for real". So from my own machine (where I'm root) I create a SUID root file inside the share, then go back to the target, run it, and I'm root.

---

### ⏰ 6. Scheduled Tasks (Cron)

**Cron** is the machine's alarm clock. It runs commands at set times (every minute, every hour...). Most of them run as root.

```bash
cat /etc/crontab
ls -la /etc/cron*
crontab -l
```

**The three weaknesses I look for:**

1. **A root script that I can edit.** I put my command in it, and next minute it runs as root.
2. **A relative path** like `backup.sh` instead of `/usr/local/bin/backup.sh`. The system searches for it in `PATH`, and I can put a file with the same name earlier in the path.
3. **A wildcard** like `tar czf backup.tgz *`. If I can create files in that folder, I can name a file `--checkpoint=1`. `tar` reads it as an **option**, not a filename. (Full details in Lab 07.)

---

### 📦 7. Software: "What can I use, and what's outdated?"

```bash
dpkg -l            # Debian/Ubuntu
rpm -qa            # RedHat/CentOS
which gcc python3 python perl wget curl nc
```

Two things I learn: which vulnerable versions are installed, and which tools I already have on the target. For example, if `gcc` exists I can compile an exploit locally (Lab 03).

---

## 💥 Exploitation

None in this lab. Enumeration only reads what the system already shows me. But every finding points to a next step:

| What I find | What it suggests | Lab |
| :-- | :-- | :--: |
| Old kernel or OS version | Kernel exploit | [03](./03-kernel-exploits-cve-2015-1328.md) |
| Entries in `sudo -l` | Sudo abuse (GTFOBins, `LD_PRELOAD`) | 04 |
| Unusual SUID binaries | SUID abuse | 05 |
| `cap_setuid` or similar | Capabilities abuse | 06 |
| Root cron job calling a writable script | Cron abuse | 07 |
| Writable `PATH` folder, relative binary calls | PATH hijacking | 08 |
| `no_root_squash` in `/etc/exports` | NFS root squashing | 09 |

---

## 🧬 Root Cause

Nothing here is a bug. These commands only read what Linux already shows to normal users. The real problem is that admins leave too much exposed: old kernels, loose permissions, passwords in history, and root running things anyone can edit.

---

## 🛡️ Remediation

| Issue | Fix |
| :-- | :-- |
| Outdated kernel and packages | Patch regularly, automate updates |
| Excessive `sudo` rights | Least privilege, avoid `NOPASSWD` |
| Unnecessary SUID binaries | Audit with `find -perm -4000`, remove the bit where it's not needed |
| Passwords in history, env, or configs | Use a secrets manager, restrict file permissions |
| Readable sensitive files | `chmod 600` for keys, `640` for shadow |
| Services running as root | Run them as dedicated low-privilege users |
| Writable cron scripts | Make them root-owned, not writable by others |

---

## 📝 My Lab Notes

**What clicked for me:**
Enumeration is not the warm-up, it's the main job. The exploit is the short part at the end. The doctor example made it simple: nobody gives medicine before asking questions.

**The trap I'm avoiding:**
Being a script kiddie: grabbing a ready exploit, running it, and having no idea why it worked or why it failed. If I can't explain a command, I'm just copy-pasting.

**My routine from now on:**
Same order every time: System → Users → Processes → Network → Files → Cron → Software. Before each command I ask what I'm asking the machine and what I'll do with the answer. Then I rank my findings and pick the vector they point to.

---

## 💡 Key Takeaways

1. Enumeration is most of the attack. The exploit is the short part.
2. Before every command: what am I asking the machine, and what will I do with the answer?
3. Every finding should point to a vector. A finding with no next step is just noise.
4. Manual first to understand. Tools (Lab 02) for coverage.

---

⬅️ [Back to Labs](../README.md) &nbsp;|&nbsp; ➡️ [Next: Automated Recon](./02-automated-recon.md)
