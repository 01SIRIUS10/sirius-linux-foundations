### 🏴‍☠️ [Level 19 &rarr; Level 20]

**1. 🎯 The Objective & The "Why"**
- **The Question:** To gain access to the next level, you should use the setuid binary in the home directory. Execute it without arguments to find out how to use it. The password for this level can be found in `/etc/bandit_pass` after using the setuid binary.
- **The Goal:** There's an executable program sitting in your home folder with a special permission bit set called **SUID (Set User ID)**. This special bit lets a regular user run the program with the PERMISSIONS OF THE FILE'S OWNER, instead of their own permissions — meaning even though YOU are `bandit19`, running this program temporarily grants you the powers of whoever OWNS the binary (which turns out to be `bandit20`). You need to figure out how to use it to read `bandit20`'s password.
- **The Why:** This is your first real, hands-on exposure to one of the SINGLE most critical privilege escalation concepts in all of Linux security: **SUID binaries**. Misconfigured SUID programs are responsible for a HUGE percentage of real-world Linux privilege escalation exploits. Understanding how they work — and how to find and abuse them — is an absolute must-have skill for anyone in offensive security.

**2. 💻 The Execution (Step-by-Step)**
You're logged in as `bandit19`. Let's see what we're working with:

```bash
ls -la
```
You'll see a file, typically named something like `bandit20-do`. Notice its permission string in the `-la` output — it'll look something like `-rwsr-x---`. That `s` where you'd normally expect an `x` (in the "owner" execute position) is the tell-tale sign of the **SUID bit** being set. This is a special flag on the file that says: "whenever ANYONE executes this program, run it with the privileges of the file's OWNER, not the privileges of whoever launched it."

Let's confirm WHO owns this file:
```bash
ls -la bandit20-do
```
You'll see the owner listed as `bandit20`. This means: even though you (bandit19) are the one launching this program, while it's RUNNING, it temporarily "becomes" bandit20 in terms of what files it's allowed to touch.

Now, let's follow the level's instruction and run it WITHOUT any arguments first, just to see what it expects:

```bash
./bandit20-do
```
- `./`: Remember from way back, this prefix means "look for this file in the CURRENT directory" (necessary because the current directory usually isn't in your shell's search path by default, for security reasons).
- `bandit20-do`: The name of our setuid program.

Running it bare like this will likely print out a usage message, something like:
```
Run a command as another user.
Example: ./bandit20-do id
```
This tells us exactly how to use it: it expects us to pass IT a command as an argument, and IT will execute that command on our behalf, but running AS bandit20 instead of as us.

So now, we give it the exact command we actually want to run — reading the level 20 password file, which normally only `bandit20` has permission to read:

```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```
- `./bandit20-do`: We're launching the SUID program.
- `cat /etc/bandit_pass/bandit20`: This is the ARGUMENT we're feeding it — the actual command we want executed. Since `bandit20-do` runs with bandit20's privileges (thanks to the SUID bit), and it's running `cat` on a file that bandit20 IS allowed to read, this succeeds even though bandit19 (you) would normally get "Permission denied" trying this directly.

The output will be the level 20 password, printed straight to your screen.

**3. 🌍 Real-World & Tactical Application**
SUID binary abuse is one of THE most classic and heavily-exploited privilege escalation techniques in the real world.

- **🔴 Red Team Perspective:** On virtually every real Linux privilege escalation checklist (like the famous `linpeas.sh` script or manual enumeration guides), one of the FIRST things attackers do after gaining a low-privilege shell is hunt for SUID binaries: `find / -perm -4000 -type f 2>/dev/null` (that `-perm -4000` flag specifically searches for files with the SUID bit set). If they find a custom or misconfigured SUID binary that lets them run arbitrary commands, read arbitrary files, or spawn a shell — especially one owned by `root` — that's an instant, often trivial path to full root privilege escalation. GTFOBins.github.io is an entire website dedicated to cataloging how common Linux binaries (like `find`, `vim`, `less`) can be abused for privilege escalation specifically when they have the SUID bit set.
- **🔵 Blue Team Perspective:** System administrators should regularly audit their servers for unnecessary SUID binaries using that exact `find -perm -4000` command, and remove the SUID bit (`chmod u-s filename`) from anything that doesn't ABSOLUTELY require it. Security hardening guides for Linux servers almost universally include "audit and minimize SUID/SGID binaries" as a top recommendation, because every single SUID program is a potential privilege escalation vector if it's even slightly misconfigured or has a bug in it.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** Recognizing the `s` permission flag (instead of `x`) in `ls -la` output as the visual signature of a SUID binary. Understand the CORE concept: SUID means "runs with the FILE OWNER's privileges, not the launching user's privileges." Also remember `find / -perm -4000` as the standard way to hunt for these across an entire filesystem.
- **IGNORE:** Don't worry right now about the difference between SUID and its sibling SGID (Set Group ID) bit — they work on the same principle but for group ownership instead of user ownership, and that distinction matters more in later, deeper privilege escalation study. For this level, just SUID is what matters.

