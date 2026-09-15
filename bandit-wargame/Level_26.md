### 🏴‍☠️ [Level 25 &rarr; Level 26]

**1. 🎯 The Objective & The "Why"**
- **The Question:** Logging into `bandit26` from `bandit25` should be fairly easy... The shell for user `bandit26` is not `/bin/bash`, but something else. Find out what it is, how it works, and how to break out of it.
- **The Goal:** You've got the password for `bandit26`, but when you try to log in normally, you're not dropped into a familiar bash prompt — instead, you're placed into a RESTRICTED environment (a different program entirely acting as your "shell"). Your job is to figure out WHAT that program is, and exploit its behavior to break out into an actual usable bash shell.
- **The Why:** This introduces you to the concept of **restricted shells / custom login shells**, and more importantly, **shell breakout techniques** — a category of exploitation that's ENORMOUSLY relevant in real-world engagements involving jump boxes, restricted SSH environments, or "rbash" (restricted bash) setups that organizations use to limit what users can do. Breaking out of a restricted environment is a classic CTF and real-world red team skill.

---

### 2. 💻 The Execution (Step-by-Step)

You're logged in as `bandit25`. First, let's grab bandit26's password (which you should have obtained from the previous level's daemon):

Before attempting to log in, let's investigate a bit. We can peek at what shell bandit26 is configured to use by checking the system's user database:

```bash
cat /etc/passwd | grep bandit26
```
- `/etc/passwd`: This is the system file that lists every user account on the machine, along with metadata about each — including, critically, WHAT PROGRAM should be launched as their "shell" the moment they log in.
- `grep bandit26`: Filters that massive list down to just the line describing bandit26.

You'll see something like:
```
bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext
```
That LAST field, `/usr/bin/showtext`, is the key detail — instead of the normal `/bin/bash`, bandit26's designated "shell" is this custom program called `showtext`. Whatever THIS program does, it's what runs the INSTANT you successfully authenticate as bandit26.

Let's peek at what this program actually does:
```bash
cat /usr/bin/showtext
```
You'll see it's a shell script, something like:
```bash
#!/bin/sh
export TERM=linux
more ~/text.txt
exit 0
```
This tells us EXACTLY what happens: the moment you log in as bandit26, this script runs `more` on a text file, and then immediately calls `exit 0` — kicking you straight back out. This is why "logging in normally" doesn't give you a usable shell; you get a brief glimpse of a text file through `more`, and then you're forcibly disconnected.

Here's our opportunity: `more` is a PAGER program (used for viewing long text files page-by-page), and pagers like `more` and `less` have a well-known, classic feature/quirk — you can often execute shell commands FROM WITHIN them, using specific keyboard shortcuts, BEFORE the pager exits. This is our breakout vector.

**The trick:** we need our terminal window to be SMALL ENOUGH that the text file being displayed by `more` doesn't fit on one screen — because `more` only gives you interactive control (letting you press keys) if there's MORE content to scroll through (hence the name). If the whole file fits on your screen at once, `more` just dumps it all and exits immediately, giving you no chance to interact.

**Step 1: Resize your terminal window to be very small (fewer rows).**
Physically shrink your terminal window (drag the window border smaller, or reduce font size) so it only shows a few lines of text at a time. This is a manual, visual step you do on your OWN terminal application before connecting.

**Step 2: Log in as bandit26.**
```bash
ssh bandit26@bandit.labs.overthewire.org -p 2220
```
Enter the bandit26 password. Since your terminal is now small, `more` will only show you the FIRST portion of the text file and then PAUSE, waiting for you to press a key to see more (instead of dumping everything and exiting instantly).

**Step 3: While `more` is paused, use its command-execution feature.**
While `more` is sitting there waiting for input, press:
```
!bash
```
- The `!` character, inside `more` (and `less`), is a classic feature meaning "execute the following as a shell command." Typing `!bash` tells `more` to spawn an actual bash shell process.

