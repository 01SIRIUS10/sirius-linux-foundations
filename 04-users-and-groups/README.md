# 👥 04 - Users and Groups

## Status: 🟢 Completed

---

> ### 🧩 The Question That Starts Everything
> Have you ever asked yourself why, when you open an online store or any website, you never see how many admins the site has, or the users' passwords sitting in plain sight?
>
> How does a system with thousands of files and millions of potential users decide **exactly who gets to do what**? Without a permission system, any random user could delete anything, or read anyone's password.

In our track, this single topic is the exact difference between having a **weak shell** and having **full root** on a server. Almost every privilege escalation technique traces back to a misunderstanding of something we're about to break down right now.

---

## 🏗️ Foundation: Why Multi-User Even Exists

Linux was designed from the ground up to be **multi-user** — a single server can be accessed by 50 employees simultaneously, each with their own files and their own visibility scope.

---

## 📇 Part 1: Reading `/etc/passwd` — Like an Investigator, Not a Student

We've mentioned `/etc/passwd` a hundred times already. This time, we're reading it like a detective. Consider this:

~~~text
root:x:0:0:root:/root:/bin/bash
SIRIUS:x:1000:1000:SIRIUS:/home/bob:/bin/bash
SIR:x:1:1:SIR:/usr/sbin:/usr/sbin/nologin
~~~

### 🚩 Field #3: The UID

If you ever spot a **non-root** username with **UID = 0**, that's a massive red flag. Why? Because a UID of zero means **root-level privileges**, regardless of what name is attached to it.

> 🧨 **Exploit Spotlight: UID 0 Backdoor**
> If an attacker manages to inject a new line into `/etc/passwd` with `UID=0`, they've effectively created a second root account under any name they like. This is a textbook classic attack.

### 🐚 Field #7: The Shell Column

You need to know that `/usr/sbin/nologin` blocks interactive login entirely.

~~~bash
cat /etc/passwd | grep -v "nologin\|false"
~~~

This filters the list down to only users with a real, usable shell — the actual human accounts who might have passwords worth brute-forcing, or credentials that leaked somewhere else.

> 💡 **Bottom line:** the system doesn't understand names — it understands numbers (UID and GID). root is always 0.

---

## 💰 Part 2: `/etc/shadow` — The Real Treasure
This is one of the most security-critical files in all of Linux, and here's why.

Originally, password hashes were stored directly inside `/etc/passwd` — a file that's readable by literally everyone (world-readable), because many system tools need to read basic info from it (like `ls -l` translating a UID into a username).

The problem: this meant any regular user could read the hashes and attempt to crack them offline.

The fix: the hashes were moved to a separate file — `/etc/shadow` — and locked down so only root can read it.

~~~bash
ls -l /etc/shadow
~~~

Expected output:

~~~text
-rw-r----- 1 root shadow
~~~

Notice the group with read access is called shadow, not root. Any user who's a member of the shadow group can read this file without ever needing to be root.

> ⚠️ **Common Misconfiguration Alert:** If you ever find your user accidentally added to the shadow group, that door just opened wide for you.

---

## 🔢 Part 3: Permission Bits — The Foundation Everything Else Builds On

~~~text
-rwxr-xr--
~~~

| Position | Meaning |
|---|---|
| 1st character | Entry type (`-` = file, `d` = directory, `l` = symbolic link) |
| Characters 2-4 | Owner permissions |
| Characters 5-7 | Group permissions |
| Characters 8-10 | Others / World permissions |

### 🤔 Why Specifically 4, 2, and 1?
Because computers think in binary. With 3 available slots per group:

| Permission | Binary Power | Value |
|---|---|---|
| Read | 2² | 4 |
| Write | 2¹ | 2 |
| Execute | 2⁰ | 1 |

---

## 🔧 Part 4: chmod — Symbolic vs. Octal, and When to Actually Use Each
Here's the practical rule professionals actually follow:

~~~bash
chmod u+x script.sh       # symbolic: add execute permission for the owner only
chmod g-w file.txt        # symbolic: remove write permission from the group
chmod 755 script.sh       # octal: rwx for owner, r-x for group and others
chmod 644 file.txt        # octal: rw for owner, r-only for group and others
~~~

The command you'll use 90% of the time during pentesting:

~~~bash
chmod +x exploit.sh       # the fastest way to make any exploit script executable
~~~

### 📊 The Number Table You Need Memorized
Full permissions `rwx` = 7. Why? Because Read (4) + Write (2) + Execute (1) = 7.

| Number | Meaning |
|---|---|
| 7 | `rwx` — Read, Write, Execute |
| 6 | `rw-` — Read and Write, no Execute |
| 5 | `r-x` — Read and Execute (4 + 1) |
| 0 | `---` — Nothing at all |

> 🚨 **Critical Security Note:** `chmod 777` on anything is a massive red flag in any security audit. It means literally anyone on the system can read, write, and execute it. If you find a 777 file during a pentest engagement, that's the very first thing you investigate as a potential attack vector.

