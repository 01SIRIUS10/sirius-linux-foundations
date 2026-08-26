# ✍️ 08 - Text Editors: nano & vim

## Status: 🟢 Completed

---

## 🤔 Why Do We Even Learn Text Editors?

### What Problem Does This Solve?
Imagine you've compromised a company's server and you're staring at a black terminal screen — no mouse, no VS Code, no Notepad. You find a script running with **root** privileges, and if you can inject a single line into it, the entire server becomes yours. **How do you write that line?**
This is exactly where **Command Line Text Editors** like `nano` and `vim` step in.

### 📍 Where This Shows Up in a Real Pentester's Life

| Use Case | Description |
|---|---|
| **🛠️ Editing Exploits** | You download exploit code from the internet onto the victim's server, but you need to edit the IP address inside it for it to work against you. |
| **🕳️ Persistence** | You open the server's configuration file and plant code that lets you get back in anytime you want. |
| **💣 The Great Escape (Privilege Escalation)** | This is the bombshell the lecturer never mentioned — and I'll break it down below. |

---

## 📝 Part 1: `nano` — Deceptively Simple
A straightforward program — you open it and start typing immediately. Save with `Ctrl+O`, exit with `Ctrl+X`, search with `Ctrl+W`.

### 🤔 Why Is It Taught First?
Because it's the easiest editor for beginners. But here's the catch: during an exploitation, when you receive a **Reverse Shell** from a victim, that shell is often "dumb" (a **Dumb Shell**). If you open `nano` inside a dumb shell, your screen freezes, the server kicks you out, and your entire exploitation collapses! This is exactly why professional hackers almost always prefer `vim`, or perform something called **TTY Spawning** first, before ever opening `nano`.

---

## 🐍 Part 2: `vim` — The Beast
`vim` operates using different **Modes**, and requires specific navigation, exit, and writing techniques.

### 🕰️ From the Roots: Why Is `vim` So Complex, and Why Does It Have Modes?
Back in the day, keyboards had no arrow keys and no mouse. So the same letter keys needed to serve **two purposes**: sometimes typing text, and sometimes moving the cursor or deleting. This is exactly why **Command Mode** and **Insert Mode** were created.

### 🚗 The Manual Car Analogy
Think of `vim` like a manual transmission car. The moment you get in, you're in **"driving" mode (Command Mode)** — the steering wheel and gear stick move the car around, but they don't write anything. If you press `i`, you've essentially stepped out of the car and picked up a pen (**Insert Mode**) to write freely. Finished writing? Press `Esc` to get back behind the wheel (Command Mode) again.

### 🔑 The Life-or-Death Summary (Memorize This)

| Action | Key |
|---|---|
| Start typing | `i` |
| Stop typing | `Esc` |
| Save and quit | `:wq` |
| Everything's broken, force quit without saving | `:q!` |

---

## 💣 The Bombshell the Lecturer Never Mentioned: GTFOBins
In Red Teaming, sometimes a careless admin grants you permission to run `vim` with root privileges via `sudo`.
As a hacker, you're **not** going to open `vim` to write text! You'll open `vim`, and while in **Command Mode**, you'll type:

~~~vim
:!/bin/sh
~~~

This command tells `vim` to spawn a shell from inside itself. And since `vim` is running as root, the shell that opens is a **Root Shell**! The entire system falls into your hands because of a simple text editor!

### 🔬 Try It Now on Kali
1. The command: `sudo vim` (opens the program as root).
2. The technique: the moment it opens, type a colon, followed by this spell: `:!/bin/sh`, then hit Enter.
3. The result: you'll find yourself dropped into a completely different command shell. Type `whoami` and it'll tell you `root`!
> **Why did this happen?** This is a Living off the Land technique exploiting built-in system tools. `vim` was designed to be able to execute external commands, and we've turned that feature against the system itself.
*(To exit, type `exit`, then `:q!`.)*

---

## 🎬 Real-World Scenario (Red Team Operation)
1. You managed to get into the company's server as a regular user.
2. You start hunting for admin activity, and you find a file called `backup.sh` running in the background with root privileges — and you have permission to edit it.
3. You type: `vim backup.sh`
4. You press `i` to enter Insert Mode.
5. You scroll down with the arrow keys to the last line, and type in the command that will send a reverse shell back to your machine.
6. You press `Esc` to return to Command Mode.
7. You type `:wq` and press Enter.
8. A minute later, the script executes, and it throws you a connection with root privileges. Congratulations — the bounty is in your pocket.

---

## 🧾 Section Summary — The Essentials

**In nano:**
* Save: `Ctrl+O` then Enter.
* Exit: `Ctrl+X`.

**In vim (Life or Death):**
* Enter writing mode: `i`
* Exit writing mode: `Esc`
* Save and exit: `:wq`
* Force exit (if everything broke): `:q!`
* Hacker magic (GTFOBins): If running under `sudo`, type `:!/bin/sh` to obtain a root shell.

