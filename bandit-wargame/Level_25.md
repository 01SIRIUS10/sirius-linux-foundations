### 🏴‍☠️ [Level 24 &rarr; Level 25]

**1. 🎯 The Objective & The "Why"**
- **The Question:** A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 AND a secret numeric 4-digit PIN code. There's no way to retrieve the PIN except by trying ALL 10,000 combinations — brute-forcing. You don't need to create new connections each time.
- **The Goal:** You need to write a script that AUTOMATES sending your password plus every single possible 4-digit PIN (0000 through 9999) to a listening service, and captures/filters the ONE response that actually contains a real password instead of an error message.
- **The Why:** This is your first genuine introduction to **brute-forcing**, one of the most fundamental (and controversial) techniques in all of offensive security. Understanding how to automate a "try everything" attack, and specifically how to do it EFFICIENTLY (the hint about not needing new connections each time is a huge efficiency lesson), is core knowledge for password attacks, PIN cracking, and any scenario involving a limited keyspace.

---

### 2. 💻 The Execution (Step-by-Step)

You're logged in as `bandit24`. Let's grab your current password first:

```bash
cat /etc/bandit_pass/bandit24
```

Now, let's think about the problem. Manually connecting via `nc` and typing 10,000 different combinations by hand would take literally forever — this is EXACTLY why we need to write a script that automates this entire process.

First, let's understand what format the daemon expects. Connecting manually and experimenting (or just reading the level clue carefully) tells us it expects something like: `password pincode` on one line. Let's build our brute-force script.

**Step 1: Generate every possible 4-digit PIN combination.**

```bash
for i in $(seq -w 0000 9999); do echo $i; done > /tmp/pins.txt
```
- `seq -w 0000 9999`: `seq` generates a sequence of numbers. `-w` stands for "equal width," meaning it PADS all numbers with leading zeros so they're all the same length (so `5` becomes `0005`, not just `5` — critical, because a 4-digit PIN like `0007` needs those leading zeros preserved). This generates every number from `0000` to `9999`.
- We loop through and dump each one into a file called `pins.txt`, one PIN per line — though honestly, we can skip this file-generation step and do it all inline in our main script, which is cleaner.

**Step 2: Write the actual brute-force script.**

Let's create a script that loops through every PIN, sends it along with the password, and filters for a successful response:

```bash
for i in $(seq -w 0000 9999); do
    echo "[Password_Appears_Here] $i"
done | nc -q1 localhost 30002 > /tmp/results24.txt
```

Let's break this down carefully because there's a LOT happening:
- `for i in $(seq -w 0000 9999); do ... done`: This is a bash loop that runs 10,000 times, once for each possible PIN, with `$i` holding the current PIN value each time.
- `echo "[Password_Appears_Here] $i"`: Inside the loop, we print a line containing our REAL bandit24 password followed by a space and the current PIN being tried. This is the exact format the daemon expects.
- `| nc -q1 localhost 30002`: Here's the CLEVER part addressing the level's hint ("you don't need to create new connections each time"). Instead of opening a brand new `nc` connection for EACH of the 10,000 attempts (which would be catastrophically slow), we pipe the ENTIRE loop's output — all 10,000 lines, generated one after another — into a SINGLE `nc` connection. The daemon reads each line one at a time, checks it, and responds, all within ONE continuous connection.
- `-q1`: This flag tells `nc` to wait 1 second after receiving EOF (end of input) before automatically closing the connection. Without this, `nc` might just hang forever waiting for more input, or close too abruptly before receiving the daemon's final responses.
- `> /tmp/results24.txt`: We redirect ALL the output (all 10,000 responses, mostly "wrong pin" style messages, and hopefully ONE actual password) into a file, since scrolling through 10,000 lines of terminal output live would be insane.

**Step 3: Filter through the results to find the ONE that worked.**

```bash
grep -v "Wrong" /tmp/results24.txt
```
- `grep -v`: Remember `-v` means "invert the match" — instead of showing lines that CONTAIN the pattern, show lines that DO NOT contain it. Since 9999 out of 10000 attempts will fail with some kind of "Wrong password/pin" message, we filter OUT every line containing "Wrong," leaving us with just the rare success message(s) that don't match that pattern.

This should leave you with a small handful of lines, one of which is clearly your level 25 password sitting right there.

---

### 3. 🌍 Real-World & Tactical Application

Brute-forcing over a persistent connection is a genuinely important technique across offensive security.

- **🔴 Red Team Perspective:** This exact "generate all possibilities, pipe through one persistent connection, filter results" pattern is the backbone of tools like Hydra, Medusa, and custom brute-force scripts used against login forms, SSH services, or API endpoints with limited keyspaces (PINs, short codes, 2FA bypass attempts). Attackers specifically optimize for connection reuse because many services have rate-limiting or connection-count monitoring — keeping ONE connection open and just spamming attempts through it can sometimes evade naive detection systems that only count NEW connections, not requests within an existing one.
- **🔵 Blue Team Perspective:** This is EXACTLY why real-world authentication systems implement account lockouts, rate limiting per-connection (not just per-IP), CAPTCHAs, and exponential backoff delays after failed attempts — a 4-digit PIN with NO rate limiting is trivially brute-forceable in seconds by any competent attacker, exactly as this level demonstrates. Any security audit finding a PIN/password system without proper attempt-limiting is a critical, must-fix finding.

