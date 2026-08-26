# ⚙️ 09 - Process Management

## Status: 🟢 Completed

---

## 🧬 What Is a Process, Really?
Anything running on a computer — from the terminal window you have open right now, to a background service quietly ticking away in the dark — is a **process**. A process is a living entity: it has an **ID**, an **owner**, a **state**, and a **parent-child relationship** with other processes.

### 🤔 What Would Happen If We Couldn't Control Processes?
- A program **freezes**, and you have no way to stop it.
- You can't detect a **cryptominer** silently stealing your CPU.
- You can't build any **automation** (scheduled tasks) whatsoever.
- Detecting or stopping **malware** becomes practically impossible.

### 📍 Where This Shows Up in a Real Pentester's Life

| Scenario | Application |
|---|---|
| **Just landed a shell** | First move: `ps aux` — what's running? Is there an antivirus? A monitoring tool? |
| **Need Persistence** | The most famous method: a malicious **cron job**. |
| **Need to disable EDR** | You need to deeply understand `kill` and **signals** before the EDR catches you. |

---

## 📸 Part 1: `ps` — A Snapshot of Processes
`ps` stands for **Process Status**. It's a **snapshot** — a single frozen frame in time, not a live feed that updates itself.

~~~bash
ps
~~~

Expected result: you'll only see processes belonging to the current user, in the same terminal window — usually just two lines: the bash shell itself, and the ps command you just ran.

### 🤔 Why Is It So Limited by Default?
Because of the core Unix philosophy: "The default should be safe and minimal — if you want more, ask for it explicitly." If `ps` showed you the entire system by default, you'd drown in irrelevant information every single time you typed it.

### 🔤 The Three Types of Options

| Style | Format | Example |
|---|---|---|
| UNIX/POSIX | single dash, can be combined | `-ef` |
| BSD | no dash at all | `aux` |
| GNU long options | double dash, full word | `--all, --user` |

### 🔬 Try It Now

~~~bash
ps aux | less -S
~~~

Expected result: a table with columns USER, PID, %CPU, %MEM, VSZ, RSS, TTY, STAT, START, TIME, COMMAND — for every process on the entire system, not just yours.
Why `less -S`? The `-S` flag prevents word-wrapping — long lines get truncated instead of wrapping onto a new line, keeping the table clean and readable.

### 📊 Breaking Down Every Column — And Why It Exists

**USER / UID**
The owner of the process. The real security significance: a process inherits the same privileges as its owning user. If you find a process running as root that shouldn't be, that's a massive red flag — could be a successful privilege escalation, or a backdoor.

**PID (Process ID)**
A unique number for every process. Why do we need it? Because when you use `kill`, you must specify exactly which process you mean (like a national ID number for a person).

**PPID (Parent Process ID)**
Who "gave birth" to this process. This is critically important for security:

> 🎬 **Real Red Team Scenario:** If you see a process named `bash` or `sh`, and its PPID belongs to a process named `apache2`, `nginx`, or `mysqld`... this is an extremely dangerous compromise indicator! Why? Because a normal web server should never spawn a shell process. This is exactly what happens when an attacker exploits Remote Code Execution (RCE) in a web application and obtains a reverse shell — the shell becomes a "child" of the web server process.

> 🛡️ **From the Blue Team's Perspective:** EDR tools like CrowdStrike Falcon monitor exactly this relationship (process tree / parent-child relationships) to detect attacks. If this escalation happens (web server → shell), it triggers an immediate alert.

**%CPU**
CPU usage percentage. Real scenario: your machine suddenly slows down and the fan goes wild? First thing to check:

~~~bash
ps aux --sort=-%cpu | head
~~~

This shows you who's eating your CPU — could be cryptominer malware.

**STIME / START**
When the process started. From a forensics perspective: if a process started at a strange time (say, 3 AM) while you weren't awake, that's an indicator someone else accessed the system.

**TTY**
The terminal associated with the process. A question mark (`?`) means no terminal is attached — completely normal for background/daemon processes like `systemd` or `cron` itself.

> 🤔 **Philosophical question:** why do some processes have no TTY at all? Because they were launched without direct human intervention from a terminal — started during boot, or via scheduling like cron. This matters a lot: if you find a reverse shell process with an unusual TTY, or none at all when it should be interactive, that suggests someone is trying to hide their presence.

**TIME**
The total CPU time the process has consumed since it started (not the elapsed wall-clock time — this distinction matters!). If a process has been running for an hour but its TIME shows `00:00:01`, it isn't actually consuming much — it's mostly sleeping.

**COMMAND**
The command that launched the process. A security trick: attackers try to mask process names to look legitimate (naming malware `systemd` or `kworker` to blend in with real system processes). The defense: always verify the full path of the executable, not just the name.

### 🔧 Additional Important ps Options

