# Bash Scripting for Hackers 🐚

A complete breakdown of Bash scripting fundamentals — written from an offensive security and defensive analyst perspective.

## 📌 Why Does Bash Scripting Even Matter?
As Red Teamers and Bug Bounty Hunters, we don't write complex programs in Bash — we write scripts for one core purpose:

**Automation**
Instead of typing 10 commands manually every time you need to scan a network, you write them once into a single file and let it run itself.

**Payloads (Weapons)**
You write a small script, upload it to a compromised server, and use it to trigger a reverse shell or exfiltrate data.

### 🎯 The Problem It Solves
Imagine you're doing a pentest. Every single day you repeat the same steps:
1. Enumerate subdomains
2. Run a port scan
3. Filter the results
4. Save everything to a file

If you type every command manually each time, you will:
* Waste time
* Make mistakes (human error)

**The solution:** write these commands once into a file, and that file executes them sequentially — exactly as if you were typing them yourself, but automatically.

**If Bash Didn't Exist**
Every single security tool would need to be written as a standalone program in a full programming language (C, Python, etc.) — even for something as simple as "run 3 commands in sequence." This would massively slow down the work of any pentester or SOC analyst.

### 🔎 Where This Actually Shows Up in Real Life

| Use Case | Description |
|---|---|
| **Recon Tools** | Tools like `theHarvester` and `sublist3r` were originally simple shell scripts based on this exact logic. |
| **Privilege Escalation** | If you find a SUID bash script on a compromised server, understanding the script's structure shows you exactly how to exploit it. |
| **Persistence (Red Team Ops)** | Bash scripts placed in cron jobs help maintain your access to a system automatically. |
| **Blue Team / SOC** | Scripts written to scan logs automatically every hour for anomalies. |

---

## 1️⃣ The Shebang — `#!/bin/bash`
The shebang is the first line of any script. It tells the system exactly which interpreter should run the script.

```bash
#!/bin/bash
```

### Why This Actually Matters (From the Ground Up)
When you make a script executable and run it with `./script.sh`, the Linux kernel reads the first two bytes of the file. If it finds `#!`, it takes the rest of that line, uses it as the interpreter, and passes the script's name to it as an argument.
In other words, what actually happens under the hood is:

```bash
/bin/bash ./script.sh
```

**ELI5**
It's like writing on an envelope: "To be opened using: key #5." The system doesn't try to understand the language inside the file by itself — it just looks at the "key" written on top and hands the file to whoever can open it.

### Philosophical Question: What If You Don't Write a Shebang?
Try it yourself:

```bash
echo 'echo "hi"' > noshebang.sh
chmod +x noshebang.sh
./noshebang.sh
```

### The Difference Between These Two

```bash
#!/bin/bash
#!/usr/bin/env bash
```

The second form is far better practice in professional scripts, because it searches for bash in your `PATH` instead of assuming it exists exactly at `/bin/bash`. This matters a lot if the script will run across different systems (like macOS, or rare Linux distributions where bash lives elsewhere).

---

## 🏗️ Pillar One: Structure — The Shebang & Execution
The shebang line (`#!/bin/bash`) is the skeleton's first bone of any script.

**ELI5**
It's the sign hung on the factory door telling the system: "When you come to read this file, send it to the factory manager called `bash`, because he's the one who understands this language." Without it, the system might get confused.

### Execution & Permissions
To run the script, you need to either:

```bash
chmod +x script.sh
./script.sh
```

Or run it directly through the interpreter:

```bash
bash script.sh
```