---

### 4. 🚦 The Filter (Focus vs. Ignore)

- **FOCUS ON:** The `seq -w` trick for generating zero-padded number ranges, the concept of piping a LOOP's output into a SINGLE persistent `nc` connection instead of opening thousands of separate ones, and `grep -v` for filtering OUT noise to isolate a rare success message. This efficiency-first brute-forcing mindset is universally valuable.
- **IGNORE:** Don't bother trying to write this in a "fancier" scripting language (Python, Perl) right now unless you're already comfortable there — plain bash loops handle this task perfectly fine, and understanding the core LOGIC matters more than the specific language syntax.

---

### 5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)

>  اللفل ده هيعرفك على الـ brute-forcing، واحدة من أهم (وأكتر إثارة للجدل) التقنيات في الأمن الهجومي كله، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** فيه daemon (خدمة) واقفة على بورت 30002 وهتديك باسورد bandit25 لو ديتها باسورد bandit24 **وكمان** كود PIN سري مكون من 4 أرقام. مفيش طريقة تجيب بيها الـ PIN غير إنك تجرب كل الـ 10000 احتمال—وده اسمه brute-forcing. مش محتاج تفتح اتصالات جديدة كل مرة.
> - **الهدف:** لازم تكتب سكريبت **يؤتمت** إرسال باسوردك زائد كل PIN ممكن مكون من 4 أرقام (من 0000 لحد 9999) لخدمة واقفة تستنى، ويلقط/يفلتر الرد **الوحيد** اللي فعلاً فيه باسورد حقيقي بدل رسالة خطأ.
> - **الليه؟** ده أول تعامل حقيقي ليك مع **الـ brute-forcing**، واحدة من أهم التقنيات (والأكتر جدلاً) في عالم الأمن الهجومي كله. فهمك إزاي تؤتمت هجوم "جرب كل حاجة"، وبالتحديد إزاي تعمله **بكفاءة** (التلميح عن عدم الحاجة لفتح اتصالات جديدة كل مرة درس ضخم في الكفاءة)، معرفة أساسية لهجمات الباسوردات وكسر الـ PIN وأي سيناريو فيه مساحة احتمالات محدودة.
>
> **2. 💻 التنفيذ خطوة بخطوة**
>
> انت عامل login كـ `bandit24`. يلا ناخد باسوردك الحالي الأول:
>
> ```bash
> cat /etc/bandit_pass/bandit24
> ```
>
> دلوقتي، خلينا نفكر في المشكلة. الاتصال يدوياً بـ `nc` وكتابة 10000 تركيبة مختلفة بإيدك هياخد وقت للأبد حرفياً—عشان كده بالظبط محتاجين نكتب سكريبت يؤتمت العملية دي كلها.
>
> الأول، يلا نفهم أنهي فورمات الخدمة متوقعاها. الاتصال يدوياً والتجربة (أو مجرد قراءة تلميح اللفل بعناية) بيقولنا إنها متوقعة حاجة زي: `password pincode` في سطر واحد. يلا نبني سكريبت الـ brute-force بتاعنا.
>
> **الخطوة 1: ولّد كل تركيبة PIN ممكنة من 4 أرقام.**
>
> ```bash
> for i in $(seq -w 0000 9999); do echo $i; done > /tmp/pins.txt
> ```
> - `seq -w 0000 9999`: أمر `seq` بيولّد سلسلة أرقام. `-w` معناها "equal width" (عرض متساوي)، يعني بيحشو كل الأرقام بأصفار في البداية عشان يكونوا كلهم بنفس الطول (فـ `5` بتبقى `0005`، مش بس `5`—مهم جداً، لإن PIN مكون من 4 أرقام زي `0007` لازم يحتفظ بالأصفار دي في الأول). ده بيولّد كل رقم من `0000` لحد `9999`.
> - بنلف على كل واحد ونطبعه في ملف اسمه `pins.txt`، PIN واحد كل سطر—بس بصراحة، نقدر نتخطى خطوة توليد الملف دي ونعملها كلها inline جوه السكريبت الرئيسي بتاعنا، وده أنضف.
>
> **الخطوة 2: اكتب سكريبت الـ brute-force الفعلي.**
>
> يلا نعمل سكريبت بيلف على كل PIN، يبعته مع الباسورد، ويفلتر على رد ناجح:
>
> ```bash
> for i in $(seq -w 0000 9999); do
>     echo "[Password_Appears_Here] $i"
> done | nc -q1 localhost 30002 > /tmp/results24.txt
> ```
>
> خليني افككها بعناية لإن فيه حاجات كتير بتحصل:
> - `for i in $(seq -w 0000 9999); do ... done`: دي حلقة باش بتتكرر 10000 مرة، مرة لكل PIN ممكن، والمتغير `$i` بيحمل قيمة الـ PIN الحالي كل مرة.
> - `echo "[Password_Appears_Here] $i"`: جوه الحلقة، بنطبع سطر فيه باسورد bandit24 **الحقيقي** بتاعنا متبوع بمسافة والـ PIN الحالي المُجرَّب. ده الفورمات بالظبط اللي الخدمة متوقعاه.
> - `| nc -q1 localhost 30002`: هنا الجزء **الذكي** اللي بيعالج تلميح اللفل ("مش محتاج تفتح اتصالات جديدة كل مرة"). بدل ما نفتح اتصال `nc` جديد تماماً لكل واحدة من الـ 10000 محاولة (وده هيكون بطيء بشكل كارثي)، بنعمل pipe لمخرج **الحلقة كلها**—كل الـ 10000 سطر، متولدين واحد ورا التاني—جوه اتصال `nc` **واحد**. الخدمة بتقرا كل سطر لوحده، تتحقق منه، وترد، كل ده جوه اتصال واحد مستمر.
> - `-q1`: الفلاج ده بيقول لـ `nc` إنه يستنى ثانية واحدة بعد ما يستقبل EOF (نهاية الإدخال) قبل ما يقفل الاتصال أوتوماتيك. من غيرها، `nc` ممكن تفضل معلقة للأبد مستنية إدخال أكتر، أو تقفل بشكل مفاجئ قبل ما تستقبل ردود الخدمة النهائية.
> - `> /tmp/results24.txt`: بنحول **كل** المخرج (كل الـ 10000 رد، معظمهم رسايل "wrong pin" شبه بعض، وبإذن الله واحد بس فيه باسورد حقيقي) لملف، لإن عمل سكرول على 10000 سطر مخرج مباشر على التيرمينال هيبقى جنون.
>
> **الخطوة 3: فلتر النتايج عشان تلاقي الوحيد اللي نجح.**
>
> ```bash
> grep -v "Wrong" /tmp/results24.txt
> ```
> - `grep -v`: افتكر `-v` معناها "اعكس المطابقة"—بدل ما يوريك السطور اللي **فيها** النمط، يوريك السطور اللي **مفيهاش** النمط. بما إن 9999 من أصل 10000 محاولة هتفشل برسالة نوعها "Wrong password/pin"، بنفلتر بره أي سطر فيه كلمة "Wrong"، وبيفضلنا بس رسالة/رسايل النجاح النادرة اللي مش مطابقة للنمط ده.
>
> ده المفروض يسيبلك حفنة صغيرة من السطور، واحد منها هيكون بوضوح باسورد اللفل 25 بتاعك موجود قدامك.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
>
> الـ brute-forcing عبر اتصال مستمر تقنية مهمة جداً وفعلياً في كل مجالات الأمن الهجومي.
>
> - **🔴 من منظور الـ Red Team:** النمط ده بالظبط ("ولّد كل الاحتمالات، اعمل pipe عبر اتصال واحد مستمر، فلتر النتايج") هو عمود أدوات زي Hydra و Medusa وسكريبتات brute-force مخصصة مستخدمة ضد فورمات تسجيل الدخول، أو خدمات SSH، أو نقاط API بمساحة احتمالات محدودة (PINs، أكواد قصيرة، محاولات تجاوز 2FA). المهاجمين بيحسّنوا بالتحديد على إعادة استخدام الاتصال لإن كتير من الخدمات عندها rate-limiting أو مراقبة عدد الاتصالات—الاحتفاظ باتصال **واحد** مفتوح وبس إرسال المحاولات من خلاله ممكن أحياناً يهرب من أنظمة كشف ساذجة بتعد الاتصالات **الجديدة** بس، مش الطلبات جوه اتصال موجود.
> - **🔵 من منظور الـ Blue Team:** ده بالظبط سبب إن أنظمة المصادقة الحقيقية بتطبق قفل الحسابات، وrate limiting لكل اتصال (مش بس لكل IP)، والـ CAPTCHAs، وتأخير متزايد (exponential backoff) بعد المحاولات الفاشلة—PIN من 4 أرقام **من غير** حد للمحاولات قابل للكسر بـ brute-force في ثواني من أي مهاجم كفء، بالظبط زي ما اللفل ده بيوريك. أي تدقيق أمني بيلاقي نظام PIN/باسورد من غير تحديد صحيح للمحاولات ده اكتشاف حرج ولازم يتصلح فوراً.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** حيلة `seq -w` لتوليد نطاقات أرقام بأصفار حشو، ومفهوم عمل pipe لمخرج **حلقة** جوه اتصال `nc` **واحد** مستمر بدل فتح آلاف اتصالات منفصلة، و `grep -v` لفلترة الضوضاء عشان تعزل رسالة نجاح نادرة. عقلية الـ brute-forcing دي اللي بتركز على الكفاءة الأول قيّمة في كل مكان.
> - **تجاهل:** متضيعش وقتك تحاول تكتب ده بلغة "أفخم" (Python أو Perl) دلوقتي إلا لو انت مرتاح فيها أصلاً—حلقات الباش العادية بتؤدي المهمة دي كويس تماماً، وفهم **المنطق** الأساسي أهم من صيغة اللغة المحددة.
