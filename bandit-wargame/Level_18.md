### 🏴‍☠️ [Level 17 &rarr; Level 18]

**1. 🎯 The Objective & The "Why"**
- **The Question:** There are 2 files in the home directory: `passwords.old` and `passwords.new`. The password for the next level is in `passwords.new` and is the only line that has been changed between `passwords.old` and `passwords.new`.
- **The Goal:** You've got two nearly-identical files, each probably containing hundreds of lines of random-looking password candidates. Somewhere in there, EXACTLY one line is different between the two versions — that difference IS your password. You need to compare both files and isolate that single changed line, instead of eyeballing hundreds of lines manually.
- **The Why:** This teaches you **file comparison / diffing** — one of THE most fundamental skills in software development, system administration, AND security auditing. Every time a config file changes, a piece of code gets modified, or a system file gets tampered with by an attacker, "diffing" is how you spot exactly what changed. This single skill underlies version control (Git), intrusion detection, and code review.

**2. 💻 The Execution (Step-by-Step)**
You're logged in as `bandit17`. Let's confirm both files exist:

```bash
ls -la
```
You'll see `passwords.old` and `passwords.new` sitting in your home directory. If you tried `cat`-ing both and scrolling through by eye, you'd probably lose your mind and your eyesight — these files likely have hundreds of lines each, and the differences could be subtle (maybe just one character different). We need a tool built specifically for spotting differences between two files.

Enter `diff`:

```bash
diff passwords.old passwords.new
```
- `diff`: This command's entire job is to **compare two files line-by-line and report exactly what's different between them.** If a line exists identically in both files, `diff` says NOTHING about it — it stays completely silent on matching content. It ONLY speaks up about lines that don't match.
- `passwords.old`: The FIRST file we're comparing (think of this as the "before" version).
- `passwords.new`: The SECOND file we're comparing (the "after" version).

Running this command will output something like:
```
< oldPasswordCandidateLine
---
> aBcDeFgHiJkLmNoPqRsTuVwXyZ12345
```

Let's decode this specific `diff` syntax, because it uses special symbols that mean something precise:
- The `<` symbol marks a line that exists in the FIRST file (`passwords.old`) but is DIFFERENT/missing in the second.
- The `---` is just a visual separator, meaning "here's the boundary between the old version and the new version of this specific difference."
- The `>` symbol marks the corresponding line in the SECOND file (`passwords.new`) that replaced it.

Since the level tells us the password is in `passwords.new`, you want the line marked with `>` — that's your level 18 password, sitting right there, isolated from the hundreds of unchanged lines that `diff` correctly ignored.

