### 🏴‍☠️ [Level 27 &rarr; Level 28]

**1. 🎯 The Objective & The "Why"**
- **The Question:** There's a git repository at `ssh://bandit27-git@bandit.labs.overthewire.org/home/bandit27-git/repo` via port 2220. The password for `bandit27-git` is the same as for `bandit27`. From your LOCAL machine (not the OverTheWire server), clone the repository and find the password for the next level.
- **The Goal:** For the first time, the challenge explicitly demands you work from YOUR OWN computer, not from inside the Bandit SSH session. You need `git` installed locally, and you need to clone a remote repository over SSH, then examine its contents to find the password.
- **The Why:** This is your first real exposure to **Git**, the single most important version control tool in all of software development and, by extension, a MASSIVE part of security work. Source code repositories are goldmines for attackers (leaked credentials in commit history, exposed `.git` folders on web servers), and understanding basic Git operations is completely non-negotiable for any security professional, red or blue team.

---

### 2. 💻 The Execution (Step-by-Step)

This is important: STOP being SSH'd into the Bandit server for this part. Open a fresh terminal window on YOUR OWN local computer (your laptop/desktop, not the remote Bandit machine).

First, make sure `git` is actually installed on your machine:
```bash
git --version
```
If this returns a version number, you're good. If not, you'd need to install it first (`sudo apt install git` on Debian/Ubuntu, `brew install git` on Mac, or download it from git-scm.com on Windows) — but for our purposes, let's assume it's there.

Now, let's clone the repository:

```bash
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
```

Let's break this URL/command down piece by piece, because Git's SSH-based URLs look a bit different from a normal `ssh` command:
- `git clone`: This is Git's command for "download a complete copy of a remote repository (including its entire history) onto my local machine."
- `ssh://`: This tells Git to use the SSH PROTOCOL to reach the repository (as opposed to `https://`, which is the other common way Git repos are accessed, like on GitHub).
- `bandit27-git@`: The username we're authenticating as — notice it's `bandit27-git`, NOT `bandit27`. This is a DIFFERENT, special account specifically created for accessing Git repositories, though the level tells us its password is IDENTICAL to bandit27's regular password.
- `bandit.labs.overthewire.org`: The server hostname, same as always.
- `:2220`: Here, the PORT is specified using a COLON directly after the hostname (this is standard Git URL syntax) — different from the `-p 2220` flag syntax we use with plain `ssh`.
- `/home/bandit27-git/repo`: The exact PATH to the repository on the remote server.

When prompted, enter the bandit27 password (which doubles as bandit27-git's password). Git will download the repository into a new folder on YOUR local machine, typically named `repo` (matching the repository's name).

Now move into that newly-created folder:
```bash
cd repo
```

Let's see what's inside:
```bash
ls -la
```
You'll likely find a file, maybe `README.md` or similar. Let's read it:
```bash
cat README.md
```
(Adjust the filename based on whatever you actually find.) The level 28 password will be sitting right there in the file content, since the repository owner (carelessly, on purpose for this challenge) committed the password directly into a tracked file.

---

### 3. 🌍 Real-World & Tactical Application

Cloning Git repositories and hunting through their contents is an everyday activity in both legitimate development work and offensive security.

- **🔴 Red Team Perspective:** Exposed Git repositories are a GOLDMINE for attackers. If a `.git` folder is accidentally left accessible on a public web server (a shockingly common misconfiguration), tools like `git-dumper` or manually crafted `wget`/`curl` requests can reconstruct the entire repository, revealing full commit history — including secrets that were added and later "deleted" in a subsequent commit (spoiler: deleting a file in a NEW commit does NOT remove it from Git's HISTORY; it's still fully recoverable). This is precisely why "hardcoded credentials accidentally committed to Git" remains one of the most common and damaging real-world security incidents.
- **🔵 Blue Team Perspective:** Organizations MUST use tools like `git-secrets`, `gitleaks`, or pre-commit hooks that scan for credential patterns BEFORE code ever gets committed, and MUST ensure `.git` directories are never exposed on public-facing web servers (a simple web server config check). If a secret DOES get committed by accident, simply deleting it in a new commit is NOT sufficient remediation — the secret must be considered compromised, ROTATED (changed) immediately, and ideally the git history itself should be scrubbed using tools like `git filter-repo` or BFG Repo-Cleaner.