### 🔑 Tattooed-in-Your-Brain Summary
> *"nano is your quick notepad, but it can betray you in a dumb shell. vim is the tank — you enter with i, escape with Esc, save with :wq, and if the world falls apart, you smash a chair through the club with :q!. And if you ever catch vim running with root privileges, the spell :!/bin/sh brings the entire server crashing to its knees."*

---

## 📂 Bonus: How to Actually Open a File and Work With It

~~~bash
nano filename.txt
~~~

### 🎹 Full nano Keyboard Reference

| Key | Function | Practical Note |
|---|---|---|
| `Ctrl+G` | Open the Help page | If you forget a command, you don't have to abandon the file and open Google |
| `Q` | Exit the Help page | — |
| `Alt+X` | Close the bottom menu bar, freeing up more writing space | Useful on small screens, like a narrow SSH session |
| `Ctrl+V` | Page Down | — |
| `Ctrl+Y` | Page Up | — |
| `Ctrl+C` | Shows your exact cursor position (line and column number) | Extremely useful in large files like log files |
| `Alt+/` | Jump to the end of the file | — |
| `Alt+/` (or `Ctrl+Home`) | Jump to the beginning of the file | — |
| `Alt+U` | Undo the last edit | Important note: undoes the entire line, not just a single character |
| `Alt+E` | Redo | — |
| `Alt+A` | Start a selection from the cursor's position | Like marking the first point you'll cut from |
| `Alt+^` (usually `Alt+Shift+6`) | Copy the selected text | Note: the character the cursor is sitting on at the moment of copying does NOT get copied |
| `Ctrl+U` | Paste | — |
| `Ctrl+K` | Cut the entire line, or the selected text | — |
| `Ctrl+W` | Search for text | Like Ctrl+F in any regular program |
| `Alt+W` | Jump to the next search match | — |
| `Alt+Q` | Jump to the previous search match | — |
| `Ctrl+O` | Save (Write Out) | You must press Enter afterward to confirm the filename |
| `Ctrl+X` | Exit | If there are unsaved changes, it will ask whether to save or not |

---

## 🎭 The Three Modes of vim

| Mode | Description |
|---|---|
| **Command Mode** (default) | Every key you press is interpreted as a command, not a character to be typed. |
| **Insert Mode** | This is where it behaves like nano — every key you press actually types into the file. |
| **Last Line Mode** (Command-line mode) | Entered by typing `:`, used for things like saving, quitting, searching, and find-and-replace. |

### 🔎 Searching in vim

~~~vim
/search_word
~~~
Then press Enter. After that:

| Key | Action |
|---|---|
| `n` (lowercase) | Next match |
| `N` (uppercase) | Previous match |

### ✍️ Writing and Editing (Insert Mode)

| Command | Function |
|---|---|
| `i` | Enter Insert mode before the cursor's position |
| `a` | Enter Insert mode after the cursor's position (append) |
| `Esc` | Exit Insert mode back to Command mode |

> 🤔 **Why does the difference between `i` and `a` even matter?** If your cursor is sitting on the first letter of a word and you want to type something before it, use `i`. If you want to type right after the character your cursor is sitting on, use `a`. It's a small detail, but it saves significant time once it becomes second nature.

---

## 🔐 Advanced Security Angles

**1. vim's Relationship With the Shell and Binary Exploitation**
In CTFs and real pentesting, if you run `sudo -l` and find `vim` or `nano` listed as available, immediately go check GTFOBins:

~~~bash
sudo vim -c ':!/bin/sh'
~~~

**2. vim in Reverse Shells and Payload Crafting**
When you're writing a payload or script on a compromised server, the only available tool is usually `vim` or `nano`. You need to be fast with them, because any delay could give the Blue Team time to detect you.

**3. From the Blue Team's Perspective (Detection)**
If someone opens `vim` or `nano` with sudo privileges on sensitive files like `/etc/passwd` or `/etc/shadow`, this gets logged in:

~~~bash
/var/log/auth.log   # on Debian/Ubuntu
/var/log/secure     # on RHEL/CentOS
~~~

**The IOC (Indicator of Compromise)** here: unusual use of sudo with text editing tools on sensitive system files, especially when the user isn't normally an administrative account.

**4. Common Beginner Mistakes**
* Forgetting they're in Insert mode and trying to run Command mode actions (the famous, embarrassing mistake everyone makes at least once).
* Using `dd` unintentionally and deleting an important line without immediately undoing it.
* Opening `vim` with root privileges unnecessarily (running `sudo vim file.txt` when they could've edited it with their normal permissions) — this needlessly widens the attack surface.

---
*This content was studied and written in my own words based on the concepts covered in TCM Security's Linux 101 course. Any errors in explanation or terminology are entirely my own.*