---

## 🎭 Part 5: SUID, SGID, and the Sticky Bit

### 🔑 SUID — The Passwd Paradox
Think about this question: the `passwd` command changes your password, and that password is stored in `/etc/shadow`, which nobody except root can write to. So how does a regular user manage to run `passwd` successfully?

The answer: the **SUID bit**.

When a binary has SUID set, it executes with the privileges of the file's owner (root) instead of the privileges of whoever ran it. So when SIRIUS runs `/usr/bin/passwd`, they temporarily operate "as if they were root" — but only for the duration of that command.

~~~bash
ls -l /usr/bin/passwd
~~~

Expected output:

~~~text
-rwsr-xr-x 1 root root 68208 /usr/bin/passwd
~~~

Notice the letter `s` replacing `x` in the owner's slot — that's the SUID bit, visually confirmed.

### 🎭 SGID (Set Group ID) — Same Idea, For Groups
Important use case: if you set SGID on a directory, any new file created inside it inherits the directory's group, not the group of the user who created it. This is common in team-collaboration environments. From a security angle, SGID on a binary works just like SUID, but grants the group's privileges instead of the owner's.

### 🧷 Sticky Bit — Why `/tmp` Stays (Relatively) Safe Despite Being World-Writable

~~~text
drwxrwxrwt
~~~

Notice the letter `t` at the end instead of `x`.

Why does this exist? If `/tmp` is writable by everyone (and it genuinely is), without the sticky bit, any user could delete or modify another user's files inside `/tmp` — even files they don't own. The sticky bit enforces this rule: "Anyone can write here, but only the file's owner (or root) can delete or modify their own file."

---

## 🏷️ Part 6: Ownership — chown and chgrp

~~~bash
chown sally file.txt           # change the owner
chown sally:sally file.txt     # change both owner and group at once
chgrp sally file.txt           # change only the group
~~~

> 🚨 **Security Note:** `chown` and `chgrp` normally require root privileges. If you ever find yourself able to change a file's ownership without root, that either means you're already the owner, or there's a sudoers misconfiguration granting you this power — and that's a dangerous indicator worth investigating further.

---

## 🛡️ Part 7: sudo — The Most Important Privilege Escalation Tool in Linux History

### 🕰️ Why Does `sudo` Even Exist? (A Look at the Roots)
The old philosophy was: log in directly as root and just work from there. The problem: one single mistake, with nothing to stop you, would execute with the highest privilege possible.

The fix: operate as a regular user at all times, and elevate your privileges only temporarily, exactly when needed — and every single use of `sudo` gets logged.

~~~bash
tail -f /var/log/auth.log
~~~

Every time someone uses `sudo`, a line gets recorded here. This is the foundation of virtually every Blue Team detection mechanism for privilege escalation attempts.

---

## 🔑 Part 8: The passwd Command — Just Two Lines
The command is simple:

* `passwd` → change your own password.
* `sudo passwd username` → change someone else's password.

The one thing that actually matters: if you're root and you change another user's password, there's no validation on how strong the new password needs to be. This is a common technique used in CTFs — showing you how to reset a user's password after reaching root, purely to maintain persistence.

### 🕵️ Enumeration with `sudo -l`
The very first command you should run in any engagement, right after landing any shell:

~~~bash
sudo -l
~~~

This reveals exactly which commands you're allowed to run as root without needing root's actual password (you just need your own password — and sometimes not even that, if `NOPASSWD` is configured).

---

## 🎬 Real-World Scenario (HTB/OSCP Classic)
* A UID of zero is root, regardless of the username — any other account carrying it is a backdoor.
* The execute bit on a directory isn't like it is on a file — it's the actual key to entry, not just permission to see names.
* The SUID bit is the single most dangerous thing to hunt for on any machine:

~~~bash
find / -perm -4000
~~~

because it grants the privileges of the original owner instead of the person who ran the command.
* `sudo -l` is the first question you ask any system: "What am I allowed to do as root, without actually being root?" — and the answer is often your direct path to full compromise via GTFOBins.

---

## 🧾 Section Summary

| Concept | Key Takeaway |
|---|---|
| `/etc/passwd` | The list of users — essential reading to know who exists on the server. |
| `/etc/shadow` | The list of encrypted passwords (hashes) — you need root to read it, and once you do, you crack it with Hashcat. |
| `id` command | The most important command for discovering your own privileges and numeric identity (UID/GID). |

### 🔢 The Octal Magic
* Read = 4
* Write = 2
* Execute = 1
* Order: (Owner) (Group) (World)

### 🚨 Critical Warnings
* `chmod 777` → Security malpractice. Everyone can read, write, and execute. A fatal vulnerability if applied to a sensitive file.
* `sudo -l` → The magic key to discovering exactly what you can do with elevated privileges.

---
*This content was studied and written in my own words based on the concepts covered in TCM Security's Linux 101 course. Any errors in explanation or terminology are entirely my own.*
