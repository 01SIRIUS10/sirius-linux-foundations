### 🏴‍☠️ [Level 3 &rarr; Level 4]

**1. 🎯 The Objective & The "Why"**
- **The Question:** The password for the next level is stored in a hidden file in the `inhere` directory.
- **The Goal:** Navigate into a subdirectory called `inhere`, then locate a HIDDEN file inside it (a file that doesn't show up with a plain `ls`), and read its content.
- **The Why:** Hidden files are one of the oldest tricks in the Linux book for stashing away configuration, credentials, or "stuff you don't want casual users to accidentally stumble on." Attackers exploit hidden files heavily (hidden `.ssh` folders, hidden malware persistence files), so learning to reveal them is a non-negotiable skill.

**2. 💻 The Execution (Step-by-Step)**
You're logged in as `bandit3`. Let's start by seeing what's in our home directory:

```bash
ls -la
```
You'll spot a directory called `inhere`. Let's move into it:

```bash
cd inhere
```
- `cd` stands for "change directory." It's literally you walking from one room (folder) into another. You give it the name of the folder you want to enter, and boom, you're now "standing inside" it.

Now, let's try a normal `ls` first, just to prove a point:

```bash
ls
```
You'll notice... nothing. Empty. Absolutely nothing shows up. You might ask, "Wait, is this folder actually empty?" NO — I'll tell you exactly why nothing shows: by default, `ls` deliberately HIDES any file or folder whose name starts with a dot `.` (this is a Linux convention going back decades — dot-prefixed names are considered "hidden/system" files, meant to declutter your default view). The file exists, `ls` is just choosing not to show it to you.

So now we bring out the flag that reveals everything:

```bash
ls -la
```
- `-a`: "all" — this is the flag that FORCES `ls` to show hidden files too (anything starting with `.`).
- `-l`: "long listing" — gives us details like permissions and size, which is helpful for spotting the file clearly and checking if it's even readable.

Now you'll see something like a file named `...Hiding-From-You` or similar (the exact hidden filename varies by challenge instance, but it'll clearly start with a dot).

Read it with `cat`:

```bash
cat .filename-you-found
```
Replace `.filename-you-found` with whatever the actual hidden filename was that you saw in the `ls -la` output. This dumps the password for level 4 straight onto your screen.

**3. 🌍 Real-World & Tactical Application**
Hidden files (dotfiles) are EVERYWHERE in real Linux systems — `.bashrc`, `.bash_history`, `.ssh/`, `.env` files for applications, `.git/` folders for version control.

- **🔴 Red Team Perspective:** After compromising a machine, attackers ALWAYS run `ls -la` in every directory they enter, specifically hunting for dotfiles. Why? Because `.bash_history` can reveal previously-typed commands (sometimes including plaintext passwords typed by mistake!), `.ssh/id_rsa` can be a private key granting access to OTHER servers, and `.env` files often contain database credentials and API keys in plaintext. Attackers also HIDE their own malicious files/scripts by naming them with a leading dot, so they blend in and don't show up in a lazy admin's casual `ls`.
- **🔵 Blue Team Perspective:** Never store credentials in dotfiles without encryption, and always monitor for unusual dotfiles being created in unexpected directories (like `/tmp/.hidden_backdoor`) — this is a classic persistence technique for malware. Security teams use File Integrity Monitoring (FIM) tools specifically to alert on new or modified hidden files.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** `ls -la` as your default reflex EVERY time you enter a new directory from now on. Never trust a plain `ls` alone again. Also lock in `cd` for directory navigation.
- **IGNORE:** `file`, `du`, and `find` are still overkill for this specific level since `ls -la` solves it directly and instantly. Save your brainpower for when they actually become necessary.

---