~~~bash
ps -U root -u root u
~~~
Shows only processes owned by root. When to use it? When you need to check what's running with administrative privileges to confirm nothing suspicious is happening.

~~~bash
ps -eH
~~~
Hierarchical display — shows who's the child of whom, using indentation.

~~~bash
pstree
~~~
A separate tool showing the same hierarchical tree, but visually clearer and simpler, without PIDs or extra info. Great for quickly understanding "who launched whom" without clutter.

~~~bash
pstree -p
~~~
The `-p` flag adds PIDs next to each process in the tree, giving you the best of both worlds.

---

## 📡 Part 2: `top` — Live Monitoring

### 🤔 Why Do We Need `top` If We Already Have `ps`?
`ps` is a photograph. `top` is a live, continuously refreshing video feed. If your machine is slow right now, you need to see what's happening at this moment, not a snapshot from a second ago.

~~~bash
top
~~~

### 📋 The Summary Area at the Top Shows:

| Metric | Meaning |
|---|---|
| uptime | How long the machine has been running |
| load average | The system's average load (3 numbers: last 1 min, last 5 min, last 15 min) |
| Tasks | Number of processes (running/sleeping/stopped/zombie) |
| CPU usage | Breakdown of processor usage |
| Memory usage | RAM being used |

> 🤔 **Philosophical Question:** What does a load average of 4.5 on a machine with only 2 cores actually mean?
It means there's a queue of processes waiting their turn for the CPU, and the system is overloaded beyond its capacity. If this number consistently exceeds the core count, it's a genuine performance problem indicator (or a local DoS attack if someone is deliberately consuming resources).

### ⌨️ Important Shortcuts Inside top

| Key | Action |
|---|---|
| `d` | Change the refresh interval (the instructor set it to 1 second instead of the default 3) |
| `k` | Kill a process from directly inside top (it'll ask for the PID, then a signal) |
| `q` | Quit |
| `←` `→` | Navigate between columns if your screen is too narrow |

---

## 🎭 Part 3: Foreground vs. Background Processes
When you run a normal command, the shell waits (freezes) until it finishes. This means the command is running in the **foreground**. You can't have more than one foreground process in the same terminal at the same moment.

### 👶 ELI5
Imagine you're on a phone call with someone (foreground) — you can't talk to someone else on the same line at the same time. But you can have other people "on hold" (background) waiting for you.

### 🚀 Running a Process in the Background

~~~bash
xeyes &
~~~

The `&` at the end means: "Run this command, but don't wait for it — give me back control of the shell immediately."

~~~bash
sleep 100 &
~~~

Expected result: you'll get something like `[1] 12345` (job number and PID), and you'll immediately return to the prompt without waiting 100 seconds.

### 🔄 Moving a Process Between Foreground and Background

~~~bash
Ctrl+Z    # suspends the process running in front of you
jobs      # shows all jobs in this shell and their status
bg        # resumes the last suspended job, in the background
fg        # brings the last job back to the foreground
fg %2     # brings job number 2 specifically back to the foreground
~~~

### ⚡ The Difference Between Ctrl+C and Ctrl+Z

| Shortcut | Signal Sent | Effect |
|---|---|---|
| `Ctrl+C` | SIGINT (interrupt) | Terminates the process completely |
| `Ctrl+Z` | SIGTSTP (stop) | Temporarily suspends the process — it's still in memory |

> 🤔 **Philosophical Question:** Why would we ever need to suspend a process instead of just closing it?
Because sometimes you're in the middle of a long task (like `vim` open on an important file with unsaved changes) and you want to "hide" it briefly to do something quick in the same terminal, then return to it exactly where you left off. If you'd closed it with `Ctrl+C`, you would've lost all your unsaved changes.

### ⚠️ The Problem
If you launch a process in the background with `&`, and then close the terminal (or your SSH session disconnects), the process gets terminated automatically — because it receives a SIGHUP (hang up) signal when the terminal closes.

### 🎬 A Critical Real-World Scenario for You as a Pentester
You've established a reverse shell on a victim's server, and you want to run something that keeps running even if your SSH session drops (like an nc listener, or a data exfiltration script). The solution:

~~~bash
nohup long_running_command &
~~~

`nohup` = "no hang up" — it makes the process ignore SIGHUP and keep running even after you close the terminal.
Or, if the process is already running:

~~~bash
disown -h %1
~~~

This detaches the job from shell control, so if the shell closes, the job keeps running.

### 🥷 From a Red Team Perspective — A Much Better Tool
Instead of `nohup`, professionals use `tmux` or `screen`:

~~~bash
tmux new -s persist
~~~

This lets you detach from the entire session (`Ctrl+B` then `D`) and reattach to it later, even after your SSH connection drops — and it's far more powerful than `nohup` because it preserves your entire terminal state, not just a single process.

---

## 🚦 Part 4: Process States and Signals — The Most Critical Security Part of This Entire Section

### The Four States (and the Fifth One Everyone Forgets)

| State | Symbol | Meaning |
|---|---|---|
| Running | `R` | The CPU is actively executing it right now |
| Sleeping | `S` | Waiting for a resource (like data from a file or the network). Not consuming CPU while asleep |
| Stopped | `T` | Suspended by a signal (like `Ctrl+Z`). Completely inactive until reactivated |
| Zombie | `Z` | The process is actually dead, but its parent hasn't collected its exit status yet |

### ☠️ Uninterruptible Sleep (D)
A dangerous state — the process is waiting on an I/O operation (like reading from a damaged disk, or a hung network share) and cannot even be stopped with `kill -9`! If you see many processes stuck in D state persistently, it's a strong indicator of a hardware or storage problem.

### 🧟 The Zombie Process — Explained
Imagine a student (child process) finished their exam, handed in the paper, and walked out of the exam hall (essentially, "died" as a person in that room). But the teacher (parent process) hasn't yet collected their final grade from the record sheet. The student has effectively "finished," but their name still sits in the hall's records as "awaiting result." This zombie takes up a tiny slot in the process table but doesn't consume any real CPU or memory.

### 🧨 How Is This Dangerous, Security-Wise?
If you have an application (like a web server) that constantly forks child processes (spawning a new process for every request) and has a bug where the parent never collects the results of its children, zombie processes will accumulate until the system's entire PID table fills up (there's a maximum limit on the number of PIDs). This is a form of Denial of Service (DoS) through PID exhaustion. An attacker could deliberately exploit a bug like this if they find one in a specific application.

