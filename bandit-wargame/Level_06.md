### 🏴‍☠️ [Level 5 &rarr; Level 6]

**1. 🎯 The Objective & The "Why"**
- **The Question:** The password for the next level is stored in a file somewhere under the `inhere` directory and has all of the following properties: human-readable, 1033 bytes in size, not executable.
- **The Goal:** This time it's not one flat folder with 10 files — `inhere` contains MULTIPLE nested subdirectories, each hiding several decoy files. You need to hunt down ONE specific file across this whole tree, based on three exact criteria: it's readable text, it's exactly 1033 bytes, and it's NOT marked as executable.
- **The Why:** This is your first real introduction to **searching by file attributes/metadata instead of by name**. In real investigations (forensics, incident response, or offensive recon), you rarely know the exact filename you're looking for — but you often know its characteristics (size, permissions, owner, modification date). Learning to search the filesystem intelligently based on properties instead of blindly guessing names is a massive productivity and effectiveness skill.

**2. 💻 The Execution (Step-by-Step)**
You're logged in as `bandit5`. Let's move into the target directory:

```bash
cd inhere
```

Let's peek at the structure:

```bash
ls -la
```
You'll see a bunch of folders named something like `maybehere00`, `maybehere01`, `maybehere02`, etc. Each of these contains MORE files, and possibly more subfolders. Manually `cd`-ing into every single one and running `ls` and `file` on every file would take forever and be incredibly inefficient — imagine doing this on a real server with thousands of files. This is exactly the pain point that the `find` command was built to solve.

```bash
find . -type f -size 1033c ! -executable
```

Let's break this down piece by piece because every single flag here matters:
- `find`: This is the command itself. Its entire job is to **recursively search through directories and subdirectories** (meaning it digs into every folder, and every folder inside that folder, automatically) looking for files/folders that match criteria YOU specify.
- `.`: This tells `find` WHERE to start searching. The single dot `.` means "start right here, in my current directory" (which is `inhere`). `find` will then automatically dive into every subfolder underneath it.
- `-type f`: This filters results to only show **regular files** (type `f`), not directories. You might ask, "Why do we care?" I'll tell you: without this, `find` would also list every folder it walks through, cluttering your results with things you don't care about. We only want files.
- `-size 1033c`: This is the size filter. The number `1033` combined with the `c` suffix means "exactly 1033 **bytes** (c stands for 'characters/bytes')." Without the `c`, `find` assumes you mean 1033-byte BLOCKS (512 bytes each), which is a totally different, much bigger number — a classic beginner mistake. Always remember: `c` = bytes, no suffix = 512-byte blocks.
- `! -executable`: The exclamation mark `!` in bash/find means "NOT." So `-executable` alone would find files that ARE executable, but `! -executable` flips that logic to find files that are **NOT executable** — exactly matching the level's requirement.

Run that command, and `find` will spit out the exact path to the ONE file matching all three criteria simultaneously, something like:
```
./maybehere07/.file2
```

Now just read it:
```bash
cat ./maybehere07/.file2
```
(Replace with whatever exact path `find` actually gave you.) This dumps the level 6 password straight to your screen.

**3. 🌍 Real-World & Tactical Application**
Searching by attributes instead of names is a daily-driver skill for anyone doing serious systems work or security investigations.

