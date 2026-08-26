 # 🗂️ 03 - Files and The Filesystem

## Status: 🟢 Completed

---

> **TL;DR:** The server you're attacking is nothing more than a giant collection of files and folders. If you don't understand this map, you can't steal data, you can't hide from the Blue Team, and you can't escalate your privileges. This is your GPS.

---

## 🧭 Why This Module Matters More Than You Think
After **Getting Started**, we now step into **Files** and **The Filesystem**. Here's the core truth every hacker needs to internalize:

At the end of the day, the server you're compromising is just files and directories. If you don't understand this layout, you won't be able to steal data, hide from your second-greatest enemy after the devil himself — **the Blue Team** — or escalate your privileges. This is literally your **GPS**.

In both bug bounty hunting and pentesting, the moment you land any access — even a weak one — you're now racing against the clock. What's the most important thing to find and extract fast, before someone detects you? Every single command in this module directly serves that race.

This lesson is broken down into **four core pillars** that matter most to us as hackers:

---

## 🏛️ Part 1: The Filesystem Map — Where the Treasure Lives

| Path | What Lives Here |
|---|---|
| **`/etc`** | Configuration files — a goldmine for recon. Contains `/etc/passwd` (all users) and sometimes configs with forgotten credentials. |
| **`/tmp`** | Temporary files, usually **world-writable** — any user can write here. Heavily used in privilege escalation and cron job hijacking. |
| **`/var/log`** | Where all the logs live — this is the Blue Team's territory. |
| **`/proc`** | Live information about every running process — can leak sensitive data. |
| **`/root`** | If you make it here, congratulations — you're root. |
| **`/dev`** | Hardware-related info — rarely touched except in advanced exploitation. |

### 🤔 Why should you care about this hierarchy at all?
Linux starts from a single root: `/`. Everything lives underneath it — there's no `C:` or `D:` like in Windows. Whether it's a hard drive or a USB flash drive, it all gets mounted under `/mnt` or `/media` using the `mount` command.

### 🏢 The Apartment Building Analogy
Think of the entire filesystem as an apartment building:
- **`/`** → The building's main entrance door.
- **`/home`** → The residents' apartments.
- **`/root`** → The building owner's penthouse.
- **`/tmp`** → The street or the lobby — anyone can walk in and dump anything.
- **`/etc`** → The bank vault of secrets. Inside it, `/etc/passwd` holds usernames, and `/etc/shadow` holds the encrypted passwords. This is the very first place we run to.

### 🔓 The Writable Zones: `/tmp` and `/dev/shm`
Both of these directories allow regular users to write freely. As a pentester, once you compromise a server with a low-privilege user, you upload your exploits to `/tmp` — because it's the only place the server will actually let you write to.

> ⚠️ **Warning:** Anything inside `/tmp` gets wiped clean if the server restarts.

### 📹 `/var/log` — The Security Cameras
This directory is, quite literally, the equivalent of surveillance cameras. Everything you do gets logged here — which means you need to get comfortable with covering your tracks to slip past the Blue Team.

### 📚 Want the Full Map?
~~~bash
man hier
~~~

## 🛣️ Part 2: Absolute vs. Relative Paths — The Foundation of Path Traversal
Why does this matter so much, especially in web application security?

Picture any website that accepts a filename from you to display it, like this:

~~~text
example.com/view?file=report.pdf
~~~

If the server treats this filename as a relative path without proper validation, you could try:

~~~text
example.com/view?file=../../../../etc/passwd
~~~

What does this actually mean?
Using `../` means "go back to the parent directory." By chaining it repeatedly, you climb out of the current directory entirely and reach sensitive system files. This is a famous, well-documented vulnerability:

### 🧨 Exploit Spotlight: Path Traversal (CWE-22)
One of the most historically significant entries in the OWASP Top 10, and directly tied to your goals in web app pentesting.

**The Attack in Practice**
When we find a website reading a file like this:

~~~text
website.com/view?file=image.jpg
~~~

We try feeding it:

~~~text
/etc/passwd/../../../../
~~~

We keep going back and back until we reach the building's main entrance (`/`), then walk straight into the password file. `../` literally means "step backward."

### 🎯 The Full Bug Bounty Scenario
1. You find an endpoint that accepts a filename parameter.
2. You try `../../../etc/passwd`.
3. If it works — you've got Path Traversal, and it might escalate into Local File Inclusion (LFI) if the file gets executed rather than just read.

### 🛡️ Blue Team Defense
Protection comes from path sanitization — rejecting any string containing `../`, or using something like `realpath()` and verifying the result still falls within the allowed directory.

### 🔬 Hands-On Practice

~~~bash
cd /tmp                                    # navigate to /tmp
mkdir -p test/sub                          # create nested directories in one go
echo "secret" > /tmp/test/secret.txt       # create a file containing the word "secret"
cd /tmp/test/sub                           # move into the nested folder

cat ../secret.txt        # relative path — works because it's short and simple
cat /tmp/test/secret.txt # absolute path — works from anywhere, regardless of location
~~~