---

### 4. 🚦 The Filter (Focus vs. Ignore)

- **FOCUS ON:** The `git clone ssh://user@host:port/path` syntax, and critically, remembering to run this from your OWN local machine, not the remote server. Also lock in that Git repos need to be explored just like any filesystem (`ls`, `cat`) once cloned — the "git-ness" of it doesn't change basic file navigation.
- **IGNORE:** Don't worry about deeper Git concepts yet (branches, commits, `git log`, `git diff`) — those become essential in the NEXT level. For this specific level, `git clone` plus basic file reading is genuinely all you need.

---

### 5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)

>  ، اللفل ده هيعرفك على Git، أهم أداة في عالم البرمجة كله، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** فيه مستودع git على `ssh://bandit27-git@bandit.labs.overthewire.org/home/bandit27-git/repo` عن طريق بورت 2220. باسورد يوزر `bandit27-git` هو نفسه باسورد `bandit27`. من جهازك **المحلي** (مش سيرفر OverTheWire)، استنسخ المستودع ولاقي باسورد اللفل الجاي.
> - **الهدف:** لأول مرة، التحدي بيطلب منك صراحة تشتغل من **جهازك انت**، مش من جوه جلسة SSH بتاعة Bandit. لازم يكون عندك `git` مثبت محلياً، ولازم تستنسخ مستودع بعيد عن طريق SSH، وبعدين تفحص محتواه عشان تلاقي الباسورد.
> - **الليه؟** ده أول تعامل حقيقي ليك مع **Git**، أهم أداة تحكم بالإصدارات في عالم تطوير البرمجيات كله، وبالتبعية، جزء **ضخم** من شغل الأمن. مستودعات الكود المصدري كنوز للهاكرز (باسوردات مسربة في تاريخ الـ commits، فولدرات `.git` مكشوفة على سيرفرات الويب)، وفهم عمليات Git الأساسية إجباري تماماً لأي محترف أمن، سواء Red أو Blue Team.
>
> **2. 💻 التنفيذ خطوة بخطوة**
>
> مهم جداً: **بطل** تكون عامل SSH لسيرفر Bandit في الجزء ده. افتح نافذة تيرمينال جديدة على **جهازك انت** (اللابتوب أو الديسكتوب بتاعك، مش جهاز Bandit البعيد).
>
> الأول، تأكد إن `git` فعلاً مثبت على جهازك:
> ```bash
> git --version
> ```
> لو ده رجعلك رقم إصدار، تمام. لو لأ، محتاج تثبته الأول (`sudo apt install git` على Debian/Ubuntu، `brew install git` على Mac، أو تنزله من git-scm.com على ويندوز)—بس لغرضنا، خلينا نفترض إنه موجود.
>
> دلوقتي، يلا نستنسخ المستودع:
>
> ```bash
> git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
> ```
>
> خليني افكك الرابط/الأمر ده حتة حتة، لإن روابط Git المعتمدة على SSH شكلها مختلف شوية عن أمر `ssh` العادي:
> - `git clone`: ده أمر Git الخاص بـ "نزّل نسخة كاملة من مستودع بعيد (بتاريخه كله) على جهازي المحلي."
> - `ssh://`: ده بيقول لـ Git يستخدم **بروتوكول SSH** عشان يوصل للمستودع (على عكس `https://`، وهي الطريقة الشائعة التانية اللي مستودعات Git بيتم الوصول ليها بيها، زي على GitHub).
> - `bandit27-git@`: اليوزر نيم اللي بنصادق بيه—لاحظ إنه `bandit27-git`، **مش** `bandit27`. ده حساب **مختلف وخاص** اتعمل بالتحديد عشان الوصول لمستودعات Git، رغم إن اللفل بيقولنا باسورده **مطابق تماماً** لباسورد bandit27 العادي.
> - `bandit.labs.overthewire.org`: اسم مضيف السيرفر، زي دايماً.
> - `:2220`: هنا، **البورت** متحدد باستخدام نقطتين مباشرة بعد اسم المضيف (دي صيغة قياسية لروابط Git)—مختلفة عن صيغة الفلاج `-p 2220` اللي بنستخدمها مع `ssh` العادية.
> - `/home/bandit27-git/repo`: المسار بالظبط للمستودع على السيرفر البعيد.
>
> لما يطلب منك، اكتب باسورد bandit27 (اللي بيشتغل كمان كباسورد bandit27-git). Git هينزل المستودع في فولدر جديد على جهازك **المحلي**، غالباً باسم `repo` (مطابق لاسم المستودع).
>
> دلوقتي ادخل الفولدر الجديد ده:
> ```bash
> cd repo
> ```
>
> يلا نشوف اللي جواه:
> ```bash
> ls -la
> ```
> غالباً هتلاقي ملف، ممكن يكون `README.md` أو شبهه. يلا نقراه:
> ```bash
> cat README.md
> ```
> (عدّل اسم الملف حسب اللي فعلاً هتلاقيه.) باسورد اللفل 28 هيكون موجود قدامك في محتوى الملف، لإن مالك المستودع (باستهتار، عمداً للتحدي ده) عمل commit للباسورد مباشرة في ملف متتبع.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
>
> استنساخ مستودعات Git والبحث في محتواها نشاط يومي في شغل التطوير الشرعي والأمن الهجومي على حد سواء.
>
> - **🔴 من منظور الـ Red Team:** مستودعات Git المكشوفة كنز حقيقي للهاكرز. لو فولدر `.git` اتسرب بالغلط ومتاح على سيرفر ويب عام (خطأ إعدادات شائع بشكل مرعب)، أدوات زي `git-dumper` أو طلبات `wget`/`curl` مصممة يدوياً تقدر تعيد بناء المستودع بالكامل، وتكشف تاريخ الـ commits كامل—بما فيهم أسرار اتضافت وبعدين "اتمسحت" في commit لاحق (سبويلر: مسح ملف في commit **جديد** مش بيشيله من **تاريخ** Git؛ لسه ممكن استرجاعه بالكامل). عشان كده بالظبط "باسوردات مكتوبة ثابتة اتعمللها commit بالغلط في Git" لسه من أكتر الحوادث الأمنية الحقيقية شيوعاً وضرراً.
> - **🔵 من منظور الـ Blue Team:** المؤسسات لازم تستخدم أدوات زي `git-secrets` أو `gitleaks` أو pre-commit hooks بتفحص على أنماط باسوردات **قبل** ما الكود يتعمله commit أصلاً، ولازم تتأكد إن فولدرات `.git` أبداً متكونش مكشوفة على سيرفرات ويب عامة (فحص إعدادات سيرفر ويب بسيط). لو سر فعلاً اتعمله commit بالغلط، مجرد مسحه في commit جديد **مش كافي** كإصلاح—السر لازم يتعتبر مخترق، ويتغير (rotation) فوراً، ويفضل تاريخ الـ git نفسه يتنضف باستخدام أدوات زي `git filter-repo` أو BFG Repo-Cleaner.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** صيغة `git clone ssh://user@host:port/path`، والأهم، افتكر إنك تشغل ده من **جهازك انت**، مش السيرفر البعيد. وكمان ثبّت إن مستودعات Git لازم تُستكشف زي أي نظام ملفات عادي (`ls` و `cat`) بمجرد ما تتستنسخ—"طبيعتها كـ Git" ما بتغيرش التنقل الأساسي في الملفات.
> - **تجاهل:** متقلقش من مفاهيم Git الأعمق دلوقتي (فروع، commits، `git log`، `git diff`)—دول هيبقوا أساسيين في اللفل **الجاي**. للفل ده بالتحديد، `git clone` مع قراءة ملفات أساسية هي فعلياً كل اللي محتاجه.