**Important note on the level's warning:** The level description mentions that if you see "Byebye!" when trying to SSH into `bandit18`, don't panic — that's actually EXPECTED behavior and directly related to the NEXT level's puzzle (bandit18's shell has been intentionally configured to kick you out immediately upon login). We'll tackle that specific challenge properly in the next level's write-up.

**3. 🌍 Real-World & Tactical Application**
`diff` is used every single day by literally every software developer and system administrator on the planet.

- **🔴 Red Team Perspective:** Attackers use file-diffing techniques during post-exploitation to detect if their tools or payloads have been tampered with, or to compare a system's current config against a known baseline to find WHAT an admin recently changed (which can reveal recently patched vulnerabilities, or conversely, newly introduced misconfigurations worth exploiting). It's also core to malware analysis — comparing a suspicious file against a known-clean version of the same software to pinpoint exactly what was injected/modified.
- **🔵 Blue Team Perspective:** `diff` (and its more powerful cousins, version control systems like Git built ENTIRELY around diffing) is fundamental to security operations — File Integrity Monitoring (FIM) tools essentially run `diff`-like comparisons constantly against critical system files (`/etc/passwd`, `/etc/shadow`, web server configs) to detect unauthorized changes instantly. If an attacker modifies a config file to add a backdoor, a properly configured FIM system diffs the current state against a known baseline and immediately alerts the security team.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** The `diff file1 file2` syntax, and understanding the `<` (old file) vs `>` (new file) output symbols. This is your permanent go-to whenever you need to spot "what changed" between two versions of anything.
- **IGNORE:** Don't worry about `diff`'s more advanced output formats right now (unified diff `-u`, context diff `-c`, side-by-side `-y`) — those get more relevant once you're working with Git or more complex file comparisons. Plain `diff` output is perfectly sufficient here.

---

> **5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)**
>
>  اللفل ده هيعلمك مهارة أساسية جداً في عالم البرمجة والأمن—المقارنة بين ملفين، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** فيه ملفين في الـ home directory: `passwords.old` و `passwords.new`. الباسورد بتاع اللفل الجاي موجود في `passwords.new` وهو السطر الوحيد اللي اتغير بين الملفين.
> - **الهدف:** عندك ملفين متطابقين تقريباً، كل واحد فيهم غالباً فيه مئات السطور من كلمات شبه باسوردات عشوائية. في مكان ما جواهم، سطر واحد بس مختلف بين النسختين—الاختلاف ده هو الباسورد بتاعك. لازم تقارن الملفين وتعزل السطر المختلف ده، بدل ما تبص بعينك على مئات السطور يدوياً.
> - **الليه؟** ده بيعلمك **مقارنة الملفات (diffing)**—واحدة من أهم المهارات الأساسية في تطوير البرمجيات وإدارة الأنظمة والتدقيق الأمني. كل مرة ملف إعدادات بيتغير، أو جزء من كود بيتعدل، أو ملف نظام بيتلاعب فيه هاكر، الـ "diffing" هو الطريقة اللي بتشوف بيها بالظبط اللي اتغير. المهارة دي وحدها أساس أنظمة التحكم بالإصدارات (Git)، وكشف التسلل، ومراجعة الكود.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit17`. يلا نتأكد إن الملفين موجودين:
>
> ```bash
> ls -la
> ```
> هتلاقي `passwords.old` و `passwords.new` موجودين في الـ home directory بتاعك. لو حاولت تعمل `cat` للاتنين وتعمل سكرول بعينك، غالباً هتتجنن وعينك هتتعب—الملفات دي غالباً فيها مئات السطور، والاختلافات ممكن تكون دقيقة جداً (ممكن حرف واحد بس مختلف). محتاجين أداة معمولة بالتحديد عشان تكتشف الاختلافات بين ملفين.
>
> يدخل `diff`:
>
> ```bash
> diff passwords.old passwords.new
> ```
> - `diff`: شغلانة الأمر ده بالكامل إنه **يقارن ملفين سطر بسطر ويقولك بالظبط الاختلاف بينهم.** لو سطر موجود متطابق في الملفين، الـ `diff` مش هيقول حاجة عنه—بيفضل ساكت تماماً على المحتوى المتطابق. بيتكلم بس على السطور اللي مش متطابقة.
> - `passwords.old`: الملف **الأول** اللي بنقارنه (فكر فيه كـ نسخة "قبل").
> - `passwords.new`: الملف **التاني** اللي بنقارنه (نسخة "بعد").
>
> تشغيل الأمر ده هيطلعلك حاجة زي:
> ```
> < oldPasswordCandidateLine
> ---
> > aBcDeFgHiJkLmNoPqRsTuVwXyZ12345
> ```
>
> خليني افك الرموز الخاصة دي بتاعة `diff`، لإنها بتعني حاجة محددة:
> - الرمز `<` بيعلّم على سطر موجود في الملف **الأول** (`passwords.old`) بس مختلف/مش موجود في الملف التاني.
> - الـ `---` مجرد فاصل بصري، معناها "هنا الحد بين النسخة القديمة والنسخة الجديدة من الاختلاف ده بالتحديد."
> - الرمز `>` بيعلّم على السطر المقابل في الملف **التاني** (`passwords.new`) اللي حل محل السطر القديم.
>
> بما إن اللفل بيقولنا إن الباسورد موجود في `passwords.new`، انت عايز السطر المعلم بـ `>`—ده باسورد اللفل 18 بتاعك، موجود قدامك، معزول عن مئات السطور اللي متغيرتش واللي `diff` بشكل صحيح تجاهلها.
>
> **ملاحظة مهمة على تحذير اللفل:** وصف اللفل بيقول لو ظهرلك "Byebye!" وانت بتحاول تعمل SSH لـ `bandit18`، متتخضش—ده فعلاً سلوك **متوقع** ومرتبط مباشرة بلغز اللفل **الجاي** (شِل bandit18 اتظبط عمداً عشان يطردك فوراً بمجرد ما تعمل login). هنتعامل مع التحدي المحدد ده بشكل صحيح في اللفل الجاي.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> `diff` بتتستخدم كل يوم من كل مبرمج وأدمن نظام على الكوكب حرفياً.
>
> - **🔴 من منظور الـ Red Team:** الهاكرز بيستخدموا تقنيات مقارنة الملفات أثناء ما بعد الاختراق عشان يكتشفوا لو أدواتهم أو حمولاتهم اتلاعب فيها، أو عشان يقارنوا إعدادات النظام الحالية بنسخة أساسية معروفة عشان يعرفوا الأدمن غيّر إيه مؤخراً (ده ممكن يكشف ثغرات اتصلحت حديثاً، أو بالعكس، إعدادات جديدة خاطئة تستاهل استغلالها). دي كمان أساس تحليل المالوير—مقارنة ملف مشبوه بنسخة نظيفة معروفة من نفس البرنامج عشان تحدد بالظبط اللي اتحقن/اتعدل.
> - **🔵 من منظور الـ Blue Team:** `diff` (وأقاربها الأقوى، أنظمة التحكم بالإصدارات زي Git المبنية بالكامل على فكرة المقارنة) أساسية في عمليات الأمن—أدوات مراقبة سلامة الملفات (FIM) أساساً بتعمل مقارنات شبه `diff` باستمرار على ملفات النظام الحرجة (`/etc/passwd` و `/etc/shadow` وإعدادات سيرفر الويب) عشان تكتشف أي تغييرات غير مصرح بها فوراً. لو هاكر عدّل ملف إعدادات عشان يضيف باكدور، نظام FIM مظبوط صح بيقارن الوضع الحالي بنسخة أساسية معروفة وبينبه فريق الأمن على طول.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** صيغة `diff file1 file2`، وفهم رموز المخرجات `<` (الملف القديم) مقابل `>` (الملف الجديد). دي أداتك الدائمة كل ما تحتاج تشوف "إيه اللي اتغير" بين نسختين من أي حاجة.
> - **تجاهل:** متقلقش من صيغ مخرجات `diff` الأكتر تقدماً دلوقتي (unified diff بـ `-u`، context diff بـ `-c`، side-by-side بـ `-y`)—دول بيبقوا أكتر أهمية لما تشتغل مع Git أو مقارنات ملفات أعقد. مخرجات `diff` العادية كافية تماماً هنا.
