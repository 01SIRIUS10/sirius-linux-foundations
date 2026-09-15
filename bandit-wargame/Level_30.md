### 🏴‍☠️ [Level 29 &rarr; Level 30]

**1. 🎯 The Objective & The "Why"**
- **The Question:** There's a git repository at `ssh://bandit29-git@bandit.labs.overthewire.org/home/bandit29-git/repo` via port 2220. The password for `bandit29-git` is the same as for `bandit29`. From your local machine, clone the repository and find the password for the next level.
- **The Goal:** You clone the repo expecting the same easy win as before, but this time the current file content genuinely does NOT contain a usable password — and unlike last level, checking the commit history (`git log`) won't reveal it either, because it was never DELETED from the main line of development. Instead, it's hiding in a completely SEPARATE **branch** that was never merged into the main one. You need to discover that branch exists and switch into it.
- **The Why:** This teaches you about **Git branches** — one of the most fundamental concepts in version control. Real-world codebases almost NEVER live on a single linear timeline; developers constantly create branches for features, experiments, and fixes. Attackers and auditors both need to know that "the master/main branch" is often just ONE slice of the full picture — critical secrets, unfinished features, or leftover debug code frequently sit forgotten in branches nobody bothered to clean up.

---

### 2. 💻 The Execution (Step-by-Step)

Make sure you're on YOUR OWN local machine, not inside the Bandit SSH session.

```bash
git clone ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo
```
Enter the bandit29 password when prompted (same as your regular bandit29 SSH password). Move into the cloned folder:

```bash
cd repo
```

Let's check what's here on the surface:
```bash
cat README.md
```
You'll likely see something that says the password is NOT here anymore, or some text with no real password format. Let's check the log just to rule that avenue out:
```bash
git log
```
You'll probably see it's clean/unremarkable — no obvious "fix leak" commit like last time. This is your cue that the secret isn't hidden in the HISTORY of this branch — it might be hiding on a DIFFERENT branch entirely.

Every git repository can have multiple branches — think of it like parallel timelines of the same project, all stemming from a shared starting point but diverging to hold different, independent sets of changes. By default, when you clone, you only SEE the current branch (usually `master` or `main`) unless you explicitly ask to see what else exists.

```bash
git branch -a
```
- `git branch`: Lists the branches in the repository.
- `-a`: Stands for "all." Without this flag, you'd only see LOCAL branches you've already checked out. The `-a` flag reveals ALL branches, including remote-tracking branches that exist on the server but that you haven't personally switched to yet.

This will output something like:
```
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/sploits-dev
```
The asterisk `*` marks which branch you're CURRENTLY on (`master`). But look — there's another one listed: `remotes/origin/sploits-dev`. That's a branch that exists on the SERVER but that you haven't switched your local view to yet. Its name alone (something dev/experimental-sounding) is a strong hint this is worth investigating.

Let's switch to it:
```bash
git checkout sploits-dev
```
- `git checkout [branchname]`: This command switches your entire working directory to match whatever that branch's content looks like. Git is smart enough here to recognize you mean the remote branch `origin/sploits-dev` and automatically set up a local tracking branch for you.

Now let's look at the files again, since checking out a different branch can introduce NEW or DIFFERENT files:
```bash
ls -la
cat README.md
```
This time, the file content on THIS branch reveals the level 30 password directly.

---

### 3. 🌍 Real-World & Tactical Application

Branch enumeration is a routine but frequently overlooked step in both software auditing and offensive security assessments.

- **🔴 Red Team Perspective:** When attackers get access to a source code repository (leaked, purchased, or exposed via a misconfigured `.git` folder on a public server), checking ONLY the default branch is a rookie mistake. Real attackers ALWAYS run `git branch -a` and `git log --all --oneline` to enumerate every branch, because feature branches, "wip" (work-in-progress) branches, and abandoned dev branches frequently contain hardcoded test credentials, unfinished security controls, or debug backdoors that developers never intended to leave in the final merged codebase.
- **🔵 Blue Team Perspective:** Organizations should enforce branch hygiene as a security practice — old, stale, unmerged branches should be regularly audited and deleted, and CI/CD security scanning tools (like secret scanners) should be configured to scan ALL branches, not just `main`/`master`, since a secret sitting quietly in a forgotten feature branch is just as real a leak as one in the primary codebase.