> ⚠️ **The Hacker's Trick (Rarely Mentioned)**
> When you upload an exploit script to a compromised server, the admin often locks down execute permissions (meaning you can't do `chmod +x`). A skilled hacker doesn't stop there — they force the script to run regardless, using:
> ```bash
> bash script.sh
> ```
> or a lesser-known command:
> ```bash
> source script.sh
> ```

---

## 2️⃣ Conditional Statements — "If This Happens, Do That"
Standard control structures: `if`, `elif`, `else`, and `case`.

### The Security Takeaway
You don't need to overthink conditionals. In offensive scripting, `if` is used most often to answer one of two questions:
1. "Is this tool installed or not?"
2. "Am I root or a regular user?"

### Important File Tests

| Test | Meaning |
|---|---|
| `-d` | Is this a directory? |
| `-f` | Is this a file? |
| `-e` | Does this exist at all? |

### Real-World Security Scenario
Writing a script that steals a password file:

```bash
if [ -f /etc/shadow ]; then
    cat /etc/shadow > /tmp/stolen.txt
fi
```
Translation: "If the shadow file exists, read it and hide it in /tmp."

### Root Check Example

```bash
if [ $(id -u) -eq 0 ]; then
    echo "I am ROOT, Server is Mine!"
else
    echo "I am a normal user"
fi
```
**Expected result:** If you're logged in as a regular user, it prints the second message. If you're running with sudo/su prior, it prints the first.
**Why it works this way:** We executed the command `id -u`, which returns the numeric user ID. If that number equals zero (root), `-eq` matches, and the victory message prints.

---

## 3️⃣ Loops — Repetition & The Machine
Covers Definite loops (`for`) and Indefinite loops (`while`).

### The `for` Loop — Your Most Important Weapon
We use `for` loops to iterate over IPs or ports.

**Real hacker workflow:** You want to quickly scan an entire network (say, 192.168.1.1 through .254) to see which hosts are up, without using Nmap. You use a `for` loop to iterate through the numbers and ping each one.

```bash
for i in {1..254}; do
    ping -c 1 192.168.1.$i
done
```

### The `while` Loop — Persistence
As Red Teamers, we use `while` loops for infinite loops to maintain persistence. Example: a script running `while true` that attempts to connect back to your machine every 5 seconds.

---

## 4️⃣ Command Line Arguments — `$1`, `$2`, `$#`
Covers how a script receives input data at runtime.

### From the Ground Up
Just as you'd type `nmap 192.168.1.5`, the IP here is an argument. Your script should be smart enough to receive the target IP from outside, instead of hardcoding it inside the script every time.

### Symbols to Memorize

| Symbol | Meaning |
|---|---|
| `$0` | The script's own name |
| `$1`, `$2` | First input, second input (e.g., IP, then port) |
| `$#` | Number of inputs (to confirm the user provided everything correctly) |

### Quick Reference Cheat Sheet

```bash
#!/bin/bash                    # Always the first line
bash script.sh                 # Run without needing chmod
```

**Important File Tests:**
* `-f file` — Does the file exist?
* `-d dir` — Does the directory exist?
* `-eq` (numeric equality) vs `==` (string equality)

**Essential Loop (Network Scanning):**
```bash
for i in {1..254}; do ping -c 1 192.168.1.$i; done
```

**Arguments:**
* `$1` — First target
* `$#` — Argument count (validation)

---

## 5️⃣ Anatomy of Every Script

```bash
#!/bin/bash
set -euo pipefail   # Always include this — not in most tutorials, but essential
```

* **Shebang (`#!/bin/bash`)** — Must be the first line; defines which interpreter runs the script.
* **`#` after this** = a comment; Bash ignores it.
* **`chmod +x script.sh`** is required to run it as `./script.sh`. Or skip chmod and run it via `bash script.sh` directly.

### `set -euo pipefail` — Breakdown

| Flag | Meaning |
|---|---|
| `-e` | Stop the script immediately if any command fails |
| `-u` | Stop if an undefined variable is used |
| `-o pipefail` | Checks every part of a pipeline (`\|`), not just the last command |

---

## 6️⃣ Command Substitution

```bash
$(command)      # Modern, correct syntax — run a command and capture its output
`command`       # Legacy syntax — same thing, but deprecated. Don't use it.
```

> ⚠️ **Danger: Command Injection**
> Any variable coming from user input, or the result of another command, must be wrapped in quotes (`"$var"`) before being used in another command — otherwise, an attacker could inject additional commands using `;` or `|`.

---

## 7️⃣ Conditionals (Deep Dive)

```bash
if [[ -d /etc ]]; then
    ...
elif [[ condition ]]; then
    ...
else
    ...
fi
```

### File Tests

| Test | Meaning |
|---|---|
| `-d` | directory |
| `-f` | file |
| `-e` | exists |
| `-x` | executable |
| `-r` | readable |
| `-w` | writable |
| `-s` | not empty |

### Numeric Comparisons
```text
-eq -ne -gt -ge -lt -le
```

### String Comparisons
```text
== != < >
```
> ⚠️ **Don't confuse them** — string comparison is character-by-character, meaning `"10" < "9"` returns true when compared as strings.

`elif` chains are evaluated top-to-bottom — the first true condition wins, so order matters.

### case Statement

```bash
case $var in
    pattern1|pattern2) commands ;;
    *) default_commands ;;
esac
```
`case` uses pattern matching (like `*.jpg|*.png`, `5*`), not numeric comparison.

---

## 8️⃣ Loops (Deep Dive)

```bash
for i in 1 2 3; do ... done                    # list
for i in $(seq 1 10); do ... done              # sequence
for (( i=0; i<10; i++ )); do ... done          # arithmetic (C-style)
for file in *.sh; do ... done                  # wildcard/globbing
while [[ condition ]]; do ... done             # indefinite
while IFS= read -r line; do ... done < file    # (most important pattern) reading a file line by line
```

* **Definite** = you know the number of iterations in advance.
* **Indefinite** = depends on a changing condition.

> ⚠️ The condition inside a `while` loop must change eventually, otherwise you get an infinite loop (risk of DoS if you're targeting a real server).

`*.sh` globbing is expanded by Bash using filenames, before the command even runs — it is not regex.

---

## 9️⃣ Special Variables

| Variable | Meaning |
|---|---|
| `$0` | Script's own name |
| `$1 $2 ...` | Arguments in order |
| `${10}+` | Requires braces for double-digit arguments |
| `$#` | Number of arguments |
| `"$@"` | All arguments, each treated separately (always use this in loops) |
| `"$*"` | All arguments as a single combined string |
| `$?` | Exit status of the last command (0 = success, anything else = failure) — critical, and rarely emphasized enough |

---

## 🔟 Advanced Processing Tools

```bash
wc -l < file                       # Count number of lines
openssl rand -hex N                # Cryptographically secure randomness — far better than $RANDOM
$((16#$hex))                       # Convert hex to decimal
%                                  # Modulus — constrains a random number within a specific range
awk -v var=x 'NR==x {print}' file  # Print a specific line by its number
${word^}                           # Capitalize first letter
${word^^}                          # ALL CAPS
${word,,}                          # all lowercase
```

---

## 1️⃣1️⃣ Missing Pieces You Need to Know

### Functions
```bash
func_name() {
    local x=$1
    ...
}
```

### Arrays
```bash
arr=(22 80 443)
for port in "${arr[@]}"; do
    ...
done
```
*Always use `"$@"` in quotes to avoid whitespace-related issues.*

---

## 🔐 Connecting the Dots (Security Perspective)

| Concept | Security Implication |
|---|---|
| `$()` used on unquoted user input | Instant command injection |
| SUID bash scripts you can modify | Privilege escalation |
| Using `$RANDOM` for tokens/passwords | Insecure randomness vulnerability |
| Infinite loop on an external target | Can trigger DoS, potentially getting you kicked out of a bug bounty program |
| A real-world recon script | shebang + `set -euo pipefail` + `while`/`read` + `curl` + `if` + `sleep` (rate limiting) |

---
*📘 This content was studied and compiled from TCM Security's Linux 101 course. All explanations above were written by me personally to reinforce my own understanding — if you spot any mistake, feel free to open an issue or correct me.*
