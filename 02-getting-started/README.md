# 🖥️ 02 - Getting Started: The Terminal, The Prompt & Basic Navigation

## Status: 🟢 Completed

---

## 🎯 Setting the Scene

We already talked about the black screen. Now let's imagine a real scenario: you found a vulnerability in a web application and managed to obtain something called a **Reverse Shell** — meaning the victim's server is now calling back to your machine and handing you control.

In large enterprise environments, servers **do not** have a graphical interface (GUI). No mouse, no icons, nothing. If you don't know how to speak the language of the command line, you're basically walking into a bank in complete darkness — and the **Linux CLI is your flashlight**.

---

## 🧩 Understanding The Prompt

The prompt usually looks something like this:
SIRIUS@hostname:~$

text


Let's break down every single piece of it:

| Component | Meaning |
|---|---|
| **Username (`SIRIUS`)** | The Kernel (which we covered in the previous lesson) needs to know *who* is requesting resources in order to decide what privileges to grant. Every process in Linux is tied to a **UID (User ID)**, and the username is simply a human-readable alias for that ID. |
| **Hostname** | Extremely important when you're working across multiple machines simultaneously (multiple SSH sessions). This is the first thing that tells you the difference between "I'm on my own machine" and "I'm inside the target." |
| **Current Working Directory (`~`)** | The tilde (`~`) is shorthand for `/home/SIRIUS`. The system always needs to know "where are we right now," because any command you run without specifying a full path gets executed relative to this location. |

---

## ⚠️ Critical Concept: Case Sensitivity

Here's something that **will** cause you a frustrating error one day if you don't internalize it now: **Linux is case-sensitive.**

`Passwords.txt` and `passwords.txt` are two completely different files as far as Linux is concerned.

**Why does this matter for security?** In bug bounty hunting, you sometimes find an endpoint behaving with different case-sensitivity than expected — and that alone gives away information about how the backend is built (a case-sensitive filesystem like Linux, versus a case-insensitive one like Windows).

---

## 🔑 The Golden Question: Who Am I?

Everything the prompt shows you answers one core question: **who are you, on which machine, and where are you standing right now.**

As a Red Teamer, this matters enormously — because the moment you compromise a server, the first question you ask yourself is: *"Who am I? Do I have any real privileges or not?"*

- The `$` symbol means you're a **regular user** — minimal privileges.
- Your ultimate goal in almost every engagement is to flip that `$` into a `#`.
- The `#` symbol is the **root** prompt — the user who can do literally anything.

This transformation process has a name: **Privilege Escalation**.

---

## 🕵️ The Holy Trinity of Reconnaissance

At the very start of any engagement, three commands almost always go together, and they form the foundation of basic **Reconnaissance**:

### `whoami`
Tells you your current identity. As an attacker, you're checking: am I `www-data` (a web server user), a regular user, or already root?

### `pwd` (Print Working Directory)
Tells you exactly which "room" you're standing in on the server.

### `ls`
Turns on the lights — shows you what exists in the current directory.

**Important catch:** `ls` by default hides files that start with a dot (`.`). To reveal them, use:

