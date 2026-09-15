### 🏴‍☠️ [Level 22 → Level 23]

**1. 🎯 The Objective & The "Why"**
- **The Question:** A program is running automatically at regular intervals from cron. Look in `/etc/cron.d/` for the configuration and see what command is being executed. NOTE: reading shell scripts written by other people is a very useful skill — the script here is intentionally easy to read, and if you're confused, try executing it to see debug output.
- **The Goal:** Same investigative process as last level — trace a cron job to its script — but this time the script is a bit more clever. It doesn't dump the password to a static, predictable filename. Instead, it dynamically GENERATES a filename using some command output, meaning you need to actually READ and understand the script's LOGIC (not just its final output) to figure out where the password actually ends up.
- **The Why:** This level pushes you one step further into genuine **shell script literacy** — the ability to read someone else's bash code and mentally "execute" it in your head to predict its behavior. This is an absolutely non-negotiable skill in security work: reading malware scripts, auditing deployment scripts, and reverse-engineering automation are all just variations of "can you read bash and understand what it actually does."

---

### 💻 The Execution (Step-by-Step)
You're logged in as `bandit22`. Let's find the cron configuration:

```bash
ls -la /etc/cron.d/
```
You'll spot a file relevant to this level, something like `cronjob_bandit23`. Let's read it:

```bash
cat /etc/cron.d/cronjob_bandit23
```
This will show something like:
```
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh
```
Same structure as before — runs every minute, as user `bandit23`, executing that script. Let's read the script itself:

```bash
cat /usr/bin/cronjob_bandit23.sh
```
The content will look something like this:
```bash
#!/bin/bash
myname=$(whoami)
target=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)
echo Copying passwordfile /etc/bandit_pass/$myname to /tmp/$target
cat /etc/bandit_pass/$myname > /tmp/$target
```

You might ask, "This looks like gibberish, how do I 'read' this?" I'll walk you through it line by line, exactly like the level suggests — treating it like a puzzle to mentally trace through:

- `myname=$(whoami)`: This runs the `whoami` command (which prints the CURRENT user the script is running as) and stores that result in a variable called `myname`. Since this script is being run BY the cron job AS user `bandit23` (remember the cron config line said `bandit23`), `whoami` here will output `bandit23`, and `myname` becomes `bandit23`.
- `target=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)`: This is the tricky line. Let's trace it piece by piece:
  - `echo I am user $myname`: Since `$myname` is `bandit23`, this literally outputs the exact string `I am user bandit23`.
  - `| md5sum`: We pipe that exact string into `md5sum`, a tool that calculates the MD5 cryptographic hash of whatever text it receives. This will ALWAYS produce the exact same hash output for the exact same input string — meaning if we can figure out and reproduce the exact input string ourselves, we can calculate the SAME hash independently, without needing to actually run the script.
  - `| cut -d ' ' -f 1`: `md5sum`'s raw output normally includes the hash PLUS a trailing filename/dash, separated by a space (like `abc123hash  -`). The `cut` command trims this down: `-d ' '` sets the delimiter to a space character, and `-f 1` says "give me only the FIRST field" — meaning just the raw hash value itself, discarding everything after the space.
  - So `target` ends up being the MD5 hash of the exact string `"I am user bandit23"`.
- `cat /etc/bandit_pass/$myname > /tmp/$target`: This takes bandit23's password file and writes it into a file inside `/tmp`, but the FILENAME is that calculated MD5 hash value, not a predictable name like last level.

So here's the beautiful part — since WE know the exact formula the script uses (`echo I am user bandit23 | md5sum | cut -d ' ' -f 1`), we can just calculate that SAME hash ourselves, manually, right now, without waiting for or needing to trigger the actual cron job:

```bash
echo I am user bandit23 | md5sum | cut -d ' ' -f 1
```

Running this outputs a specific hash value — and THAT hash value is the exact filename sitting in `/tmp` waiting for us. Now just read it:

```bash
cat /tmp/[the_hash_value_you_calculated]
```
This reveals your level 23 password.

**As the level suggests, if you're ever stuck understanding a script, just RUN it yourself and watch what it prints (the `echo` line in this script literally tells you what it's doing as debug output) — this is a legitimate, professional technique for understanding unfamiliar code when reading alone isn't enough.**

---

### 🌍 Real-World & Tactical Application
Reading and mentally tracing bash scripts is a core, everyday skill in security operations.

- **🔴 Red Team Perspective:** Attackers CONSTANTLY encounter obfuscated or "clever" scripts during engagements — malware droppers, custom deployment automation, or defensive scripts trying to make output unpredictable (exactly like this MD5-hashed filename trick, which is a legitimate, mild security-through-obscurity technique). Being able to read the LOGIC and reproduce the SAME calculation independently — instead of needing to actually witness the script execute — is a genuine skill used in exploit development and malware reverse engineering, where you often need to predict values/filenames/tokens a program will generate before it generates them.
- **🔵 Blue Team Perspective:** Security engineers reviewing deployment or automation scripts need this exact skill to catch subtle bugs or vulnerabilities buried in seemingly simple bash logic — a single misplaced variable, missing quotes around a variable (which can lead to word-splitting vulnerabilities), or predictable "randomness" (like this MD5 trick, which ISN'T actually random since it's based on a fully predictable input) can create serious security holes. This exact "hash the username to generate a filename" pattern, while okay for a training wargame, would be considered weak security in production — an attacker who can predict the hash can predict the file location.