- **🔴 Red Team Perspective:** During post-exploitation, attackers use `find` extensively to hunt for interesting files across an ENTIRE filesystem in seconds — like finding all SUID binaries for privilege escalation (`find / -perm -4000`), all recently modified files (to spot what an admin just touched), or all files owned by root that are world-writable (a massive misconfiguration attackers love). This exact `find` syntax pattern (type, size, permission filters) is copy-pasted constantly in real privilege escalation checklists (like the famous "linpeas" / "GTFOBins" scripts).
- **🔵 Blue Team Perspective:** Defenders use `find` for the OPPOSITE purpose — auditing systems for security misconfigurations before an attacker finds them first. Finding world-writable files, files with no owner (orphaned accounts), or unusually large log files that might indicate data exfiltration staging. Security hardening scripts almost always start with a battery of `find` commands to baseline a fresh server.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** The `find` syntax structure: `find [where] -type [f/d] -size [Nc] [permission flags]`. Memorize the `c` suffix for exact byte sizes — this trips up EVERYONE at least once. Also lock in `!` as the "NOT" negation operator.
- **IGNORE:** Don't worry about `find`'s hundreds of other flags right now (`-newer`, `-mtime`, `-user`, etc.) — you'll pick those up naturally as later levels demand them. Also don't manually `cd` into every subfolder like a caveman; that's the exact inefficient habit this level is trying to break.

---

