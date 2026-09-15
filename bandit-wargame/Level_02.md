### 🏴‍☠️ [Level 1 &rarr; Level 2]

**1. 🎯 The Objective & The "Why"**
- **The Question:** The password for the next level is stored in a file called `-` (a single dash character) located in the home directory.
- **The Goal:** Read the content of a file that is LITERALLY named `-` (dash). Sounds simple, but this is a classic trick that trips up beginners hard, because the shell interprets `-` in a special way.
- **The Why:** This level teaches you that Linux/Bash has **special characters** that don't always behave the way you'd expect. Understanding how the shell parses your commands is CRUCIAL in offensive security—half of command injection and shell escaping techniques rely on knowing exactly how bash interprets weird characters.

**2. 💻 The Execution (Step-by-Step)**
You're logged in as `bandit1`. Let's check what's around:

```bash
ls -la
```
You'll see a file simply named `-`. Now, your first instinct might be to just do:

```bash
cat -
```

**STOP. Don't do that.** You might ask, "Why not? The file is literally called dash, so `cat -` should work, right?" Here's the trick: In almost EVERY command-line tool (including `cat`), a single dash `-` is a special convention. It doesn't mean "the file named dash." Instead, it means **"read from standard input (stdin)"** — meaning `cat` will just sit there waiting for YOU to manually type something into the terminal, instead of reading the file. It's not looking for a filename; it's looking for keyboard input. This would leave you stuck, confused, wondering why nothing's happening.

So how do we tell bash "no, I actually mean the FILE named dash, not the special stdin symbol"? We have two solid methods:

**Method 1: Use the relative path prefix `./`**
```bash
cat ./-
```
- `./` means "in the current directory." By prefixing the dash with `./`, you're explicitly telling `cat`: "This is a path, not a flag/symbol. Go look inside the current folder for something named `-`." This removes all ambiguity.

**Method 2: Use the absolute path**
```bash
cat /home/bandit1/-
```
- This spells out the FULL path from the root of the filesystem (`/`) all the way to the file. There's zero room for misinterpretation here because you're not starting the argument with a bare `-` character at all.

Either method works perfectly and will dump the password for level 2 onto your screen.

**3. 🌍 Real-World & Tactical Application**
This "dash trick" isn't just a wargame gimmick—it's a genuinely famous Unix/Linux quirk that catches even experienced sysadmins off guard.

- **🔴 Red Team Perspective:** Attackers exploit this exact ambiguity in **command injection attacks**. If a web app takes user input and passes it unsanitized into a shell command, an attacker can craft filenames or inputs starting with `-` to inject unexpected flags into a program (e.g., naming a file `-rf` and getting it accidentally treated as a dangerous flag if some vulnerable script processes filenames carelessly). Understanding how bash parses `-` helps you both exploit AND defend against these edge cases.
- **🔵 Blue Team Perspective:** Developers writing scripts that process filenames from user-controlled sources should ALWAYS sanitize input and use safe path handling (like prefixing with `./` internally, or using `--` to explicitly mark "end of options" in commands, e.g., `command -- $filename`). This prevents malicious filenames from being misinterpreted as flags.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** The concept that `-` alone often means "stdin/stdout" in Unix tools, and the fix is prefixing with `./` or using the full path. This single lesson will save you hours of confusion later on.
- **IGNORE:** Don't get lost in Google searching every single special character in bash right now (`|`, `>`, `&`, etc.)—those get their own lessons naturally as you progress. Just lock in the dash lesson for now.

---