> 💡 **Practical Rule:** In any script or exploit you write, always use absolute paths for critical operations. If your script relies on a relative path and an attacker manages to change the working directory at execution time, it could end up operating on a completely wrong file. This concept ties directly into PATH hijacking, which we'll revisit in the Environment Variables lesson.

## ✂️ Part 3: File Manipulation & Escaping Protection (Wildcards & Files)
Nothing too complex here — the commands themselves are simple:

| Command | Function |
|---|---|
| `touch file` | Creates the file if it doesn't exist, or updates its timestamp if it does. |
| `cp source destination` | Copies a file (or a directory with `-r`). The `-n` flag prevents overwriting an existing file. |
| `mv source destination` | Short for "move" — used for moving files or renaming them. |
| `rm` | Dangerous. Permanent deletion — there's no recycle bin. `-i` asks for confirmation before deleting, `-rf` forces recursive deletion. |
| `mkdir` | Creates a directory. `-p` creates nested directories all at once. |
| `rmdir` | Deletes an existing empty directory. |

### 🕵️ touch Is More Dangerous Than It Looks
I told you `touch` updates the last modification timestamp, right? But this goes much deeper from an attacker's perspective.

When you establish persistence on a machine (leaving behind a backdoor), the very first thing the Blue Team hunts for is recently modified files — often using something like:

~~~bash
find / -mtime -1
~~~

Professional attackers counter this with a technique called **Timestomping**:

~~~bash
touch -r /etc/hostname malicious_file.sh
~~~

This copies the timestamp of `/etc/hostname` (an old, legitimate system file) onto the malicious file — making it blend in with regular system files and evade timeline-based forensic analysis.

### 🛡️ Blue Team Counter:
This is exactly why forensic investigators don't rely solely on `mtime`. They also check the inode change time (`ctime`), which is much harder to tamper with, or the filesystem's own journaling logs.

### 🗑️ rm Doesn't Actually Delete Anything
Here's a crucial point: `rm` doesn't physically erase a file from the disk — it just removes the pointer to it. The actual data stays on the disk until something else overwrites it. This is exactly why forensics teams can "recover" deleted files.

If you need a real, secure deletion (say, after extracting sensitive data during an engagement), use:

~~~bash
shred -u sensitive_file.txt
~~~

## 💬 Part 4: Spaces and Quoting — The Foundation of Command Injection
**The Real Security Connection**
Every Command Injection vulnerability (a fixture of the OWASP Top 10) fundamentally comes down to the server taking user input and dropping it inside a shell command without proper escaping. If you understand how the shell splits commands by spaces, you'll understand exactly how an attacker "breaks" a command to inject another one.

**Real-World Example**
Say a website has a ping tool that runs:

~~~bash
ping -c 4 $user_input
~~~

If you input:

~~~text
8.8.8.8; cat /etc/passwd
~~~

And the server doesn't properly escape or quote the input, the shell interprets the semicolon as a separator between two commands and executes both. This is the exact same principle as spaces — just applied to a broader category called separator characters (space, semicolon, pipe, ampersand — the shell treats all of them as separators).

### 🔬 Hands-On Practice

~~~bash
mkdir "/tmp/my folder"    # note the quotes, because there's a space in the name
cd /tmp
ls -la                    # you'll see the name wrapped in quotes

cat my file.txt            # ERROR — gets split into argument1 and argument2
cat "my file.txt"          # works correctly
cat my\ file.txt            # works correctly, using an escape character
~~~

> 💡 **Best Practice in any Bash script you write:** always wrap variables in double quotes — `"$variable"`, never `$variable`. This difference prevents word-splitting issues and a whole category of script-level injection vulnerabilities.

## 🃏 Part 5: Globbing/Wildcards — A Genuinely Dangerous Vulnerability
There's a real, serious attack here called **Wildcard Injection** (or Argument Injection).

**The Full Story**
Imagine a cron job running as root, executing daily:

~~~bash
cd /var/backups && tar -czf backup.tar.gz *
~~~

The dangerous part: that `*` undergoes glob expansion — expanding to match anything sitting in that directory, even if the filenames look exactly like command-line options.

When the cron job runs, the `*` expands to include those maliciously-named files, and `tar` interprets them as command-line flags instead of filenames — executing your payload with root privileges, because the cron job itself was running as root.

> 🧨 **Exploit Spotlight:** This isn't theoretical — it's a documented, actively used privilege escalation technique, listed under `tar` in GTFOBins.

## 👁️ Part 6: head, tail, diff — Your Monitoring Toolkit

~~~bash
tail -f /var/log/auth.log
~~~

This is a fundamental tool in any Blue Team monitoring workflow. Every failed SSH login, every sudo usage, every privilege escalation attempt — this file exposes it all. It's the file that reveals brute-force attacks (hundreds of failed attempts from the same IP in a short window is your red flag).

| Command | Function |
|---|---|
| `head` | Shows the first 10 lines of a file. |
| `tail` | Shows the last 10 lines of a file. |
| `tail -f` | Keeps the file open live. Perfect for watching brute-force attempts unfold in real time, or as a Blue Teamer watching who's currently attacking you — you'll see new entries stream in line by line. |