---

### 4. 🚦 The Filter (Focus vs. Ignore)

- **FOCUS ON:** `git branch -a` to reveal ALL branches (including remote ones you haven't checked out), and `git checkout [branchname]` to switch your working directory to that branch's content. Internalize that "the default branch" is never the WHOLE story.
- **IGNORE:** Don't worry about the deeper mechanics of how Git internally tracks remote vs. local branches (refs, tracking branches, `origin/` prefixes) right now — just know that `-a` reveals everything, and `checkout` lets you jump into it.

---

### 5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)

>  اللفل ده هيعلمك إن الحقيقة مش دايماً على الخط الرئيسي—أحياناً بتكون في تايم لاين تاني كامل، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** فيه مستودع git على `ssh://bandit29-git@bandit.labs.overthewire.org/home/bandit29-git/repo` عن طريق بورت 2220. باسورد `bandit29-git` هو نفسه باسورد `bandit29`. من جهازك المحلي، استنسخ المستودع ولاقي باسورد اللفل الجاي.
> - **الهدف:** بتستنسخ المستودع متوقع فوز سهل زي قبل، بس المرة دي محتوى الملف الحالي فعلاً **مفيهوش** باسورد قابل للاستخدام—وعلى عكس اللفل اللي فات، فحص تاريخ الـ commits (`git log`) مش هيكشفه برضو، لإنه أصلاً مانتشلش من الخط الرئيسي للتطوير. بدل كده، هو مستخبي في **فرع (branch)** منفصل تماماً ما اتدمجش أبداً في الفرع الرئيسي. لازم تكتشف إن الفرع ده موجود وتنتقل ليه.
> - **الليه؟** ده بيعلمك عن **فروع Git**—واحد من أهم مفاهيم التحكم بالإصدارات. الأكواد الحقيقية شبه أبداً ما بتعيش على خط زمني واحد—المبرمجين باستمرار بيعملوا فروع للميزات والتجارب والإصلاحات. المهاجمين والمدققين محتاجين يعرفوا إن "الفرع الرئيسي/الماستر" غالباً بس **جزء واحد** من الصورة الكاملة—أسرار حرجة، أو ميزات غير مكتملة، أو كود debug متروك، غالباً بتفضل ناسية في فروع محدش اهتم ينضفها.
>
> **2. 💻 التنفيذ خطوة بخطوة**
>
> تأكد إنك على **جهازك المحلي**، مش جوه جلسة SSH بتاعة Bandit.
>
> ```bash
> git clone ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo
> ```
> اكتب باسورد bandit29 لما يطلب منك (نفس باسورد SSH العادي بتاعك). ادخل الفولدر المستنسخ:
>
> ```bash
> cd repo
> ```
>
> يلا نشوف اللي على السطح هنا:
> ```bash
> cat README.md
> ```
> غالباً هتشوف حاجة بتقول إن الباسورد مش هنا بقى، أو نص من غير فورمات باسورد حقيقي. يلا نفحص اللوج بس عشان نستبعد المسار ده:
> ```bash
> git log
> ```
> غالباً هتلاقيه نضيف/عادي—مفيش commit واضح شبه "fix leak" زي المرة اللي فاتت. دي إشارتك إن السر مش مستخبي في **تاريخ** الفرع ده—ممكن يكون مستخبي في فرع **مختلف** تماماً.
>
> كل مستودع git ممكن يكون عنده فروع متعددة—فكر فيها كتايم لاينز متوازية لنفس المشروع، كلهم طالعين من نقطة بداية مشتركة بس بيتفرعوا عشان يحتفظوا بمجموعات مستقلة من التغييرات. افتراضياً، لما تستنسخ، بتشوف بس الفرع الحالي (غالباً `master` أو `main`) إلا لو طلبت صراحة تشوف إيه الموجود غير كده.
>
> ```bash
> git branch -a
> ```
> - `git branch`: بيسرد الفروع في المستودع.
> - `-a`: معناها "all" (الكل). من غير الفلاج ده، هتشوف بس الفروع **المحلية** اللي فعلاً عملتلها checkout قبل كده. الفلاج `-a` بيكشف **كل** الفروع، بما فيهم الفروع المتتبعة عن بعد الموجودة على السيرفر واللي انت شخصياً لسه ما نقلتش ليها.
>
> ده هيطلعلك حاجة زي:
> ```
> * master
>   remotes/origin/HEAD -> origin/master
>   remotes/origin/master
>   remotes/origin/sploits-dev
> ```
> النجمة `*` بتعلّم على الفرع اللي انت **عليه حالياً** (`master`). بس بص—فيه فرع تاني مذكور: `remotes/origin/sploits-dev`. ده فرع موجود على **السيرفر** بس انت لسه ما نقلتش عرضك المحلي ليه. اسمه لوحده (شكله dev/تجريبي) تلميح قوي إنه يستاهل التحقيق.
>
> يلا ننقل ليه:
> ```bash
> git checkout sploits-dev
> ```
> - `git checkout [اسم_الفرع]`: الأمر ده بينقل فولدر شغلك بالكامل عشان يطابق شكل محتوى الفرع ده. Git ذكي بما فيه الكفاية هنا إنه يفهم إنك قصدك الفرع البعيد `origin/sploits-dev` ويظبط فرع تتبع محلي ليك أوتوماتيك.
>
> دلوقتي يلا نبص على الملفات تاني، لإن الانتقال لفرع مختلف ممكن يجيب ملفات **جديدة** أو **مختلفة**:
> ```bash
> ls -la
> cat README.md
> ```
> المرة دي، محتوى الملف على الفرع **ده** هيكشف باسورد اللفل 30 مباشرة.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
>
> حصر الفروع خطوة روتينية بس غالباً بتتنسى في التدقيق البرمجي وتقييمات الأمن الهجومي.
>
> - **🔴 من منظور الـ Red Team:** لما الهاكرز يوصلوا لمستودع كود مصدري (متسرب، مشترى، أو مكشوف عن طريق فولدر `.git` مظبوط غلط على سيرفر عام)، فحص **بس** الفرع الافتراضي غلطة مبتدئين. الهاكرز المحترفين دايماً بيشغلوا `git branch -a` و `git log --all --oneline` عشان يحصروا كل فرع، لإن فروع الميزات، وفروع "wip" (شغل تحت التنفيذ)، وفروع التطوير المهجورة غالباً فيها بيانات دخول اختبار ثابتة، أو ضوابط أمنية غير مكتملة، أو باكدورات debug المبرمجين ما كانوش ناويين يسيبوها في الكود النهائي المدمج.
> - **🔵 من منظور الـ Blue Team:** المؤسسات لازم تفرض نظافة الفروع كممارسة أمنية—الفروع القديمة والمهجورة وغير المدمجة لازم تتدقق وتتمسح بشكل دوري، وأدوات فحص أمان الـ CI/CD (زي فاحصات الأسرار) لازم تتظبط عشان تفحص **كل** الفروع، مش بس `main`/`master`، لإن سر قاعد بهدوء في فرع ميزة منسي تسريب حقيقي بنفس درجة تسريب في الكود الأساسي.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** `git branch -a` لكشف **كل** الفروع (بما فيهم البعيدة اللي لسه ما عملتلهاش checkout)، و `git checkout [اسم_الفرع]` عشان تنقل فولدر شغلك لمحتوى الفرع ده. استوعب إن "الفرع الافتراضي" أبداً مش القصة كاملة.
> - **تجاهل:** متقلقش من الآليات الداخلية الأعمق لطريقة تتبع Git للفروع البعيدة مقابل المحلية (refs، فروع التتبع، بادئات `origin/`) دلوقتي—بس اعرف إن `-a` بتكشف كل حاجة، و`checkout` بتخليك تنط ليها.