Hit Enter after typing `!bash`, and you'll drop directly into a real, interactive bash shell — running with bandit26's privileges, completely bypassing the restrictive `showtext` program that was designed to kick you out. From here, you can navigate normally with `ls`, `cat`, etc., to find the level 26/27 flag (this specific level chain often continues directly into privilege escalation via a SUID binary you'll find in bandit26's home directory, setting up the NEXT level's challenge).

**Note on Windows users:** As the level warns, PowerShell's SSH client sometimes handles terminal sizing/signaling differently and can interfere with this technique — using Command Prompt (cmd.exe) or a proper terminal emulator instead avoids these quirks.

---

### 3. 🌍 Real-World & Tactical Application

Breaking out of restricted shells and pagers is a certified, real-world red team and CTF classic.

- **🔴 Red Team Perspective:** Organizations sometimes deploy "jump boxes" or restricted accounts using tools like `rbash` (restricted bash) or custom wrapper scripts specifically to limit what a user/service account can do. Attackers who land on such a restricted shell IMMEDIATELY try known breakout techniques — spawning subshells through text editors (`vi` has `:!bash`), pagers (`more`/`less` have `!bash`), or other trusted binaries that have shell-escape functionality (this is literally what GTFOBins.github.io catalogs extensively). Mastering this specific `!bash` trick in `more`/`less` is directly transferable to real penetration tests against poorly-hardened restricted environments.
- **🔵 Blue Team Perspective:** If an organization genuinely needs to restrict a user to a specific limited toolset, they must be EXTREMELY careful about which programs are allowed to run within that restricted shell — ANY program with a known shell-escape feature (text editors, pagers, even some network tools) completely defeats the purpose of the restriction. Proper security hardening involves disabling shell-escape features in allowed tools, or using more robust sandboxing/containerization technology instead of relying purely on a restricted shell wrapper, which is a notoriously leaky and hard-to-fully-secure approach.

---

### 4. 🚦 The Filter (Focus vs. Ignore)

- **FOCUS ON:** Checking `/etc/passwd` to discover a user's configured login shell, reading that shell/script to understand its behavior, and recognizing `more`/`less`'s `!command` shell-escape feature as a classic breakout technique. This "read the restriction's code, then find its escape hatch" mindset applies to nearly every restricted environment you'll ever encounter.
- **IGNORE:** Don't worry about memorizing every possible shell-breakout technique for every possible restricted program right now (vi, awk, python, etc. all have their own escape methods) — that's a growing mental checklist you build over your career (GTFOBins is your future best friend for this). For this level, `!bash` inside `more` is your one specific target.

---

### 5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)

> ، اللفل ده هيعلمك إزاي تكسر بيئة مقيدة، مهارة كلاسيكية في الـ Red Teaming الحقيقي، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** الدخول لـ `bandit26` من `bandit25` المفروض يكون سهل... بس الشِل بتاع يوزر `bandit26` مش `/bin/bash`، ده حاجة تانية. لازم تكتشف هي إيه، وبتشتغل إزاي، وإزاي تكسر منها.
> - **الهدف:** عندك باسورد `bandit26`، بس لما تحاول تعمل login عادي، مش هتتحط في برومبت باش مألوف—هتتحط في بيئة **مقيدة** (برنامج مختلف تماماً بيعمل دور "الشِل" بتاعك). مهمتك إنك تعرف البرنامج ده إيه، وتستغل سلوكه عشان تكسر منه لشِل باش حقيقي قابل للاستخدام.
> - **الليه؟** ده بيعرفك على مفهوم **الشِلات المقيدة/شِلات تسجيل الدخول المخصصة**، والأهم من كده، **تقنيات الكسر من الشِل (breakout)**—فئة من الاستغلال مهمة جداً في العمليات الحقيقية اللي فيها jump boxes أو بيئات SSH مقيدة أو إعدادات "rbash" (باش مقيد) اللي شركات بتستخدمها عشان تحدد إيه اليوزر يقدر يعمله. الكسر من بيئة مقيدة مهارة كلاسيكية في الـ CTF والـ Red Team الحقيقي.
>
> **2. 💻 التنفيذ خطوة بخطوة**
>
> انت عامل login كـ `bandit25`. الأول، يلا ناخد باسورد bandit26 (المفروض تكون جبته من خدمة اللفل اللي فات):
>
> قبل ما نحاول نعمل login، يلا نحقق شوية. نقدر نشوف أنهي شِل bandit26 متظبط يستخدم عن طريق فحص قاعدة بيانات يوزرز النظام:
>
> ```bash
> cat /etc/passwd | grep bandit26
> ```
> - `/etc/passwd`: ده ملف النظام اللي بيسرد كل حسابات اليوزرز في الجهاز، مع بيانات ميتا عن كل واحد—وفيهم بالتحديد، **أنهي برنامج** المفروض يشتغل كـ "شِل" ليه لحظة ما يعمل login.
> - `grep bandit26`: بيفلتر القايمة الضخمة دي عشان يوريك بس السطر اللي بيوصف bandit26.
>
> هتشوف حاجة زي:
> ```
> bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext
> ```
> الحقل **الأخير** ده، `/usr/bin/showtext`، هو التفصيلة الأساسية—بدل الـ `/bin/bash` العادي، الشِل المحدد لـ bandit26 هو البرنامج المخصص ده اسمه `showtext`. أي حاجة البرنامج ده بيعملها، هي اللي بتشتغل **لحظة** ما تسجل دخولك كـ bandit26 بنجاح.
>
> يلا نشوف البرنامج ده بيعمل إيه فعلياً:
> ```bash
> cat /usr/bin/showtext
> ```
> هتلاقيه سكريبت شِل، حاجة زي:
> ```bash
> #!/bin/sh
> export TERM=linux
> more ~/text.txt
> exit 0
> ```
> ده بيقولنا **بالظبط** إيه اللي بيحصل: لحظة ما تعمل login كـ bandit26، السكريبت ده بيشغل `more` على ملف نصي، وبعدين على طول بينادي `exit 0`—وبيطردك برة على طول. عشان كده بالظبط "الـ login العادي" مش بيديك شِل قابل للاستخدام؛ بتاخد لمحة سريعة من ملف نصي عن طريق `more`، وبعدين بتتقطع بالإجبار.
>
> هنا فرصتنا: `more` هو برنامج **pager** (بيتستخدم لعرض ملفات نصية طويلة صفحة صفحة)، والـ pagers زي `more` و `less` عندهم خاصية/كويرك معروفة كلاسيكية—تقدر غالباً تنفذ أوامر شِل **من جواهم**، باستخدام اختصارات كيبورد معينة، **قبل** ما البرنامج يخرج. دي بوابة الكسر بتاعتنا.
>
> **الحيلة:** محتاجين نافذة التيرمينال بتاعتنا تكون **صغيرة بما فيه الكفاية** بحيث الملف النصي المعروض بواسطة `more` مبيتلمش في شاشة واحدة—لإن `more` بيديك تحكم تفاعلي (يسمحلك تدوس أزرار) بس لو فيه **محتوى أكتر** تسكرول عليه (عشان كده اسمه more). لو الملف كله لاقي مكانه في شاشتك مرة واحدة، `more` بس بيطبع كل حاجة ويخرج فوراً، ومبيديكش أي فرصة تتفاعل.
>
> **الخطوة 1: صغّر نافذة التيرمينال بتاعتك عشان تبقى صغيرة جداً (سطور أقل).**
>
> صغّر فعلياً نافذة التيرمينال (اسحب حدود النافذة أصغر، أو قلل حجم الخط) عشان تعرض بس سطور قليلة من النص في المرة الواحدة. دي خطوة يدوية وبصرية بتعملها على برنامج التيرمينال بتاعك انت قبل ما تتصل.
>
> **الخطوة 2: اعمل login كـ bandit26.**
> ```bash
> ssh bandit26@bandit.labs.overthewire.org -p 2220
> ```
> اكتب باسورد bandit26. بما إن التيرمينال بتاعك دلوقتي صغير، `more` هتوريك بس الجزء **الأول** من الملف النصي وبعدين **تتوقف**، مستنية إنك تدوس زرار عشان تشوف أكتر (بدل ما تطبع كل حاجة وتخرج فوراً).
>
> **الخطوة 3: وانت `more` واقفة مستنية، استخدم خاصية تنفيذ الأوامر بتاعتها.**
>
> وانت `more` قاعدة مستنية إدخال، اكتب:
> ```
> !bash
> ```
> - الحرف `!` جوه `more` (و `less`) خاصية كلاسيكية معناها "نفذ اللي بعدي كأمر شِل." كتابة `!bash` بتقول لـ `more` تفتح عملية شِل باش حقيقية.
>
> دوس Enter بعد ما تكتب `!bash`، وهتنزل مباشرة في شِل باش حقيقي وتفاعلي—شغال بصلاحيات bandit26، متخطي تماماً برنامج `showtext` المقيد اللي كان مصمم يطردك. من هنا، تقدر تتنقل عادي بـ `ls` و `cat` وغيرهم عشان تلاقي علم اللفل 26/27 (سلسلة اللفلات دي غالباً بتكمل مباشرة لتصعيد صلاحيات عن طريق برنامج SUID هتلاقيه في فولدر bandit26، وده هيجهز لتحدي اللفل الجاي).
>
> **ملاحظة ليوزرز الويندوز:** زي ما اللفل بيحذر، كلاينت SSH بتاع PowerShell أحياناً بيتعامل مع حجم/إشارات التيرمينال بشكل مختلف وممكن يعطل التقنية دي—استخدام Command Prompt (cmd.exe) أو محاكي تيرمينال حقيقي بدلاً منه بيتجنب الكويرك ده.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
>
> الكسر من الشِلات المقيدة والـ pagers مهارة كلاسيكية موثقة في الـ Red Team والـ CTF الحقيقي.
>
> - **🔴 من منظور الـ Red Team:** المؤسسات أحياناً بتنشر "jump boxes" أو حسابات مقيدة باستخدام أدوات زي `rbash` (باش مقيد) أو سكريبتات wrapper مخصصة بالتحديد عشان تحدد إيه اليوزر/حساب الخدمة يقدر يعمله. المهاجمين اللي بيوصلوا لشِل مقيد زي ده بيجربوا **فوراً** تقنيات الكسر المعروفة—فتح subshells عن طريق محررات نصوص (`vi` عنده `:!bash`)، أو pagers (`more`/`less` عندهم `!bash`)، أو برامج موثوقة تانية عندها خاصية shell-escape (ده حرفياً اللي موقع GTFOBins.github.io بيوثقه بكثرة). إتقان حيلة `!bash` المحددة دي جوه `more`/`less` قابلة للتطبيق مباشرة في اختبارات اختراق حقيقية ضد بيئات مقيدة مش محصنة كويس.
> - **🔵 من منظور الـ Blue Team:** لو مؤسسة فعلاً محتاجة تقيد يوزر لمجموعة أدوات محدودة، لازم تكون **حذرة جداً جداً** بخصوص أنهي برامج مسموح تشتغل جوه الشِل المقيد ده—**أي** برنامج عنده خاصية shell-escape معروفة (محررات نصوص، pagers، حتى بعض أدوات الشبكة) بيهزم الغرض من التقييد بالكامل. التحصين الأمني الصحيح بيتضمن تعطيل خصائص shell-escape في الأدوات المسموحة، أو استخدام تقنية sandboxing/containerization أقوى بدل الاعتماد بشكل كامل على wrapper شِل مقيد، وده أسلوب معروف إنه مليان ثغرات وصعب تأمينه بالكامل.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** فحص `/etc/passwd` عشان تكتشف الشِل المحدد لتسجيل دخول يوزر معين، قراءة الشِل/السكريبت ده عشان تفهم سلوكه، والتعرف على خاصية `!command` shell-escape بتاعة `more`/`less` كتقنية كسر كلاسيكية. عقلية "اقرا كود التقييد، بعدين لاقي بوابة الهروب بتاعته" دي بتنطبق على تقريباً أي بيئة مقيدة هتقابلها في حياتك.
> - **تجاهل:** متقلقش من حفظ كل تقنية كسر ممكنة لكل برنامج مقيد ممكن دلوقتي (`vi` و `awk` و `python` وغيرهم كل واحد عنده طريقة هروب خاصة بيه)—دي قائمة ذهنية بتكبر مع مشوارك المهني (وGTFOBins هيبقى صاحبك المفضل في الموضوع ده). للفل ده، `!bash` جوه `more` هي هدفك المحدد الوحيد.
