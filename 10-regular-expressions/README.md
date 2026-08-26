 # 🎯 10 - Regular Expressions (Regex)

## Status: 🟢 Completed

---

## 💣 The Scenario That Starts Everything

Imagine you've compromised a server and found a file called `database.sql`, weighing in at **5GB**. You're certain there are passwords inside it — but those passwords are written in wildly different formats. If you search for the word `password`, you'll get a million results.

What you actually want to tell the computer is: *"Give me any line that starts with the word `pass`, followed by an equals sign, followed by any digits, and ending with a quotation mark."*

This descriptive language is called **Regex**.

### 🤔 What Would Happen If Regex Didn't Exist?

We'd be stuck reading logs and code **line by line**, and a hacker would burn out long before finding a single vulnerability.

### 📍 Where This Shows Up in a Real Pentester's Life

| Phase | Application |
|---|---|
| **🔍 Recon** | Pulling a website's JavaScript (Frontend) files and running Regex on them to extract **Hidden Endpoints** or **API Keys** that the developer forgot about. |
| **⛏️ Post-Exploitation (Data Hunting)** | Extracting Visa numbers, emails, or phone numbers from leaked database dumps. |

### 🧰 What If It Didn't Exist? Every Cybersecurity Tool Would Be Crippled

Think about it with me:

- **`Nmap`** uses regex to analyze **banners** and identify the service type.
- **`Burp Suite`** uses regex in **match/replace rules** and in **Intruder**.
- **`Snort`/`Suricata`** (IDS/IPS tools) — all their rules are built entirely on **pattern matching**, like regex.
- **`grep`, `sed`, `awk`** in Linux — your daily weapon in **forensics** investigations and extracting information from tool output.
- **WAF rules** and **SQLi filters** rely almost entirely on regex patterns like `' OR 1=1--`.

---

## 🧬 From the Roots: What Is Regex, Really?

Regex is a **"descriptive language."** You're describing the shape of a criminal to a police sketch artist so they can pick him out of a million faces.

### ⚙️ A Fundamental Truth You Must Understand Before Regex

To a computer, any text is nothing more than a stream of **bytes** (**ASCII** or **Unicode** — every character has a number). When you write regex, you're not literally "talking to letters" in the poetic sense — you're talking to a **pattern-matching engine**: a program inside the tool (`grep`, `sed`, `Python`, `PCRE library`...) that reads the text byte by byte, and tries to match it against the rule you wrote.

---

## 🔑 The Essential Symbols You Need to Know

### ⚓ Anchors — Extremely important for precise filtering

| Symbol | Meaning |
|---|---|
| `^` | Beginning of the line |
| `$` | End of the line |

### 🗂️ Classes

| Symbol | Meaning |
|---|---|
| `.` | Any character in existence |
| `\d` | Any digit |
| `\w` | Any letter or digit |

### 🔢 Quantifiers

| Symbol | Meaning |
|---|---|
| `*` | Zero or more (optional, can repeat) |
| `+` | One or more (must appear at least once) |
| `?` | Optional (0 or 1) |

---

## 🍰 The Greediness Problem

Regex engines were built by programmers to eat the **biggest possible slice of cake** that fits the description. If you tell it to search for anything between two brackets `<.*>` inside HTML code like this:

~~~html
<h1>Hello</h1>
~~~

Will it match `<h1>` correctly? No! It'll get greedy and grab `<h1>Hello</h1>` — swallowing the entire line from the first `<` all the way to the last `>` in the line!

✅ **The Fix**
We add a question mark `?` after the asterisk — `<.*?>` — to make it Lazy, stopping at the first closing bracket it encounters.

### 🎯 Only Matching — The Wizard's Trick
As Hackers, if we have a line with 50,000 characters and we want just one IP address, we don't want the entire line! This is where the magic flag comes in: `-o` with `grep` (short for Only matching).

### 🔬 Try It Now

~~~bash
echo "My target IP is 192.168.1.5 and it is vulnerable" | grep -E -o "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"
~~~

Expected result: it'll print `192.168.1.5` alone, on its own line, and throw everything else into the trash.

Why: the regex describes the shape of an IP address (numbers from 1 to 3 digits, repeated and separated by dots), and the `-o` flag tells it: "Just give me that exact match, and nothing else." (This exact command gets used every single day in Bug Bounty work.)

