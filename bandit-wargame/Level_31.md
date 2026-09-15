### 🏴‍☠️ [Level 30 &rarr; Level 31]

**1. 🎯 The Objective & The "Why"**
- **The Question:** There's a git repository at `ssh://bandit30-git@bandit.labs.overthewire.org/home/bandit30-git/repo` via port 2220. The password for `bandit30-git` is the same as for `bandit30`. Clone from your local machine and find the password.
- **The Goal:** Just like the branch hunt in the last level, this time the current file has NO password, `git log` shows nothing revealing, and there's no interesting branch either. The secret this time is attached to a **tag** — a special, fixed pointer to a specific commit, typically used to mark release versions (`v1.0`, `v2.0`, etc.).
- **The Why:** This introduces you to **Git tags**, another core version control concept. Tags are commonly used to mark "official" release points in a project's history. Attackers auditing a leaked codebase check tags because release tags sometimes point to commits that contain secrets which were present at THAT release moment but got cleaned up later — same underlying lesson as branches, just a different mechanism for referencing a specific point in history.

---

### 2. 💻 The Execution (Step-by-Step)

On your local machine:

```bash
git clone ssh://bandit30-git@bandit.labs.overthewire.org:2220/home/bandit30-git/repo
```
Enter the bandit30 password. Move in:

```bash
cd repo
```

Check the obvious stuff first:
```bash
cat README.md
git log
git branch -a
```
Assume all three come up empty-handed or unremarkable — no visible password, no suspicious commit messages, no extra branches worth checking. This is your cue to check for TAGS.

```bash
git tag
```
- `git tag`: This lists every tag that exists in the repository. Tags are essentially bookmarks/labels attached to a specific commit — unlike branches, they typically DON'T move forward as new commits get added; they stay frozen pointing at one exact historical point.

You'll see something like:
```
secret
```
That's a tag with a suspicious, deliberately hinting name. Let's examine what that tag actually points to:

```bash
git show secret
```
- `git show [tagname]`: Just like showing a commit, this displays the FULL content/diff of whatever commit the tag `secret` points to.

This will show you the commit's details along with its diff/content, and buried in there — likely in a file that shows different content than what's on your current `master` branch — is the level 31 password.

**Alternative approach — checking out the tag directly:**
```bash
git checkout secret
cat README.md
```
This physically moves your working directory to match exactly what things looked like at that tagged commit, letting you browse the files normally with `cat`/`ls` as if you'd time-traveled back to that exact snapshot.

---

### 3. 🌍 Real-World & Tactical Application

Tag enumeration rounds out your Git reconnaissance toolkit alongside commit history and branch checking.

- **🔴 Red Team Perspective:** During source code review or leaked-repository analysis, a complete Git audit ALWAYS includes `git tag` and `git show` for each one — release tags sometimes get created BEFORE a final security cleanup pass, meaning the tagged commit can contain secrets, debug flags, or vulnerable code that was fixed AFTER the tag was cut but never retroactively cleaned from the tagged snapshot itself. Tools like `trufflehog` explicitly scan tags in addition to branches and raw commit history for exactly this reason.
- **🔵 Blue Team Perspective:** Development teams should treat tags with the SAME security scrutiny as any other part of history — since tags are permanent, immutable references, any secret ever captured in a tagged release is essentially "published forever" from a security auditing standpoint. Automated secret-scanning in CI/CD pipelines should be configured to also scan on tag-creation events, not just push events to branches.

---

### 4. 🚦 The Filter (Focus vs. Ignore)

- **FOCUS ON:** `git tag` (list all tags) and `git show [tagname]` (inspect what a tag points to) as the third pillar of your Git investigation toolkit, alongside `git log`/`git show [commit]` and `git branch -a`/`git checkout`. Together, these three techniques (history, branches, tags) cover the vast majority of "hidden secrets in a git repo" scenarios you'll encounter.
- **IGNORE:** Don't worry about the technical distinction between "lightweight" and "annotated" tags right now (a deeper Git concept) — for investigative purposes, `git tag` and `git show` work identically regardless of tag type.

---

### 5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)