---

### 🚦 The Filter (Focus vs. Ignore)
- **FOCUS ON:** The methodology of reading a script LINE BY LINE, tracking what each variable holds after each command runs, and mentally "executing" the logic in your head. Also lock in `md5sum` and `cut -d ' ' -f 1` as a common pattern for extracting just the hash value from command output.
- **IGNORE:** Don't worry about MD5's cryptographic weaknesses as a hashing algorithm right now (it's broken for security purposes like password storage, but that's a separate, later lesson) — here it's just being used as a "semi-unpredictable filename generator," and that's the only property that matters for this level.

---

### 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)

> اللفل ده هيخليك تقرا كود باش زي المحترفين، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** فيه برنامج بيتشغل أوتوماتيك على فترات منتظمة من الـ cron. شوف في `/etc/cron.d/` عشان تلاقي الإعدادات وتشوف أنهي أمر بيتنفذ. ملحوظة: قراءة سكريبتات كتبها ناس تانية مهارة مفيدة جداً—السكريبت هنا معمول عمداً عشان يكون سهل القراءة، ولو تايه، جرب تشغله عشان تشوف معلومات الـ debug اللي بيطبعها.
> - **الهدف:** نفس عملية التحقيق بتاعة اللفل اللي فات—تتبع مهمة cron لسكريبتها—بس المرة دي السكريبت أذكى شوية. هو مش بيطبع الباسورد في اسم ملف ثابت ومتوقع. بدل كده، هو بيولّد اسم ملف **ديناميكي** باستخدام مخرج أمر معين، ده معناه إنك لازم فعلياً **تقرا وتفهم منطق السكريبت** (مش بس مخرجه النهائي) عشان تعرف الباسورد بيروح فين فعلياً.
> - **الليه؟** اللفل ده بيدفعك خطوة كمان جوه **قراءة سكريبتات الشِل بشكل حقيقي**—القدرة إنك تقرا كود باش كتبه حد تاني وتنفذه ذهنياً في دماغك عشان تتوقع سلوكه. دي مهارة إجبارية جداً في شغل الأمن: قراءة سكريبتات مالوير، تدقيق سكريبتات النشر، والهندسة العكسية للأتمتة، كلهم أشكال مختلفة من سؤال واحد: "تقدر تقرا باش وتفهم بيعمل إيه فعلياً؟"
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit22`. يلا نلاقي إعدادات الـ cron:
>
> ```bash
> ls -la /etc/cron.d/
> ```
> هتلاقي ملف يخص اللفل ده، حاجة زي `cronjob_bandit23`. يلا نقراه:
>
> ```bash
> cat /etc/cron.d/cronjob_bandit23
> ```
> ده هيوريك حاجة زي:
> ```
> * * * * * bandit23 /usr/bin/cronjob_bandit23.sh
> ```
> نفس البنية زي قبل—بيشتغل كل دقيقة، بصفة يوزر `bandit23`، وبينفذ السكريبت ده. يلا نقرا السكريبت نفسه:
>
> ```bash
> cat /usr/bin/cronjob_bandit23.sh
> ```
> المحتوى هيبان شبه كده:
> ```bash
> #!/bin/bash
> myname=$(whoami)
> target=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)
> echo Copying passwordfile /etc/bandit_pass/$myname to /tmp/$target
> cat /etc/bandit_pass/$myname > /tmp/$target
> ```
>
> هتقولي "الكلام ده شكله رموز غريبة، أقراه إزاي؟" هقولك خطوة بخطوة، بالظبط زي ما اللفل بيقترح—عاملها كأنها لغز بتتتبعه ذهنياً:
>
> - `myname=$(whoami)`: ده بيشغل أمر `whoami` (اللي بيطبع اليوزر **الحالي** اللي السكريبت شغال بصفته) وبيخزن النتيجة دي في متغير اسمه `myname`. بما إن السكريبت ده بيتشغل عن طريق مهمة الـ cron بصفة يوزر `bandit23` (افتكر سطر إعدادات cron قال `bandit23`)، الـ `whoami` هنا هيطلع `bandit23`، والمتغير `myname` هيبقى `bandit23`.
> - `target=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)`: ده السطر الحيلة. خليني اتتبعه حتة حتة:
>   - `echo I am user $myname`: بما إن `$myname` هي `bandit23`، ده حرفياً بيطبع النص بالظبط `I am user bandit23`.
>   - `| md5sum`: بنعمل pipe للنص ده جوه `md5sum`، أداة بتحسب الـ hash التشفيري MD5 لأي نص بتستقبله. ده دايماً **هيطلع نفس ناتج الـ hash بالظبط** لنفس نص الإدخال بالظبط—يعني لو قدرنا نعرف ونكرر نفس نص الإدخال بأنفسنا، نقدر نحسب **نفس الـ hash** بشكل مستقل، من غير ما نحتاج فعلياً نشغل السكريبت.
>   - `| cut -d ' ' -f 1`: مخرج `md5sum` الخام عادةً بيحتوي على الـ hash **زائد** اسم ملف/شرطة زايدة في الآخر، متفصولين بمسافة (زي `abc123hash  -`). أمر `cut` بيقطع ده: `-d ' '` بيظبط الفاصل على حرف مسافة، و `-f 1` بيقول "هاتلي الحقل **الأول** بس"—يعني قيمة الـ hash الخام لوحدها، وياخد اللي بعد المسافة.
>   - فالمتغير `target` بيبقى في النهاية هو الـ MD5 hash بتاع النص بالظبط `"I am user bandit23"`.
> - `cat /etc/bandit_pass/$myname > /tmp/$target`: ده بياخد ملف باسورد bandit23 ويكتبه في ملف جوه `/tmp`، بس **اسم الملف** هو قيمة الـ MD5 hash المحسوبة دي، مش اسم متوقع زي اللفل اللي فات.
>
> فهنا الجزء الجميل—بما إننا **عارفين** المعادلة بالظبط اللي السكريبت بيستخدمها (`echo I am user bandit23 | md5sum | cut -d ' ' -f 1`)، نقدر بس نحسب **نفس الـ hash** إحنا بأنفسنا، يدوياً، دلوقتي، من غير ما نستنى أو نحتاج نشغل مهمة الـ cron الفعلية:
>
> ```bash
> echo I am user bandit23 | md5sum | cut -d ' ' -f 1
> ```
>
> تشغيل الأمر ده هيطلع قيمة hash محددة—والقيمة دي هي بالظبط اسم الملف الموجود في `/tmp` مستنينا. دلوقتي بس اقراه:
>
> ```bash
> cat /tmp/[قيمة_الـ_hash_اللي_حسبتها]
> ```
> ده هيكشف باسورد اللفل 23 بتاعك.
>
> **زي ما اللفل بيقترح، لو حسيت انك تايه وأنت بتفهم سكريبت، بس شغّله انت بنفسك وشوف بيطبع إيه (سطر الـ `echo` في السكريبت ده حرفياً بيقولك بيعمل إيه كمعلومات debug)—دي تقنية شرعية واحترافية لفهم كود مش مألوف لما القراءة لوحدها متكونش كافية.**
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> قراءة وتتبع سكريبتات الباش ذهنياً مهارة أساسية ويومية في عمليات الأمن.
>
> - **🔴 من منظور الـ Red Team:** الهاكرز باستمرار بيقابلوا سكريبتات معماة أو "ذكية" أثناء العمليات—أدوات توصيل مالوير، أتمتة نشر مخصصة، أو سكريبتات دفاعية بتحاول تخلي المخرج غير متوقع (بالظبط زي حيلة اسم الملف بـ MD5 hash دي، وهي تقنية "أمان بالتعمية" شرعية وخفيفة). قدرتك إنك تقرا **المنطق** وتعيد إنتاج **نفس الحساب** بشكل مستقل—بدل ما تحتاج فعلياً تشاهد السكريبت وهو بيتنفذ—مهارة حقيقية بتتستخدم في تطوير الاستغلالات والهندسة العكسية للمالوير، فين غالباً محتاج تتوقع قيم/أسامي ملفات/توكينات هيولّدها برنامج قبل ما هو يولّدها.
> - **🔵 من منظور الـ Blue Team:** مهندسي الأمن اللي بيراجعوا سكريبتات النشر أو الأتمتة محتاجين بالظبط المهارة دي عشان يمسكوا bugs دقيقة أو ثغرات مدفونة في منطق باش شكله بسيط—متغير في مكان غلط، أو علامات تنصيص ناقصة حوالين متغير (ده ممكن يؤدي لثغرات word-splitting)، أو "عشوائية" متوقعة (زي حيلة الـ MD5 دي، اللي **مش عشوائية فعلياً** لإنها مبنية على إدخال متوقع بالكامل) ممكن تخلق ثغرات أمنية خطيرة. النمط ده بالظبط ("اعمل hash لاسم اليوزر عشان تولّد اسم ملف")، رغم إنه مقبول للعبة تدريبية، هيتعتبر أمان ضعيف في الإنتاج—هاكر يقدر يتوقع الـ hash يقدر يتوقع مكان الملف.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** منهجية قراءة السكريبت **سطر بسطر**، وتتبع إيه اللي كل متغير بيحمله بعد ما كل أمر يتنفذ، وتنفيذ المنطق ذهنياً في دماغك. وكمان ثبّت `md5sum` مع `cut -d ' ' -f 1` كنمط شائع لاستخراج قيمة الـ hash بس من مخرج أمر.
> - **تجاهل:** متقلقش من ضعف MD5 كخوارزمية تشفيرية دلوقتي (هي مكسورة لأغراض أمنية زي تخزين الباسوردات، بس ده درس منفصل لاحق)—هنا هي بتتستخدم بس كـ "مولّد اسم ملف شبه غير متوقع"، وده هي الخاصية الوحيدة المهمة للفل ده.
