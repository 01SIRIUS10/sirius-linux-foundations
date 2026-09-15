### 🏴‍☠️ [Level 31 &rarr; Level 32]

**1. 🎯 The Objective & The "Why"**
- **The Question:** There's a git repository at `ssh://bandit31-git@bandit.labs.overthewire.org/home/bandit31-git/repo` via port 2220. The password for `bandit31-git` is the same as for `bandit31`. Clone from your local machine and find the password.
- **The Goal:** This time, the challenge FLIPS entirely — instead of just reading history to find a hidden secret, you need to actively CONTRIBUTE to the repository. Specifically, you need to create a file with an EXACT name and EXACT content, commit it, and PUSH it back to the remote server. There's also a `.gitignore` file actively working AGAINST you, deliberately blocking the exact file type you need to add.
- **The Why:** This is your first hands-on experience with the FULL Git workflow — add, commit, push — not just passive investigation. It also teaches you about `.gitignore` and how to deliberately override it when necessary. Understanding the complete push/pull cycle is essential because in red team engagements involving CI/CD pipelines or Git-based deployment systems, you sometimes need to PUSH malicious content to trigger automated processes, not just READ existing history.

---

### 2. 💻 The Execution (Step-by-Step)

On your local machine:

```bash
git clone ssh://bandit31-git@bandit.labs.overthewire.org:2220/home/bandit31-git/repo
```
Enter the bandit31 password. Move in:

```bash
cd repo
```