> **5. 🇪🇬 كبسولة سيريوس (Sirius Notes - Egyptian Arabic)**
>
> بص    اللفل ده هيعلمك حاجة هتفضل معاك طول عمرك في اللينكس، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** الباسورد بتاع اللفل الجاي متخزن في ملف مخفي جوه فولدر اسمه `inhere`.
> - **الهدف:** تدخل فولدر اسمه `inhere`، وتدور جواه على ملف مخفي (ملف مش بيظهر بـ `ls` عادية)، وتقرا محتواه.
> - **الليه؟** الملفات المخفية من أقدم الحيل في عالم اللينكس عشان تخبي إعدادات أو باسوردات أو أي حاجة "مش عايز حد يشوفها بالصدفة." الهاكرز بيستغلوا الملفات المخفية بشكل كبير (فولدرات `.ssh` مخفية، ملفات malware مستخبية عشان تفضل موجودة)، فتعلم إزاي تكشفها مهارة مش هتقدر تستغنى عنها.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit3`. يلا نشوف اللي في الـ home directory بتاعنا:
>
> ```bash
> ls -la
> ```
> هتلاقي فولدر اسمه `inhere`. يلا ندخله:
>
> ```bash
> cd inhere
> ```
> - `cd` معناها "change directory". هي بالظبط زي إنك ماشي من أوضة (فولدر) لأوضة تانية. بتقوله اسم الفولدر اللي عايز تدخله، وخلاص بقيت "واقف جواه".
>
> دلوقتي، تعالى نجرب `ls` عادية الأول، عشان اثبتلك حاجة:
>
> ```bash
> ls
> ```
> هتلاقي... مفيش حاجة. فاضي تماماً. هتقولي "استنى، يعني الفولدر ده فاضي فعلاً؟" لأ—هقولك بالظبط ليه مفيش حاجة ظاهرة: افتراضياً، الـ `ls` بتخبي عمداً أي ملف أو فولدر اسمه بيبدأ بنقطة `.` (ده اصطلاح في اللينكس من عشرات السنين—الأسامي اللي بتبدأ بنقطة بتتعتبر ملفات "مخفية/نظام"، القصد منها إنها متبوظش شكل الـ view العادي عندك). الملف موجود فعلاً، بس الـ `ls` مش عايزة توريهولك.
>
> فدلوقتي هنطلع الفلاج اللي بيكشف كل حاجة:
>
> ```bash
> ls -la
> ```
> - `-a`: معناها "all"—ده الفلاج اللي بيجبر الـ `ls` توري الملفات المخفية كمان (أي حاجة بتبدأ بنقطة).
> - `-l`: معناها "long listing"—بيديك تفاصيل زي الصلاحيات والحجم، ده مفيد عشان تشوف الملف بوضوح وتتأكد إنه أصلاً قابل للقراءة.
>
> دلوقتي هتلاقي ملف زي `...Hiding-From-You` أو شبهه (الاسم الفعلي بيختلف حسب نسخة التحدي، بس هيبدأ بنقطة بشكل واضح).
>
> اقراه بـ `cat`:
>
> ```bash
> cat .filename-you-found
> ```
> غيّر `.filename-you-found` بالاسم الحقيقي اللي شفته في مخرجات `ls -la`. الأمر ده هيطلعلك باسورد اللفل الرابع على الشاشة على طول.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> الملفات المخفية (dotfiles) منتشرة في أنظمة اللينكس الحقيقية—زي `.bashrc` و `.bash_history` و `.ssh/` وملفات `.env` بتاعة التطبيقات، وفولدرات `.git/` بتاعة الـ version control.
>
> - **🔴 من منظور الـ Red Team:** بعد ما الهاكر يخترق جهاز، دايماً بيشغل `ls -la` في كل فولدر بيدخله، وبيدور بالتحديد على الـ dotfiles. ليه؟ لإن `.bash_history` ممكن يكشف أوامر اتكتبت قبل كده (أحياناً فيها باسوردات مكتوبة غلط بالخطأ في نص عادي!)، و `.ssh/id_rsa` ممكن يكون مفتاح خاص يديك دخول لسيرفرات تانية، وملفات `.env` غالباً فيها بيانات دخول قواعد بيانات ومفاتيح API في نص عادي. الهاكرز كمان بيخبوا ملفاتهم الخبيثة بنفس الطريقة، بيسموها بنقطة قدامها عشان تندمج ومتظهرش لأدمن كسول شغال `ls` عادية.
> - **🔵 من منظور الـ Blue Team:** أبداً متخزنش باسوردات في dotfiles من غير تشفير، ودايماً راقب أي dotfiles غريبة بتتعمل في فولدرات غير متوقعة (زي `/tmp/.hidden_backdoor`)—دي تقنية كلاسيكية للـ persistence بتاعة المالوير. فرق الحماية بتستخدم أدوات File Integrity Monitoring (FIM) بالتحديد عشان تنبههم لو فيه ملفات مخفية جديدة أو اتعدلت.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** خلي `ls -la` رد فعلك التلقائي كل ما تدخل فولدر جديد من دلوقتي. متثقش تاني في `ls` العادية لوحدها. وكمان ثبّت `cd` في دماغك للتنقل بين الفولدرات.
> - **تجاهل:** أدوات `file` و `du` و `find` لسه مبالغ فيها للفل ده لإن `ls -la` بتحل المشكلة مباشرة وفوراً. وفر طاقة دماغك للفلات اللي هيبقوا فيها فعلاً محتاجين.
