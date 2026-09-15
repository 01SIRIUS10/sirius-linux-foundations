### 🏴‍☠️ [Level 28 &rarr; Level 29]

**1. 🎯 The Objective & The "Why"**
- **The Question:** There's a git repository at `ssh://bandit28-git@bandit.labs.overthewire.org/home/bandit28-git/repo` via port 2220. The password for `bandit28-git` is the same as for `bandit28`. From your local machine, clone the repository and find the password for the next level.
- **The Goal:** This LOOKS identical to the previous level at first glance — clone a repo, find a password. But this time, when you check the obvious files, the password is NOT sitting there in plain sight. It's been REMOVED in a later commit. Your job is to dig through the repository's HISTORY to find it in an OLDER version of the file.
- **The Why:** This is a critical, real-world lesson: **Git never truly forgets.** Deleting a secret in a new commit does NOT erase it from history — anyone with access to the repository (and its full commit log) can travel back in time and see EVERYTHING that was ever committed, even things that were later "removed." This is one of the most consequential security lessons in all of modern software development.

---

### 2. 💻 The Execution (Step-by-Step)

Just like last time, make sure you're working from YOUR LOCAL machine, not the Bandit server.

```bash
git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo
```
Enter the bandit28 password when prompted. Move into the newly cloned folder:

```bash
cd repo
```

Let's look at what's here:
```bash
ls -la
```
You'll probably see a `README.md`. Let's check it:
```bash
cat README.md
```
This time, you'll notice something suspicious — the file might say something like "the password is not here anymore" or simply not contain a password-looking string at all. You might ask, "Wait, did I clone the wrong thing?" No — I'll tell you exactly what's going on: the CURRENT version of this file has had the password DELETED from it in some later commit. But Git repositories don't just store the current snapshot — they store the ENTIRE history of every change ever made.

Let's look at that history:
```bash
git log
```
- `git log`: This command displays the complete commit history of the repository — every single commit ever made, showing its unique hash ID, the author, the date, and the commit message describing what changed.

You'll see something like:
```
commit a1b2c3d4... (HEAD -> master)
Author: ...
Date: ...
    fix info leak

commit e5f6g7h8...
Author: ...
Date: ...
    add missing data
```
That FIRST commit message, "fix info leak," is a massive red flag — that's almost certainly the commit where someone REMOVED the password, realizing it shouldn't have been there. This means the commit BEFORE it (the older one, `e5f6g7h8...` in our example) likely still HAS the password in the file.

Let's see exactly WHAT changed in that suspicious "fix info leak" commit:
```bash
git show a1b2c3d4
```
- `git show [commit-hash]`: This displays the exact changes (a "diff") introduced by that SPECIFIC commit — showing you precisely which lines were added and which were removed.

You'll see something like:
```
-Password: aBcDeFgHiJkLmNoPqRsTuVwXyZ12345
+Password: [redacted]
```
The line starting with `-` shows what was REMOVED (the real password), and the line starting with `+` shows what REPLACED it (a fake placeholder). Just by viewing the diff of the commit that supposedly "fixed" the leak, you can see EXACTLY what was leaked — that's your level 29 password, right there in plain view within the commit history.

**Alternative approach — checking out the old version directly:**
```bash
git checkout e5f6g7h8
cat README.md
```
- `git checkout [commit-hash]`: This rewinds your entire working directory back to exactly how it looked at that specific point in history. After running this, `cat README.md` shows you the OLD version of the file, complete with the password still intact.

Either method (`git show` on the deletion commit, or `git checkout` to the older commit) gets you the same result.

---

### 3. 🌍 Real-World & Tactical Application

This exact scenario — secrets accidentally committed and then "deleted" — happens CONSTANTLY in real corporate environments, and it's one of the most reliable sources of real breaches.

- **🔴 Red Team Perspective:** During recon on any target with a publicly accessible or leaked internal Git repository, running `git log` and `git show` on EVERY commit is standard operating procedure — developers panic-delete secrets ALL the time thinking it "fixes" the problem, not realizing the history is fully intact and accessible to anyone who clones the repo. Tools like `trufflehog` and `gitleaks` are specifically built to automatically scan an ENTIRE git history (not just the current snapshot) for patterns matching API keys, passwords, and tokens across every single commit ever made.
- **🔵 Blue Team Perspective:** This is why the ONLY correct response to a leaked secret in Git is: (1) immediately ROTATE/change that credential everywhere it's used (assume it's permanently compromised, because it is), and (2) if the repository history itself needs cleaning for compliance reasons, use specialized tools like BFG Repo-Cleaner or `git filter-repo` to surgically rewrite history and remove the secret from EVERY commit — a simple new commit that "deletes" the line is NEVER sufficient, and any security team relying on that alone has a serious blind spot.

