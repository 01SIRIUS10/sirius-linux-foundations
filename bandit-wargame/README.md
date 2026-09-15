# 🏴 OverTheWire — Bandit Wargame

Bandit isn't a course. It's 34 levels of "figure it out yourself" —
no walkthroughs were used to solve any of these.

**Connect:**

```bash
ssh banditN@bandit.labs.overthewire.org -p 2220
```

Rule I followed: only man, --help, ExplainShell, and trial and error.

📊 Progress: 34 / 34 levels solved 

### 📊 Status: 34/34 Levels Solved | 📝 Write-ups Documentation in Progress

| Level | Concept Tested | Status | Write-up |
| :---: | :--- | :---: | :---: |
| **00 &rarr; 01** | SSH connection basics | 🟢 | [View Write-up](./Level_00.md) |
| **01 &rarr; 02** | Handling a filename that looks like a flag (`-`) | 🟢 | [View Write-up](./Level_01.md) |
| **02 &rarr; 03** | Filenames containing spaces | 🟢 | [View Write-up](./Level_02.md) |
| **03 &rarr; 04** | Finding hidden files (`ls -a`) | 🟢 | [View Write-up](./Level_03.md) |
| **04 &rarr; 05** | Identifying the human-readable file among many (`file`) | 🟢 | [View Write-up](./Level_04.md) |
| **05 &rarr; 06** | Searching by file size and permissions (`find`) | 🟢 | [View Write-up](./Level_05.md) |
| **06 &rarr; 07** | Searching the whole filesystem by owner/size, silencing errors | 🟢 | [View Write-up](./Level_06.md) |
| **07 &rarr; 08** | Searching inside a file for a value (`grep`) | 🟢 | [View Write-up](./Level_07.md) |
| **08 &rarr; 09** | Finding the one unique line in a large file (`sort`, `uniq`) | 🟢 | [View Write-up](./Level_08.md) |
| **09 &rarr; 10** | Extracting readable text from a data file (`strings`, `grep`) | 🟢 | [View Write-up](./Level_09.md) |
| **10 &rarr; 11** | Decoding Base64 | 🟢 | [View Write-up](./Level_10.md) |
| **11 &rarr; 12** | Decoding ROT13 (`tr`) | 🟢 | [View Write-up](./Level_11.md) |
| **12 &rarr; 13** | Reversing multiple layers of compression/encoding | 🟢 | [View Write-up](./Level_12.md) |
| **13 &rarr; 14** | Authenticating with an SSH private key | 🟢 | [View Write-up](./Level_13.md) |
| **14 &rarr; 15** | Submitting data to a local port (`nc`) | 🟢 | [View Write-up](./Level_14.md) |
| **15 &rarr; 16** | Connecting to an SSL service (`openssl s_client`) | 🟢 | [View Write-up](./Level_15.md) |
| **16 &rarr; 17** | Scanning a port range to find the right service (`nmap`) | 🟢 | [View Write-up](./Level_16.md) |
| **17 &rarr; 18** | Comparing two files to spot a change (`diff`) | 🟢 | [View Write-up](./Level_17.md) |
| **18 &rarr; 19** | Bypassing a `.bashrc` that logs you out instantly | 🟢 | [View Write-up](./Level_18.md) |
| **19 &rarr; 20** | Exploiting a SUID binary to read a protected file | 🟢 | [View Write-up](./Level_19.md) |
| **20 &rarr; 21** | Exploiting a SUID binary that connects back to your own listener | 🟢 | [View Write-up](./Level_20.md) |
| **21 &rarr; 22** | Reading a cron job running as another user | 🟢 | [View Write-up](./Level_21.md) |
| **22 &rarr; 23** | Exploiting a world-writable script run by cron | 🟢 | [View Write-up](./Level_22.md) |
| **23 &rarr; 24** | Exploiting a predictable temp file created by a cron script | 🟢 | [View Write-up](./Level_23.md) |
| **24 &rarr; 25** | Brute-forcing a 4-digit PIN with a loop script | 🟢 | [View Write-up](./Level_24.md) |
| **25 &rarr; 26** | Escaping a restricted shell via a pager (`more`) | 🟢 | [View Write-up](./Level_25.md) |
| **26 &rarr; 27** | Escaping a restricted shell tied to an SSH key | 🟢 | [View Write-up](./Level_26.md) |
| **27 &rarr; 28** | Finding a password inside a git repository | 🟢 | [View Write-up](./Level_27.md) |
| **28 &rarr; 29** | Recovering a password from git commit history | 🟢 | [View Write-up](./Level_28.md) |
| **29 &rarr; 30** | Finding a password in a different git branch | 🟢 | [View Write-up](./Level_29.md) |
| **30 &rarr; 31** | Finding a password in a git tag | 🟢 | [View Write-up](./Level_30.md) |
| **31 &rarr; 32** | Making a specific git commit to receive the next password | 🟢 | [View Write-up](./Level_31.md) |
| **32 &rarr; 33** | Escaping an "everything becomes uppercase" shell using `$0` | 🟢 | [View Write-up](./Level_32.md) |
| **33 &rarr; 34** | Using an allowed `sudo` command to bypass a restriction | 🟢 | [View Write-up](./Level_33.md) |