---

> **5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)**
>
>  اللفل ده هيعرفك على واحد من أخطر وأشهر مفاهيم تصعيد الصلاحيات في اللينكس كله، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** عشان توصل للفل الجاي، لازم تستخدم برنامج SUID موجود في الـ home directory. شغله من غير أي arguments عشان تعرف تستخدمه إزاي. باسورد اللفل ده هتلاقيه في `/etc/bandit_pass` بعد ما تستخدم برنامج الـ setuid.
> - **الهدف:** فيه برنامج تنفيذي موجود في الفولدر بتاعك عليه صلاحية خاصة اسمها **SUID (Set User ID)**. الصلاحية الخاصة دي بتخلي يوزر عادي يقدر يشغل البرنامج **بصلاحيات مالك الملف**، مش بصلاحياته هو—يعني رغم إنك انت `bandit19`، تشغيل البرنامج ده مؤقتاً بيديك قوة أي حد **مالك** البرنامج ده (واللي بيطلع إنه `bandit20`). لازم تعرف إزاي تستخدمه عشان تقرا باسورد `bandit20`.
> - **الليه؟** ده أول تعامل حقيقي وعملي ليك مع واحد من أهم مفاهيم تصعيد الصلاحيات في اللينكس كله: **برامج الـ SUID**. برامج SUID مظبوطة بشكل خاطئ مسؤولة عن نسبة **ضخمة** من ثغرات تصعيد الصلاحيات الحقيقية في اللينكس. فهمك إزاي بتشتغل—وإزاي تلاقيها وتستغلها—مهارة إجبارية لأي حد شغال في الأمن الهجومي.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit19`. يلا نشوف احنا شغالين على إيه:
>
> ```bash
> ls -la
> ```
> هتلاقي ملف، غالباً اسمه شبه `bandit20-do`. لاحظ سلسلة الصلاحيات بتاعته في مخرجات `-la`—هتبان شبه `-rwsr-x---`. الحرف `s` ده مكان ما كنت متوقع `x` عادي (في موضع تنفيذ "المالك")، ده العلامة المميزة إن **بت الـ SUID** مفعّل. دي علامة خاصة على الملف بتقول: "أي حد يشغل البرنامج ده، شغّله بصلاحيات مالك الملف، مش بصلاحيات اللي شغّله."
>
> يلا نتأكد **مين** مالك الملف ده:
> ```bash
> ls -la bandit20-do
> ```
> هتلاقي المالك مكتوب `bandit20`. ده معناه: رغم إنك انت (bandit19) اللي بتشغل البرنامج ده، وهو شغال، هو مؤقتاً "بيبقى" bandit20 من ناحية إيه اللي مسموحله يلمسه.
>
> دلوقتي، يلا نتبع تعليمات اللفل ونشغله من غير أي arguments الأول، بس عشان نشوف هو متوقع إيه:
>
> ```bash
> ./bandit20-do
> ```
> - `./`: افتكر من زمان، البادئة دي معناها "دور على الملف ده في الفولدر الحالي" (ضروري لإن الفولدر الحالي عادةً مش في مسار بحث الشِل بشكل افتراضي، لأسباب أمنية).
> - `bandit20-do`: اسم برنامج الـ setuid بتاعنا.
>
> تشغيله كده عاري هيطبع غالباً رسالة استخدام، حاجة زي:
> ```
> Run a command as another user.
> Example: ./bandit20-do id
> ```
> ده بيقولنا بالظبط إزاي نستخدمه: هو متوقع منّا نمرر له أمر كـ argument، وهو هينفذ الأمر ده نيابة عنّا، بس شغال **كـ bandit20** بدل ما يشتغل بصفتنا احنا.
>
> فدلوقتي، خلينا نديله الأمر بالظبط اللي عايزينه فعلاً—قراءة ملف باسورد اللفل 20، اللي عادةً بس `bandit20` عنده صلاحية يقراه:
>
> ```bash
> ./bandit20-do cat /etc/bandit_pass/bandit20
> ```
> - `./bandit20-do`: بنشغل برنامج الـ SUID.
> - `cat /etc/bandit_pass/bandit20`: ده الـ **argument** اللي بنديهوله—الأمر الفعلي اللي عايزين ينفذ. بما إن `bandit20-do` بيشتغل بصلاحيات bandit20 (بفضل بت الـ SUID)، وهو بيشغل `cat` على ملف bandit20 **مسموحله** يقراه، العملية دي بتنجح رغم إن bandit19 (انت) عادةً هياخد "Permission denied" لو حاول يعمل ده مباشرة.
>
> المخرج هيكون باسورد اللفل 20، مطبوع مباشرة على شاشتك.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> استغلال برامج SUID واحدة من أكلاسيك وأكتر تقنيات تصعيد الصلاحيات استغلالاً في الواقع.
>
> - **🔴 من منظور الـ Red Team:** في تقريباً كل قائمة تصعيد صلاحيات لينكس حقيقية (زي سكريبت `linpeas.sh` الشهير أو أدلة الحصر اليدوية)، من أول الحاجات اللي المهاجمين بيعملوها بعد ما يوصلوا لشِل بصلاحيات محدودة إنهم يدوروا على برامج SUID: `find / -perm -4000 -type f 2>/dev/null` (الفلاج `-perm -4000` ده بيدور بالتحديد على الملفات اللي بت SUID مفعّل فيها). لو لقوا برنامج SUID مخصص أو مظبوط بشكل خاطئ بيسمحلهم ينفذوا أوامر عشوائية، أو يقروا ملفات عشوائية، أو يفتحوا شِل—خصوصاً واحد مملوك لـ `root`—ده طريق فوري وغالباً بسيط لتصعيد صلاحيات كامل لـ root. موقع GTFOBins.github.io موقع كامل مخصص لتوثيق إزاي البرامج الشائعة في اللينكس (زي `find` و `vim` و `less`) ممكن تُستغل لتصعيد الصلاحيات بالتحديد لما بت الـ SUID يكون مفعّل عليها.
> - **🔵 من منظور الـ Blue Team:** أدمنز الأنظمة لازم يدققوا بشكل دوري على سيرفراتهم عشان يلاقوا برامج SUID غير ضرورية باستخدام نفس أمر `find -perm -4000`، ويشيلوا بت الـ SUID (`chmod u-s filename`) من أي حاجة مش محتاجاها فعلاً. أدلة تحصين سيرفرات اللينكس تقريباً كلها بتضم "دقق وقلل برامج SUID/SGID" كواحدة من أهم التوصيات، لإن كل برنامج SUID هو مسار محتمل لتصعيد الصلاحيات لو فيه أي خطأ إعداد بسيط أو bug فيه.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** التعرف على علامة الصلاحية `s` (بدل `x`) في مخرجات `ls -la` كإشارة بصرية لبرنامج SUID. افهم **الفكرة الأساسية**: SUID معناها "بيشتغل بصلاحيات مالك الملف، مش صلاحيات اليوزر اللي شغّله." وكمان افتكر `find / -perm -4000` كالطريقة القياسية للدوران على البرامج دي في نظام الملفات كله.
> - **تجاهل:** متقلقش دلوقتي من الفرق بين SUID وأخوه بت SGID (Set Group ID)—بيشتغلوا بنفس المبدأ بس لملكية الجروب بدل ملكية اليوزر، والفرق ده بيبقى أهم في دراسة تصعيد صلاحيات أعمق لاحقاً. للفل ده، SUID بس هو المهم.