---

### 4. 🚦 The Filter (Focus vs. Ignore)

- **FOCUS ON:** `git log` for viewing commit history, and `git show [hash]` for viewing exactly what changed in a specific commit. Internalize the core lesson HARD: Git history is permanent and fully recoverable by default — nothing is truly "deleted" just because a later commit removes it from the current file view.
- **IGNORE:** Don't worry about `git checkout`'s more advanced implications right now (like detached HEAD state warnings you might see) — for simple investigation purposes like this, it's a safe, read-only exploration. Also don't get bogged down learning `git blame` or other advanced history-tracing commands yet; `log` and `show` alone solve this level completely.

---

### 5. 🇪🇬  سيريوس (Sirius Notes - Egyptian Arabic)

> اللفل ده هيعلمك درس أمني ضخم: **Git مبينساش حاجة أبداً**، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** فيه مستودع git على `ssh://bandit28-git@bandit.labs.overthewire.org/home/bandit28-git/repo` عن طريق بورت 2220. باسورد `bandit28-git` هو نفسه باسورد `bandit28`. من جهازك المحلي، استنسخ المستودع ولاقي باسورد اللفل الجاي.
> - **الهدف:** ده هيبان مطابق تماماً للفل اللي فات أول وهلة—استنسخ مستودع، لاقي باسورد. بس المرة دي، لما تفحص الملفات الواضحة، الباسورد **مش موجود** قدامك بوضوح. اتشال في commit لاحق. مهمتك إنك تحفر جوه **تاريخ** المستودع عشان تلاقيه في نسخة قديمة من الملف.
> - **الليه؟** ده درس حقيقي وحرج: **Git مبينساش حاجة فعلياً.** مسح سر في commit جديد **مش بيمحيه** من التاريخ—أي حد عنده وصول للمستودع (وسجل الـ commits الكامل بتاعه) يقدر يرجع بالزمن ويشوف **كل حاجة** اتعمللها commit في أي وقت، حتى اللي اتشالت لاحقاً. ده واحد من أهم الدروس الأمنية في تطوير البرمجيات الحديث كله.
>
> **2. 💻 التنفيذ خطوة بخطوة**
>
> زي المرة اللي فاتت، تأكد إنك شغال من **جهازك المحلي**، مش سيرفر Bandit.
>
> ```bash
> git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo
> ```
> اكتب باسورد bandit28 لما يطلب منك. ادخل الفولدر المستنسخ الجديد:
>
> ```bash
> cd repo
> ```
>
> يلا نشوف اللي هنا:
> ```bash
> ls -la
> ```
> غالباً هتلاقي `README.md`. يلا نفحصه:
> ```bash
> cat README.md
> ```
> المرة دي، هتلاحظ حاجة مريبة—الملف ممكن يقول حاجة زي "الباسورد مش هنا بقى" أو ببساطة ملوش أي نص شكله باسورد خالص. هتقولي "استنى، أنا استنسخت الحاجة الغلط؟" لأ—هقولك بالظبط اللي بيحصل: النسخة **الحالية** من الملف ده اتشال منها الباسورد في commit لاحق. بس مستودعات Git مش بس بتخزن اللقطة الحالية—هي بتخزن **التاريخ الكامل** لكل تغيير اتعمل أي وقت.
>
> يلا نشوف التاريخ ده:
> ```bash
> git log
> ```
> - `git log`: الأمر ده بيعرض تاريخ الـ commits الكامل بتاع المستودع—كل commit اتعمل أي وقت، بيوريك الـ hash المميز بتاعه، والمؤلف، والتاريخ، ورسالة الـ commit اللي بتوصف إيه اللي اتغير.
>
> هتشوف حاجة زي:
> ```
> commit a1b2c3d4... (HEAD -> master)
> Author: ...
> Date: ...
>     fix info leak
>
> commit e5f6g7h8...
> Author: ...
> Date: ...
>     add missing data
> ```
> رسالة الـ commit **الأولى** دي، "fix info leak" (صلّح تسريب معلومات)، علم أحمر ضخم—دي شبه مؤكد الـ commit اللي حد فيه **شال** الباسورد، وهو حاسس إنه مكانش المفروض يكون موجود. ده معناه إن الـ commit **اللي قبلها** (الأقدم، `e5f6g7h8...` في مثالنا) غالباً **لسه فيه** الباسورد في الملف.
>
> يلا نشوف بالظبط **إيه اللي اتغير** في الـ commit المريب "fix info leak" ده:
> ```bash
> git show a1b2c3d4
> ```
> - `git show [commit-hash]`: ده بيعرض التغييرات بالظبط ("diff") اللي دخلت بواسطة الـ commit ده بالتحديد—بيوريك بالظبط أنهي سطور اتضافت وأنهي سطور اتشالت.
>
> هتشوف حاجة زي:
> ```
> -Password: aBcDeFgHiJkLmNoPqRsTuVwXyZ12345
> +Password: [redacted]
> ```
> السطر اللي بيبدأ بـ `-` بيوريك اللي **اتشال** (الباسورد الحقيقي)، والسطر اللي بيبدأ بـ `+` بيوريك اللي **حل محله** (نص وهمي بديل). بس بمجرد ما تشوف الـ diff بتاع الـ commit اللي المفروض "صلّح" التسريب، تقدر تشوف **بالظبط** إيه اللي اتسرب—ده باسورد اللفل 29 بتاعك، موجود قدامك بوضوح جوه تاريخ الـ commits.
>
> **طريقة بديلة—الرجوع مباشرة للنسخة القديمة:**
> ```bash
> git checkout e5f6g7h8
> cat README.md
> ```
> - `git checkout [commit-hash]`: ده بيرجّع فولدر شغلك بالكامل بالظبط لشكله في النقطة المحددة دي من التاريخ. بعد ما تشغل ده، `cat README.md` هتوريك النسخة **القديمة** من الملف، والباسورد لسه فيها.
>
> الطريقتين (`git show` على commit المسح، أو `git checkout` للـ commit الأقدم) بتوصلوك لنفس النتيجة.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
>
> السيناريو ده بالظبط—أسرار اتعمللها commit بالغلط وبعدين "اتشالت"—بيحصل **باستمرار** في بيئات شركات حقيقية، وهو واحد من أكتر مصادر الاختراقات الحقيقية موثوقية.
>
> - **🔴 من منظور الـ Red Team:** أثناء الاستطلاع على أي هدف عنده مستودع Git مكشوف عام أو متسرب داخلي، تشغيل `git log` و `git show` على **كل** commit إجراء تشغيلي قياسي—المبرمجين بيمسحوا الأسرار بذعر باستمرار فاكرين إن ده بيحل المشكلة، من غير ما يدركوا إن التاريخ سليم بالكامل ومتاح لأي حد يستنسخ المستودع. أدوات زي `trufflehog` و `gitleaks` معمولة بالتحديد عشان تفحص **تاريخ Git كامل** (مش بس اللقطة الحالية) على أنماط تطابق مفاتيح API وباسوردات وتوكينات عبر كل commit اتعمل أي وقت.
> - **🔵 من منظور الـ Blue Team:** ده سبب إن الرد الصحيح **الوحيد** على سر متسرب في Git هو: (1) **غيّر الباسورد فوراً** في كل مكان بيتستخدم فيه (افترض إنه مخترق بشكل دائم، لإنه فعلاً كده)، و(2) لو تاريخ المستودع نفسه محتاج تنظيف لأسباب امتثال، استخدم أدوات متخصصة زي BFG Repo-Cleaner أو `git filter-repo` عشان تعيد كتابة التاريخ جراحياً وتشيل السر من **كل** commit—مجرد commit جديد "بيمسح" السطر **أبداً مش كافي**، وأي فريق أمن معتمد على ده بس عنده ثغرة عمياء خطيرة.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** `git log` لعرض تاريخ الـ commits، و `git show [hash]` لعرض بالظبط إيه اللي اتغير في commit معين. استوعب الدرس الأساسي **بقوة**: تاريخ Git دائم وقابل للاسترجاع بالكامل افتراضياً—مفيش حاجة "متمسحة" فعلياً بس لإن commit لاحق شالها من عرض الملف الحالي.
> - **تجاهل:** متقلقش من تداعيات `git checkout` الأعمق دلوقتي (زي تحذيرات "detached HEAD state" ممكن تشوفها)—لأغراض التحقيق البسيط زي ده، هي استكشاف آمن للقراءة بس. وكمان متغرقش في تعلم `git blame` أو أوامر تتبع تاريخ متقدمة تانية دلوقتي؛ `log` و `show` بس بيحلوا اللفل ده بالكامل.