## 🔗 Part 7: Links — The Most Dangerous Part of This Entire Lesson
Hard Links vs. Soft Links: don't waste your time memorizing Hard Links in detail. What matters to us is Soft Links (Symbolic Links), created with `ln -s`. Think of them as Windows shortcuts.

### 🧨 Exploit Spotlight: Symlink Attack
An attacker creates a fake link pointing to a sensitive file (like a password file). When the server or another program comes along to read the attacker's file, it gets tricked by the "shortcut" and ends up reading the sensitive file instead.

This attack is known as a Symlink Attack, or more formally: **TOCTOU (Time-Of-Check-Time-Of-Use) Race Condition**.

## 📦 Part 8: Compression — Zip Slip, a Vulnerability Directly Tied to Your Goals
Why is this the most important part of this lesson for web app pentesting?

Plenty of websites let users upload a `.zip` file (say, uploading a "theme" or a "plugin"). The server automatically extracts it. The problem: if the server doesn't validate the filenames inside the zip, an attacker can craft a zip containing a file named:

~~~text
../../../../var/www/html/shell.php
~~~

### 🥷 Why Hackers Love tar
As hackers, we absolutely love `tar`. Why? Because when we're exfiltrating data from a server, we're not going to pull it file by file — we bundle everything together, compress it, and ship it to our machine in one go. Memorize these two commands cold:

~~~bash
# To build the trap (compress):
tar -czvf data.tar.gz /folder_to_steal

# To spring the trap (extract):
tar -xzvf data.tar.gz
~~~

*(c = Create, x = Extract, z = Gzip, v = Verbose, f = File)*

**The Danger**
When the server extracts a malicious zip, the file doesn't land inside the normal upload folder — it gets placed exactly where its embedded path tells it to go, potentially leading straight to full Remote Code Execution through nothing more than a simple zip upload.

> 🧨 **Exploit Spotlight: Zip Slip**
Widely disclosed in 2018, this vulnerability affected hundreds of major projects — including several well-known Java and Node.js libraries. It's directly connected to the path traversal concept covered earlier: Path Traversal + Compression = Zip Slip.

## 🔍 Part 9 (The Most Important Part of This Entire Lesson): find, locate, which, whereis
`find` — Your First Privilege Escalation Tool in Any Engagement

After landing a shell on any machine, the second command you run — right after `whoami` — should be a variation of this:

~~~bash
find / -perm -4000 -type f 2>/dev/null
~~~

Here's exactly what's happening:
* `-perm -4000` → searches for any file with the SUID bit enabled.
* `2>/dev/null` → discards all error messages (like "Permission denied") into the void, keeping your screen clean.

### 🤔 So what exactly is SUID?
When a binary has the SUID bit set, it means: whenever any user runs it, it executes with the privileges of the file's original owner, not the privileges of the user running it. This exists for legitimate reasons — for example, the `passwd` command needs to write to `/etc/shadow` (which only root can normally write to), so it's given the SUID bit to let regular users change their own passwords.

### More Recon Tools
* `which nc` → We use this to ask the server, "Do you have Netcat available?" If it returns a path, we use it to spin up a Reverse Shell.
* `find` → It's not just for searching by name. Hackers use it to hunt for files with specific permissions (like SUID binaries that let you run a program with root privileges). This command is your most loyal companion during the Privilege Escalation phase.

## 🎬 Real-World Scenario: Putting It All Together
Let's say you discovered a Remote Command Execution (RCE) vulnerability on a website — meaning you can inject Linux commands and the site executes them.

**Step 1 — Where am I?**
~~~bash
pwd
~~~
→ Result: `/var/www/html`

**Step 2 — What tools do I have available to connect back to my machine?**
~~~bash
which python3
which nc
~~~

**Step 3 — Hunt for config files containing database passwords:**
~~~bash
find /var/www/html -name "*.php"
~~~
This pulls every PHP file that might be hiding a password.

**Step 4 — Bypass the WAF:**
The WAF was blocking the word `passwd` whenever you tried to read the sensitive file. So you got creative:
~~~bash
cat /etc/pa*wd
~~~
It worked — and you just earned yourself a $1,500 bounty.

---

## 🧾 Section Summary

### 🏆 The Golden Directories
* `/etc` → Configuration files and passwords.
* `/tmp` and `/dev/shm` → The hacker's staging ground for uploading tools (wiped on reboot).
* `/var/log` → The logs (where you need to erase your tracks).

### 🕳️ The Vulnerabilities
* `../../` → Path Traversal (climbing back toward the root).
* `*` and `?` → Wildcards (used to bypass WAFs).

### ⚔️ The Most Critical Commands
* `tar -czvf file.tar.gz /folder` → For compressing and exfiltrating data.
* `tar -xzvf file.tar.gz` → For extracting.
* `tail -f /var/log/syslog` → For live monitoring.
* `which [tool]` → Is this tool even available on the target?

---
*This content was studied and written in my own words based on the concepts covered in TCM Security's Linux 101 course. Any errors in explanation or terminology are entirely my own.*