Let's see what's here:
```bash
ls -la
cat README.md
```
The README will likely instruct you directly: something like "To access the next level, add the file `key.txt` with the content `May I come in?` to this repository." (The exact required content is specified inside the actual README you'll read — follow it precisely, character for character, since these checks are usually exact-match.)

Let's try the obvious approach first:
```bash
echo "May I come in?" > key.txt
git add key.txt
```
Running `git status` right after this might reveal something suspicious:
```bash
git status
```
You might notice `key.txt` isn't showing up as staged, or you get a message hinting it's being ignored. Let's check WHY:

```bash
cat .gitignore
```
- `.gitignore`: This is a special file that tells Git "never track/stage files matching these patterns, even if someone tries to `git add` them." You'll likely see a line like `*.txt` inside it — meaning EVERY file ending in `.txt` (including our `key.txt`) is being deliberately blocked from being added to the repository.

This is a deliberate obstacle. To get around it, we need to FORCE Git to add the file despite the ignore rule:

```bash
git add -f key.txt
```
- `-f`: Stands for "force." This flag explicitly overrides `.gitignore`'s exclusion rules for this specific file, telling Git: "I know this matches an ignore pattern, add it anyway."

Now let's confirm it's staged:
```bash
git status
```
You should now see `key.txt` listed as a new file ready to be committed (staged).

Let's commit it:
```bash
git commit -m "Adding key.txt as instructed"
```
- `git commit`: This takes everything currently STAGED and permanently records it as a new snapshot in the repository's history.
- `-m "message"`: Provides a commit message inline, describing what this commit does (a mandatory part of every meaningful commit, for tracking purposes).

Finally, let's push this commit back up to the remote server:
```bash
git push
```
- `git push`: This uploads your local commits to the remote repository (the one living on the Bandit server), synchronizing your changes back up to where everyone/everything else can see them.

You might be prompted for the password again during this push. Once it succeeds, the SERVER-SIDE process (likely a git hook watching for exactly this file/content combination) validates your submission and reveals the level 32 password, either directly in the push output on your terminal, or by making it available in the repository for you to `git pull` and read.

---

### 3. 🌍 Real-World & Tactical Application

Understanding the full add/commit/push cycle, along with `.gitignore` manipulation, is essential Git literacy for both developers and security professionals.

- **🔴 Red Team Perspective:** In engagements involving CI/CD pipelines, attackers who gain write access to a repository can PUSH malicious code that automatically triggers builds/deployments (a classic supply-chain attack vector) — understanding exactly how `add`/`commit`/`push` works, and how to bypass protective mechanisms like `.gitignore` or pre-commit hooks (using `-f` force flags or other bypass techniques), is directly relevant to simulating these real attack paths during authorized red team exercises.
- **🔵 Blue Team Perspective:** `.gitignore` is a CONVENTION, not a security control — it only stops ACCIDENTAL commits by well-behaved tools/developers, but any user with `-f` force-add access can trivially bypass it, exactly as we just did. Organizations relying on `.gitignore` alone to prevent sensitive files (like `.env` files with credentials) from ever entering a repository need ADDITIONAL enforcement layers — server-side pre-receive hooks that REJECT pushes containing certain file patterns, or dedicated secret-scanning gates in the CI/CD pipeline that block merges/pushes containing detected secrets.

---

### 4. 🚦 The Filter (Focus vs. Ignore)

- **FOCUS ON:** The complete `git add` → `git commit -m "message"` → `git push` workflow — this is the single most-used Git sequence in the entire software industry, full stop. Also lock in `git add -f` as the way to override `.gitignore` exclusions when you genuinely need to.
- **IGNORE:** Don't worry about more advanced `.gitignore` pattern syntax (negation patterns, directory-specific rules) right now — just understand the basic concept that it's a file-exclusion list, and `-f` overrides it for one specific `add` operation.

---

### 5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)

>  اللفل ده هيقلب الآية—بدل ما تقرا بس، دلوقتي هتساهم فعلياً في المستودع، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** فيه مستودع git على `ssh://bandit31-git@bandit.labs.overthewire.org/home/bandit31-git/repo` عن طريق بورت 2220. باسورد `bandit31-git` هو نفسه باسورد `bandit31`. استنسخ من جهازك المحلي ولاقي الباسورد.
> - **الهدف:** المرة دي، التحدي بيتقلب تماماً—بدل ما بس تقرا التاريخ عشان تلاقي سر مخبي، لازم فعلياً **تساهم** في المستودع. بالتحديد، لازم تعمل ملف باسم **بالظبط** ومحتوى **بالظبط**، تعمله commit، وتعمله push رجوع للسيرفر البعيد. كمان فيه ملف `.gitignore` شغال **ضدك**، بيمنع عمداً نوع الملف بالظبط اللي محتاج تضيفه.
> - **الليه؟** دي أول تجربة عملية ليك مع دورة Git **الكاملة**—add، commit، push—مش مجرد تحقيق سلبي. ده كمان بيعلمك عن `.gitignore` وإزاي تتجاوزه عمداً لما يلزم. فهم دورة الـ push/pull الكاملة أساسي لإن في عمليات Red Team اللي فيها خطوط CI/CD أو أنظمة نشر معتمدة على Git، أحياناً محتاج **تدفع** محتوى خبيث عشان تفعّل عمليات آلية، مش بس تقرا تاريخ موجود.
>
> **2. 💻 التنفيذ خطوة بخطوة**
>
> على جهازك المحلي:
>
> ```bash
> git clone ssh://bandit31-git@bandit.labs.overthewire.org:2220/home/bandit31-git/repo
> ```
> اكتب باسورد bandit31. ادخل:
>
> ```bash
> cd repo
> ```
>
> يلا نشوف اللي هنا:
> ```bash
> ls -la
> cat README.md
> ```
> الـ README غالباً هيوجهك مباشرة: حاجة زي "عشان توصل للفل الجاي، ضيف ملف `key.txt` بمحتوى `May I come in?` للمستودع ده." (المحتوى المطلوب بالظبط مذكور جوه الـ README اللي هتقراه فعلياً—اتبعه بدقة، حرف حرف، لإن الفحوصات دي عادةً مطابقة تماماً.)
>
> يلا نجرب الأسلوب الواضح الأول:
> ```bash
> echo "May I come in?" > key.txt
> git add key.txt
> ```
> تشغيل `git status` على طول بعد ده ممكن يكشف حاجة مريبة:
> ```bash
> git status
> ```
> ممكن تلاحظ إن `key.txt` مش ظاهر كـ staged، أو تاخد رسالة بتلمح إنه بيتم تجاهله. يلا نفحص **ليه**:
>
> ```bash
> cat .gitignore
> ```
> - `.gitignore`: ده ملف خاص بيقول لـ Git "أبداً متتبعش/تجهز ملفات مطابقة للأنماط دي، حتى لو حد حاول يعمل `git add` ليها." غالباً هتلاقي سطر زي `*.txt` جواه—معناها **كل** ملف بينتهي بـ `.txt` (بما فيهم `key.txt` بتاعنا) بيتمنع عمداً من إضافته للمستودع.
>
> ده عقبة متعمدة. عشان نتجاوزها، لازم **نجبر** Git يضيف الملف رغم قاعدة التجاهل:
>
> ```bash
> git add -f key.txt
> ```
> - `-f`: معناها "force" (إجبار). الفلاج ده بيتخطى صراحة قواعد الاستبعاد بتاعة `.gitignore` للملف ده بالتحديد، وبيقول لـ Git: "أنا عارف إن ده مطابق لنمط تجاهل، ضيفه رغم كده."
>
> يلا نتأكد إنه اتجهز:
> ```bash
> git status
> ```
> المفروض دلوقتي تشوف `key.txt` مسرود كملف جديد جاهز يتعمله commit (staged).
>
> يلا نعمله commit:
> ```bash
> git commit -m "Adding key.txt as instructed"
> ```
> - `git commit`: ده بياخد كل اللي **staged** حالياً ويسجله بشكل دائم كلقطة جديدة في تاريخ المستودع.
> - `-m "message"`: بيديك تديله رسالة commit inline، بتوصف الـ commit ده بيعمل إيه (جزء إجباري من أي commit مفيد، لأغراض التتبع).
>
> أخيراً، يلا نعمل push للـ commit ده رجوع للسيرفر البعيد:
> ```bash
> git push
> ```
> - `git push`: ده بيرفع الـ commits المحلية بتاعتك للمستودع البعيد (اللي عايش على سيرفر Bandit)، وبيزامن تغييراتك رجوع للمكان اللي أي حد/حاجة تانية تقدر تشوفها.
>
> ممكن تتطلب منك الباسورد تاني أثناء الـ push ده. بمجرد ما ينجح، العملية **من جهة السيرفر** (غالباً git hook بيراقب بالضبط تركيبة الملف/المحتوى دي) بتتحقق من إدخالك وبتكشف باسورد اللفل 32، إما مباشرة في مخرج الـ push على تيرمينالك، أو بتخليه متاح في المستودع عشان تعمله `git pull` وتقراه.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
>
> فهم دورة add/commit/push الكاملة، مع التلاعب بـ `.gitignore`، ثقافة Git أساسية للمبرمجين ومحترفي الأمن على حد سواء.
>
> - **🔴 من منظور الـ Red Team:** في عمليات فيها خطوط CI/CD، الهاكرز اللي بيوصلوا لصلاحية كتابة على مستودع يقدروا **يدفعوا** كود خبيث بيفعّل تلقائياً عمليات بناء/نشر (مسار هجوم كلاسيكي على سلسلة التوريد)—فهمك بالضبط إزاي `add`/`commit`/`push` بتشتغل، وإزاي تتجاوز آليات حماية زي `.gitignore` أو pre-commit hooks (باستخدام فلاجات `-f` الإجبارية أو تقنيات تجاوز تانية)، مرتبط مباشرة بمحاكاة مسارات الهجوم الحقيقية دي أثناء تمارين Red Team مصرح بها.
> - **🔵 من منظور الـ Blue Team:** الـ `.gitignore` عبارة عن **اتفاقية**، مش ضابط أمني—هي بس بتوقف الـ commits **العرضية** من أدوات/مبرمجين ملتزمين، بس أي يوزر عنده صلاحية `-f` force-add يقدر يتخطاها بسهولة، بالظبط زي ما عملنا لتونا. المؤسسات المعتمدة على `.gitignore` بس عشان تمنع ملفات حساسة (زي ملفات `.env` فيها بيانات دخول) من دخول مستودع أبداً محتاجة طبقات إنفاذ **إضافية**—server-side pre-receive hooks بترفض الـ pushes اللي فيها أنماط ملفات معينة، أو بوابات فحص أسرار مخصصة في خط CI/CD بتمنع الدمج/الدفع اللي فيه أسرار مكتشفة.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** دورة `git add` → `git commit -m "message"` → `git push` الكاملة—دي أكتر سلسلة Git استخداماً في صناعة البرمجيات كلها، نقطة. وكمان ثبّت `git add -f` كطريقة تتجاوز بيها استبعادات `.gitignore` لما فعلاً محتاج.
> - **تجاهل:** متقلقش من صيغة أنماط `.gitignore` الأكتر تقدماً دلوقتي (أنماط النفي، قواعد خاصة بفولدرات)—بس افهم المفهوم الأساسي إنها قائمة استبعاد ملفات، و`-f` بيتخطاها لعملية `add` واحدة محددة.