>  اللفل ده هيعلمك عن الـ Tags، ركيزة تانية في أدوات تحقيقك في Git، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** فيه مستودع git على `ssh://bandit30-git@bandit.labs.overthewire.org/home/bandit30-git/repo` عن طريق بورت 2220. باسورد `bandit30-git` هو نفسه باسورد `bandit30`. استنسخ من جهازك المحلي ولاقي الباسورد.
> - **الهدف:** زي بالظبط مطاردة الفرع في اللفل اللي فات، المرة دي الملف الحالي **مفيهوش** باسورد، و`git log` مبيوريش حاجة تكشف، ومفيش فرع مثير للاهتمام برضو. السر المرة دي متعلق بـ **تاج (tag)**—مؤشر خاص وثابت لـ commit معين، بيتستخدم عادةً لتعليم نسخ الإصدارات (`v1.0`، `v2.0`، الخ).
> - **الليه؟** ده بيعرفك على **تاجات Git**، مفهوم أساسي تاني في التحكم بالإصدارات. التاجات بتتستخدم بشكل شائع عشان تعلّم نقاط "إصدار رسمي" في تاريخ المشروع. الهاكرز اللي بيدققوا على كود مصدري متسرب بيفحصوا التاجات لإن تاجات الإصدارات أحياناً بتشير لـ commits فيها أسرار كانت موجودة في لحظة الإصدار ده بس اتنضفت بعدين—نفس الدرس الأساسي زي الفروع، بس آلية مختلفة للإشارة لنقطة معينة في التاريخ.
>
> **2. 💻 التنفيذ خطوة بخطوة**
>
> على جهازك المحلي:
>
> ```bash
> git clone ssh://bandit30-git@bandit.labs.overthewire.org:2220/home/bandit30-git/repo
> ```
> اكتب باسورد bandit30. ادخل:
>
> ```bash
> cd repo
> ```
>
> افحص الحاجات الواضحة الأول:
> ```bash
> cat README.md
> git log
> git branch -a
> ```
> افترض إن الثلاثة كلهم طلعوا فاضيين أو عاديين—مفيش باسورد ظاهر، مفيش رسايل commit مريبة، مفيش فروع إضافية تستاهل الفحص. دي إشارتك إنك تفحص **التاجات**.
>
> ```bash
> git tag
> ```
> - `git tag`: ده بيسرد كل تاج موجود في المستودع. التاجات أساساً علامات مرجعية/ليبلات ملصقة على commit معين—على عكس الفروع، هي عادةً **مش بتتحرك** لقدام مع الـ commits الجديدة؛ بتفضل ثابتة تشير لنقطة تاريخية واحدة بالظبط.
>
> هتشوف حاجة زي:
> ```
> secret
> ```
> ده تاج باسم مريب بيلمح عمداً. يلا نفحص فين التاج ده فعلاً بيشير:
>
> ```bash
> git show secret
> ```
> - `git show [اسم_التاج]`: بالظبط زي عرض commit، ده بيعرض المحتوى/الـ diff الكامل لأي commit التاج `secret` بيشير له.
>
> ده هيوريك تفاصيل الـ commit مع الـ diff/المحتوى بتاعه، ومدفون هناك—غالباً في ملف بيوري محتوى مختلف عن اللي على فرعك الـ `master` الحالي—باسورد اللفل 31.
>
> **طريقة بديلة—الرجوع للتاج مباشرة:**
> ```bash
> git checkout secret
> cat README.md
> ```
> ده بينقل فولدر شغلك فعلياً عشان يطابق بالظبط الشكل وقت الـ commit المُعلّم عليه، ويخليك تتصفح الملفات عادي بـ `cat`/`ls` كأنك سافرت بالزمن لتلك اللقطة بالظبط.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
>
> حصر التاجات بيكمّل مجموعة أدوات استطلاع Git بتاعتك جنب تاريخ الـ commits وفحص الفروع.
>
> - **🔴 من منظور الـ Red Team:** أثناء مراجعة الكود المصدري أو تحليل مستودع متسرب، تدقيق Git كامل **دايماً** بيتضمن `git tag` و `git show` لكل واحد فيهم—تاجات الإصدارات أحياناً بتتعمل **قبل** جولة تنظيف أمنية أخيرة، يعني الـ commit المعلّم عليه ممكن يحتوي أسرار، أو flags تصحيح، أو كود فيه ثغرات اتصلح **بعد** ما التاج اتقطع بس محدش نضف اللقطة المعلّم عليها بأثر رجعي. أدوات زي `trufflehog` بتفحص صراحة التاجات بجانب الفروع وتاريخ الـ commits الخام لنفس السبب ده بالظبط.
> - **🔵 من منظور الـ Blue Team:** فرق التطوير لازم تتعامل مع التاجات بنفس التدقيق الأمني زي أي جزء تاني من التاريخ—بما إن التاجات مراجع دائمة وثابتة، أي سر اتلقط في إصدار معلّم عليه أساساً "منشور للأبد" من ناحية التدقيق الأمني. فحص الأسرار الآلي في خطوط CI/CD لازم يتظبط عشان يفحص كمان عند أحداث إنشاء التاجات، مش بس أحداث الـ push للفروع.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** `git tag` (سرد كل التاجات) و `git show [اسم_التاج]` (فحص فين التاج بيشير) كركيزة ثالثة في مجموعة أدوات تحقيقك بتاعة Git، جنب `git log`/`git show [commit]` و `git branch -a`/`git checkout`. الثلاثة تقنيات دول مع بعض (التاريخ، الفروع، التاجات) بيغطوا الأغلبية الساحقة من سيناريوهات "أسرار مخبية في مستودع git" اللي هتقابلها.
> - **تجاهل:** متقلقش من التمييز التقني بين تاجات "lightweight" و"annotated" دلوقتي (مفهوم أعمق في Git)—لأغراض التحقيق، `git tag` و `git show` بيشتغلوا بنفس الشكل بغض النظر عن نوع التاج.