> **5. 🇪🇬 كبسولة سيريوس (Sirius Notes - Egyptian Arabic)**
>
> بص   اللفل ده هيبان بسيط بس هو فخ كلاسيكي هيقع فيه ناس كتير، خليني اشرحلك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** الباسورد بتاع اللفل الجاي متخزن في ملف اسمه `-` (شرطة واحدة بس) وموجود في الـ home directory.
> - **الهدف:** تقرا محتوى ملف اسمه فعلاً "شرطة" (dash). شكلها سهلة، بس دي حيلة كلاسيكية بتقع فيها ناس كتير جداً لإن الـ Shell بتفهم الـ `-` بطريقة خاصة.
> - **الليه؟** اللفل ده بيعلّمك إن اللينكس/الباش فيه **رموز خاصة** مش دايماً بتتصرف زي ما انت متوقع. فهمك لطريقة تفسير الـ Shell للأوامر بتاعتك حاجة أساسية جداً في الأمن الهجومي—نص تقنيات الـ command injection والـ shell escaping معتمدة على إنك تعرف بالظبط الباش بتفسر الرموز الغريبة إزاي.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit1`. يلا نشوف اللي حوالينا:
>
> ```bash
> ls -la
> ```
> هتلاقي ملف اسمه بس `-`. أول حاجة هتيجي في بالك انك تعمل:
>
> ```bash
> cat -
> ```
>
> **وقف! ماتعملش كده.** هتقولي ليه؟ الملف فعلاً اسمه dash، يبقى `cat -` المفروض تشتغل صح؟ هقولك السر: في كل تقريباً أدوات سطر الأوامر (وفيها `cat`)، الشرطة الواحدة `-` دي اصطلاح خاص. مش معناها "الملف اللي اسمه شرطة"، معناها **"اقرا من الـ standard input (stdin)"**—يعني `cat` هتقعد تستنى انت تكتب حاجة بإيدك في التيرمينال، مش هتقرا الملف. هي مش بتدور على اسم ملف، هي بتستنى دخول من الكيبورد. هتفضل واقف مبهدل مش فاهم ايه اللي بيحصل.
>
> طب إزاي نقول للباش "لأ انا فعلاً قصدي الملف اللي اسمه شرطة، مش الرمز الخاص بتاع stdin"؟ عندنا طريقتين مضمونتين:
>
> **الطريقة الأولى: استخدم `./` قبل الاسم**
> ```bash
> cat ./-
> ```
> - `./` معناها "في الفولدر الحالي". لما تحط الرمز ده قبل الشرطة، انت بتقول للـ `cat` بوضوح: "دي مسار، مش فلاج أو رمز خاص. روح دور جوه الفولدر الحالي على حاجة اسمها `-`." كده مفيش أي لبس خالص.
>
> **الطريقة التانية: استخدم المسار الكامل**
> ```bash
> cat /home/bandit1/-
> ```
> - ده بيكتب المسار كامل من جذر نظام الملفات (`/`) لحد الملف. مفيش مجال للبس هنا لإنك أصلاً مش بادئ الـ argument بشرطة عارية.
>
> الطريقتين بيشتغلوا تمام وهيطلعوا لك باسورد اللفل التاني على الشاشة.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> حيلة "الشرطة" دي مش بس حاجة في اللعبة—دي فعلاً حاجة مشهورة في عالم اليونكس/لينكس بتوقع حتى الأدمنز المحترفين.
>
> - **🔴 من منظور الـ Red Team:** المهاجمين بيستغلوا اللبس ده بالظبط في هجمات **الـ command injection**. لو تطبيق ويب بياخد إدخال من اليوزر ويحطه من غير تنظيف جوه أمر شل، المهاجم ممكن يعمل اسم ملف أو إدخال بيبدأ بـ `-` ويحقن فلاج غير متوقعة في البرنامج (مثلاً اسم ملف `-rf` يتفسر بالغلط كفلاج خطيرة لو سكريبت ضعيف بيعامل أسماء الملفات باستهتار). فهمك لطريقة تفسير الباش للشرطة بيساعدك تستغل وتحمي من الحالات دي.
> - **🔵 من منظور الـ Blue Team:** المبرمجين اللي بيكتبوا سكريبتات بتعالج أسماء ملفات جاية من مصادر يتحكم فيها اليوزر، لازم دايماً ينظفوا الإدخال ويستخدموا معالجة آمنة للمسارات (زي إضافة `./` من جوه، أو استخدام `--` عشان يقول "خلصت الفلاجات هنا" زي `command -- $filename`). ده بيمنع أسماء الملفات الخبيثة من إنها تتفسر كفلاجات.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** المفهوم إن `-` لوحدها غالباً معناها "stdin/stdout" في أدوات اليونكس، والحل إنك تحط `./` قبلها أو تستخدم المسار الكامل. الدرس ده لوحده هيوفرلك ساعات من التخبط بعدين.
> - **تجاهل:** متضيعش وقتك دلوقتي تدور على كل رمز خاص في الباش (`|`, `>`, `&`, الخ)—دول هياخدوا دروسهم بشكل طبيعي وانت بتتقدم. بس ركز على درس الشرطة دلوقتي.