### 📶 Signals
`kill` doesn't just mean "kill" — it means "send a signal." Killing is just one of many possible signals.

~~~bash
kill -l
~~~

### 🔑 The Three Most Important Signals to Memorize by Heart

| Signal | Number | Meaning |
|---|---|---|
| SIGHUP | 1 | "Reload your configuration" — without a full restart |
| SIGTERM | 15 (default) | "Please shut down politely" — the process can ignore it or clean up first |
| SIGKILL | 9 | "Die immediately" — cannot be ignored, no chance to clean up |

---

## 🕰️ Part 5: Scheduling — Cron, and the Most Dangerous Persistence Technique in Linux

### 🤔 Why Does cron Even Exist?
You want something to happen automatically and repeatedly — a daily backup, weekly cleanup of temp files, an hourly report update. Without scheduling, you'd have to sit there and manually run the command every single time.

### 📂 Two Types of Crontabs

| Type | Location | Purpose |
|---|---|---|
| System-wide | `/etc/crontab`, `/etc/cron.d/`, `/etc/cron.daily/hourly/weekly/monthly` | General system-wide tasks (owned by root) |
| User crontab | Per-user | Managed with the commands below |

~~~bash
crontab -e     # edit the current user's schedule
crontab -l     # list the schedule
crontab -r     # delete the entire schedule
~~~

### ⏰ The Time Syntax — The Part That Needs to Be Burned Into Your Brain

~~~text
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of week (0-6, Sunday=0)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
~~~

~~~bash
5 1 2 * *  command      # 1:05 AM on the 2nd of every month
*/5 * * * *  command    # every 5 minutes
0 3 * * 0   command     # every Sunday at 3 AM
~~~

### 🔁 anacron — Solving the Problem of Machines That Get Shut Down
Regular `cron` assumes the machine is running 24/7. If your machine (a laptop, for example) is shut down at the scheduled time, the job simply never runs. `anacron` solves this by ensuring the task runs at least once during its period (daily/weekly/monthly), even if the machine was off at the exact scheduled moment.

---

## 🧠 Final "Tattoo" Summary — What Must Stay With You Forever
* `ps aux` / `ps -ef` = a snapshot, `top`/`htop` = a live feed. Use the first for quick recon, the second when you suspect something's eating resources right now.
* The parent-child relationship between processes (PPID) is the single most important sign of a compromise — a web server spawning a shell process is nearly a guaranteed breach.
* `&` and `Ctrl+Z` + `bg`/`fg` control foreground/background, but if you need a process to survive after the session disconnects, use `nohup`, or better yet, `tmux`.
* `SIGTERM` (15) asks politely, `SIGKILL` (9) kills by force — and which one you use has a real impact on forensics and data integrity.
* Cron is one of the most powerful Persistence techniques in Linux, and any script running with root privileges that's writable by a regular user equals instant privilege escalation.
* Modern systems use `systemd`, not SysVinit — you need to know both, because you'll encounter legacy systems in real-world work.

---
*This content was studied and written in my own words based on the concepts covered in TCM Security's Linux 101 course. Any errors in explanation or terminology are entirely my own.*