```bash
ls -a
Why does this matter for hackers? Hidden files like .bash_history — which logs every command the admin has typed (sometimes including accidentally-typed passwords) — or .ssh, which may contain private keys granting passwordless server access, are pure gold during recon.

The very first command I run the moment I land on any server is ls -la.

🌳 Navigation: The Tree Analogy
Let's introduce one more command:

Bash

cd ..
This moves you one step back to the parent directory.

Think of the server as a tree:

pwd tells you which branch you're currently standing on.
ls shows you the leaves on that branch.
cd is you jumping from branch to branch, like a monkey.
cd .. takes you one step back toward the trunk.
📖 Reading Files
To read the contents of a file, we have a few tools:

Command	Behavior
cat	Dumps the entire file content onto the screen at once.
more	Displays content page by page, but only scrolls forward.
less	The best of the bunch — page by page, scroll up and down freely, and supports searching.
Why is it called cat?
It's not about cats — it's short for concatenate, because it was originally built to merge and display file content in one go.

Important limitation: if you have a file with a million lines, cat is a terrible choice — it'll flood your screen and you'll only see the last 50 lines, with everything else scrolling past in a blur.

Real-World Example: LFI Exploitation
If you discover a Local File Inclusion (LFI) vulnerability — which allows you to read server files through the web application — you'd tell the website to read:

Bash

cat /etc/passwd
If the file content gets displayed to you, congratulations — you just earned yourself a bounty.

⚠️ Warning: Never cat a Binary File
Never use cat on a binary file (images, executables — anything that's raw 1s and 0s). Doing so will flood your terminal with garbled symbols and can freeze it entirely. If this happens, type:

Bash

reset
and hit Enter to clean the screen and bring it back to sanity. This is exactly why less is the smarter choice for unknown or large files.

🆘 Getting Help
Nobody memorizes every command and every flag — not you, not me, not Google's engineers, not the world's top hackers. What matters is understanding what a tool does conceptually, and knowing where to look for the details when you need them.

Every Linux command has a manual page — think of it as the instruction booklet that comes with any new device:

Bash

man ls
If you want something quicker and shorter than a full manual page:

Bash

ls --help
There's also a fantastic website (requires internet — which won't always be available during competitions) called explainshell.com. Paste any long, intimidating command from a write-up into it, and it'll break down every single part for you.

⚔️ Attacker vs. Defender Perspective
From the Attacker's Side
You're always hunting for forgotten information. cat gets used to read configuration files. ls -la gets used to hunt down backup files hiding in plain sight.

From the Defender's Side
If you're monitoring a server and you spot a regular user suddenly running whoami followed by:

Bash

cat /etc/shadow
(the file containing encrypted password hashes, which requires root access) — your SIEM solution should immediately fire an alert: someone is attempting privilege escalation.

🧾 Section Summary
Recon & Navigation
whoami → Who am I?
pwd → Where am I?
ls -la → What's around me, including hidden files (the -a stands for "all")?
cd → Move around. cd .. goes one step back, cd ~ returns home.
File Reading
cat [file] → Dumps the entire file at once (great for small files like /etc/passwd).
less [file] → Best for large files (like logs), with built-in search using /.
Getting Help
Forgot what a command does? Append --help to it, or run man [command] before it.
The Golden Rule
$ = limited user. # = Root, the ultimate goal.

🎯 Why Flags Aren't Everything
Don't waste your energy memorizing every single flag — 90% of them have zero practical use in a real pentesting scenario, and if you ever need one, you'll simply search for it. We're aiming to become craftsmen, not university professors reciting theory.

💪 Practice Challenges
Challenge 1 — Easy
Open the terminal, find out who you are with whoami, figure out where you're standing with pwd, and display every file in the current directory — including hidden ones — with full detail.

Challenge 2 — Medium Scenario
Navigate to Linux's password file, located at /etc/passwd. Display it on screen using cat. Were you able to view it?

Challenge 3 — Hard (Requires Thinking)
Using that same /etc/passwd file, open it with a command that allows you to search within it. Once it's open, search for the word root. (Hint: you'll need the reading tool that supports search — think about how we do that.)

🌐 Bonus: OverTheWire — Bandit
Once you've grasped these concepts, go put them into practice on OverTheWire: Bandit. Start with Levels 0, 1, and 2:

Level 0: Teaches you how to connect via SSH (we'll cover this later) and read a file using cat.
Levels 1 & 2: Teach you how to read files with unusual/tricky names using exactly what we covered in this section.
🧠 Comprehension Check: "Did You Actually Understand?"
Scenario: You've compromised a website, landed on a black screen, typed whoami and got www-data back. You then run ls -la and spot a hidden file named .db_config.

Question: Which command would you use to quickly view the contents of that file, and why?

⌨️ Quick Reference: Controls Inside less
Key	Action
Space	Move to the next page
Arrow Up/Down	Move line by line
/root	Search for the word "root"
n	Jump to the next search result
q	Quit less
🔑 Final Key Takeaway
The prompt tells you everything you immediately need to know: who you are (and whether $ or # is your real security status) and where you are. The very first command you run on any new machine should always be ls -la, never plain ls — because hidden files like .bash_history and .ssh are the true treasure of any recon phase. The man page is your permanent reference manual, and redirection with > is the very first building block of any real-world bug bounty workflow.

This content was studied and written in my own words based on the concepts covered in TCM Security's Linux 101 course. Any errors in explanation or terminology are entirely my own.