> **5. 🇪🇬  (Sirius Notes - Egyptian Arabic)**
>
> بص  ، اللفل ده هيفتحلك باب في أقوى أوامر اللينكس على الإطلاق، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** الباسورد بتاع اللفل الجاي متخزن في ملف مكانه مكان ما تحت فولدر `inhere` وعنده الصفات دي: قابل للقراءة، حجمه 1033 بايت، ومش قابل للتنفيذ (not executable).
> - **الهدف:** المرة دي مش فولدر واحد مسطح فيه 10 ملفات—فولدر `inhere` فيه فولدرات فرعية كتير، كل واحد فيهم مخبي ملفات وهمية. لازم تدور على ملف واحد بس في الشجرة دي كلها، بناءً على 3 شروط بالظبط: نص مقروء، حجمه بالظبط 1033 بايت، ومش executable.
> - **الليه؟** ده أول تعريف حقيقي ليك بمفهوم **البحث بناءً على خصائص/بيانات الملف بدل الاسم**. في التحقيقات الحقيقية (الفحص الجنائي، الاستجابة للحوادث، أو حتى استطلاع هجومي)، نادراً ما تعرف اسم الملف بالظبط اللي بتدور عليه—لكن غالباً بتعرف صفاته (الحجم، الصلاحيات، المالك، تاريخ التعديل). تعلم إزاي تدور في نظام الملفات بذكاء بناءً على الخصائص بدل ما تخمن الأسامي عشوائي، دي مهارة ضخمة جداً في الإنتاجية والفعالية.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit5`. يلا ندخل الفولدر المستهدف:
>
> ```bash
> cd inhere
> ```
>
> يلا نشوف البنية:
>
> ```bash
> ls -la
> ```
> هتلاقي مجموعة فولدرات اسمها شبه `maybehere00` و `maybehere01` و `maybehere02` وهكذا. كل واحد فيهم فيه ملفات كتير، وممكن كمان فولدرات فرعية تانية. تخيل تدخل كل فولدر يدوي وتشغل `ls` و `file` على كل ملف—هياخد وقت طويل جداً وغير فعال، خصوصاً لو تخيلت السيناريو ده على سيرفر حقيقي فيه آلاف الملفات. دي بالظبط المشكلة اللي أمر `find` اتعمل عشان يحلها.
>
> ```bash
> find . -type f -size 1033c ! -executable
> ```
>
> خليني افكك الأمر ده حتة حتة لإن كل فلاج فيه مهم:
> - `find`: ده الأمر نفسه، شغلانته إنه **يدور بشكل متكرر (recursive) في كل الفولدرات والفولدرات الفرعية** (يعني بيغوص جوه كل فولدر، وكل فولدر جوه الفولدر ده، أوتوماتيك) ويدور على ملفات/فولدرات مطابقة للشروط اللي انت حددتها.
> - `.`: ده بيقول لـ `find` من فين يبدأ البحث. النقطة الواحدة `.` معناها "ابدأ من هنا، في الفولدر الحالي" (وهو `inhere`). و `find` هيغوص أوتوماتيك في كل الفولدرات الفرعية تحته.
> - `-type f`: ده بيفلتر النتايج عشان يوريك بس **الملفات العادية** (نوع `f`)، مش الفولدرات. هتقولي احنا مهتمين ليه؟ هقولك: من غيرها، `find` هيعرض كمان كل فولدر بيمشي فيه، وده هيملى النتايج بحاجات مش مهتم بيها. احنا عايزين الملفات بس.
> - `-size 1033c`: ده فلتر الحجم. الرقم `1033` مع اللاحقة `c` معناها "بالظبط 1033 **بايت** (الـ c اختصار لـ 'characters/bytes')." من غير الـ `c`، الـ `find` هيفهم إنك قصدك 1033 **بلوك** (كل بلوك 512 بايت)، وده رقم مختلف تماماً وأكبر بكتير—غلطة كلاسيكية بيقع فيها المبتدئين. افتكر دايماً: `c` = بايت، من غير لاحقة = بلوكات 512 بايت.
> - `! -executable`: علامة التعجب `!` في الباش/find معناها "لأ / NOT". فـ `-executable` لوحدها هتدور على ملفات هي فعلاً executable، لكن `! -executable` بتقلب المنطق ده وتدور على الملفات اللي **مش** executable—بالظبط زي المطلوب في اللفل.
>
> شغل الأمر ده، و`find` هيطلعلك المسار بالظبط للملف الوحيد اللي مطابق للشروط الثلاثة مع بعض، حاجة زي:
> ```
> ./maybehere07/.file2
> ```
>
> دلوقتي بس اقراه:
> ```bash
> cat ./maybehere07/.file2
> ```
> (غيّر بالمسار اللي طلعلك فعلاً من `find`.) الأمر ده هيطلعلك باسورد اللفل السادس على طول.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> البحث بناءً على خصائص الملف بدل اسمه مهارة أساسية يومية لأي حد شغال في الأنظمة أو التحقيقات الأمنية.
>
> - **🔴 من منظور الـ Red Team:** أثناء ما بعد الاختراق، الهاكرز بيستخدموا `find` بكثافة عشان يدوروا على ملفات مهمة في نظام الملفات كله في ثواني—زي إيجاد كل ملفات SUID لتصعيد الصلاحيات (`find / -perm -4000`)، أو كل الملفات اللي اتعدلت مؤخراً (عشان يعرفوا الأدمن لمس إيه)، أو كل الملفات المملوكة لـ root وقابلة للكتابة من الجميع (خطأ إعدادات كارثي بيحبه المهاجمين). نمط أوامر `find` بالظبط ده (فلاتر النوع والحجم والصلاحيات) بيتنسخ باستمرار في قوائم تصعيد الصلاحيات الحقيقية (زي سكريبتات "linpeas" و "GTFOBins" الشهيرة).
> - **🔵 من منظور الـ Blue Team:** فرق الحماية بتستخدم `find` للغرض المعاكس—تدقيق الأنظمة على إعدادات خاطئة قبل ما الهاكر يوصلها الأول. زي إيجاد ملفات قابلة للكتابة من الجميع، أو ملفات مالكها غير موجود (حسابات يتيمة)، أو ملفات لوج كبيرة بشكل غريب ممكن تدل على تجهيز لسرقة بيانات. سكريبتات تحصين الأنظمة غالباً بتبدأ بمجموعة من أوامر `find` عشان تحدد الوضع الأساسي لسيرفر جديد.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** بنية أمر `find`: `find [مكان] -type [f/d] -size [Nc] [فلاتر صلاحيات]`. احفظ اللاحقة `c` للحجم بالبايت بالظبط—دي بتوقع كل الناس على الأقل مرة. وكمان ثبّت `!` كعامل النفي "NOT".
> - **تجاهل:** متقلقش من مئات الفلاجات التانية بتاعة `find` دلوقتي (زي `-newer` و `-mtime` و `-user`)—هتاخدهم بشكل طبيعي لما اللفلات الجاية تحتاجهم. وكمان متدخلش كل فولدر بإيدك زي زمان، دي بالظبط العادة الغير فعالة اللي اللفل ده عايز يكسرها.