### 🎬 Real-World Scenario: Bug Bounty on target.com
You're doing Bug Bounty work on target.com.

You find a minified JavaScript file, packed with extremely complex code, with all of it crammed into one absurdly long line.

You suspect there are API Endpoints hidden inside this code that the developer forgot to remove, starting with `/api/v1/`.

You open your terminal and type:

~~~bash
curl -s https://target.com/main.js | grep -E -o "/api/v1/[a-zA-Z0-9_/?=-]+"
~~~

The result: `grep` dives into that million-character wall of text and hands you back a clean list of every hidden path the developer forgot about. You take these paths and use them to break into the site, earning yourself a bounty worth thousands of dollars.

---

## 🧾 Summary

| Flag | Purpose |
|---|---|
| `grep -E` | Activates the full power of Regex (Extended Regular Expressions). |
| `grep -o` | The most important flag — gives you only the required match, discarding the rest of the line. |

### 🏆 The Golden Symbols

| Symbol | Meaning |
|---|---|
| `^` | Start of the line |
| `$` | End of the line |
| `.` | Any character |
| `*` | Any quantity (even zero) |
| `+` | One or more |

🌐 **Website:** regex101.com — this is where we test any Regex before typing it into the terminal to verify it (this is our Debugger).

### 🐕 The Sniffer Dog Analogy
*"Regex is your sniffer dog. You don't need to teach it how it was born — you just hand it the scent of what you're hunting (IP, Password, Key), unleash it with `grep -E -o`, and it will dig your target out from among millions of lines and throw the rest of the trash outside."*

---

## 🖋️ The Final Tattoo: Regex in 10 Points

1. Regex describes the shape of the text, not its literal value. Instead of "search for 42," you're saying "search for a two-digit number that looks like this."

### 🔑 The Golden Rule You Must Never Forget

| Command | When to Use |
|---|---|
| `grep -E` | Standard search (ERE), no need to escape `+` `?` `{}` `\|` `()` |
| `grep -P` | Needed if you require `\d` `\w` `\s` or lookahead (PCRE) |
| `sed -E` | When you need to actually replace/modify the text |

> ⚠️ **Never use plain `grep` without `-E` or `-P`** — everything becomes a mess of manual escaping.

2. `^` has two different meanings depending on where it's placed:

| Position | Meaning |
|---|---|
| Inside brackets `[^abc]` | Negation (NOT these characters) |
| Outside `^abc` | Beginning of the line |
| `$` | End of the line (requires multi-line mode if you're working on a file with many lines) |

3. Quantifiers:

| Symbol | Meaning |
|---|---|
| `?` | Zero or one |
| `*` | Zero or more |
| `+` | One or more |
| `{m,n}` | From m to n |

4. Groups `()` in `sed` are numbered starting from 1, and you use them in replacements with `\1` `\2` `\3`. This is the foundation of any text reformatting operation (like converting date formats).

5. Regex is greedy by nature — it takes the largest possible match. If you want it "lazy" (taking the smallest possible match), add `?` after the quantifier: `.+?` instead of `.+`. This makes a huge difference when extracting repeated HTML tags or anything recurring.

6. `\d` won't work with `grep -E` — you must use either `[0-9]` or `grep -P`. This is the most common mistake every beginner runs into.

### 💼 Daily Application in Real Work

~~~bash
# Extracting IPs from logs
grep -oE "([0-9]{1,3}\.){3}[0-9]{1,3}"

# Filtering important subdomains
grep -E "dev|staging|test"

# Detecting SQLi/XSS in logs
grep -E "union|select|--|<script>"
~~~

### 🚨 A Genuine Real Security Risk
Complex regex with nested quantifiers can cause **ReDoS** (Regular expression Denial of Service — slowing down or crashing the server). This is actually tested for as a real vulnerability in bug bounty programs.

### ⚖️ A Final Word of Wisdom
Don't use regex for everything. If `cut` or `tr` is enough for the job, use them instead — they're faster and simpler.

---
*This content was studied and written in my own words based on the concepts covered in TCM Security's Linux 101 course. Any errors in explanation or terminology are entirely my own.*
