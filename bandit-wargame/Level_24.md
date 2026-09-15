### 🏴‍☠️ [Level 23 &rarr; Level 24]

**1. 🎯 The Objective & The "Why"**
- **The Question:** A program is running automatically at regular intervals from cron. Look in `/etc/cron.d/` for the configuration and see what command is being executed. NOTE: This level requires you to create your own FIRST shell script — a big step! NOTE 2: your shell script gets DELETED once executed, so keep a copy around.
- **The Goal:** This time, the cron job doesn't just read a fixed script — it executes ANY script it finds sitting inside a specific directory, as long as that script passes certain checks, and it runs it AS bandit24. Your job is to WRITE your own malicious-but-legitimate script, drop it in the right place with the right permissions, and let cron execute it FOR you, capturing the bandit24 password in the process.
- **The Why:** This is a MASSIVE milestone — your first time actually WRITING offensive tooling instead of just reading/running existing files. This exact technique (dropping a script into a directory that a privileged process automatically executes) is one of the most common real-world privilege escalation patterns in existence. If you understand this level deeply, you understand the core concept behind dozens of real CVEs.

---

### 2. 💻 The Execution (Step-by-Step)

You're logged in as `bandit23`. Let's find the cron configuration first:

```bash
cat /etc/cron.d/cronjob_bandit24
```
This will show something like:
```
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh
```
Same pattern as before — runs every minute as `bandit24`. Let's read that script:

```bash
cat /usr/bin/cronjob_bandit24.sh
```
The content will look something like this:
```bash
#!/bin/bash
myname=$(whoami)
cd /var/spool/$myname/foo || exit
echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ]; then
        echo "Handling $i"
        owner="$(stat --format "%U" "./$i")"
        if [ "${owner}" = "bandit23" ]; then
            timeout -s 9 60 ./$i
        fi
        rm -f "./$i"
    fi
done
```

Let's trace through this LINE BY LINE, exactly like we practiced last level — you need to understand this deeply, so let's go slow:
- `myname=$(whoami)`: Since cron runs this AS bandit24, `myname` becomes `bandit24`.
- `cd /var/spool/$myname/foo`: The script moves INTO the folder `/var/spool/bandit24/foo`. This is a shared drop-folder — think of it like an inbox where anyone can leave files.
- The `for i in * .*;` loop: This iterates over EVERY file in that folder, including hidden ones (`.*`).
- `owner="$(stat --format "%U" "./$i")"`: For each file found, it checks WHO OWNS that file using `stat` (a command that reports detailed file metadata, and `--format "%U"` specifically asks it to output just the OWNER's username).
- `if [ "${owner}" = "bandit23" ];`: THIS is the critical security gate — the script ONLY executes the file if its owner is specifically `bandit23` (that's YOU!). This is a safety check meant to prevent random strangers from planting scripts, but since WE are bandit23, we're EXACTLY who's allowed through this gate.
- `timeout -s 9 60 ./$i`: If the owner check passes, it EXECUTES the file, but wrapped in `timeout` (which kills it after 60 seconds if it hangs, using signal 9/SIGKILL as a forceful kill).
- `rm -f "./$i"`: After running (or even if the owner check failed), the file gets DELETED — this is exactly why the level warns you to keep a copy of your own script, since it self-destructs after execution.

So the plan is crystal clear: write a script, as bandit23, that reads bandit24's password and dumps it somewhere WE (bandit23) can read, drop it into `/var/spool/bandit24/foo`, wait up to a minute for cron to pick it up, and read the result.

**Step 1: Write your script.**
```bash
echo '#!/bin/bash' > /tmp/mygrab.sh
echo 'cat /etc/bandit_pass/bandit24 > /tmp/bandit24_password.txt' >> /tmp/mygrab.sh
echo 'chmod 777 /tmp/bandit24_password.txt' >> /tmp/mygrab.sh
```
- `echo '...' > /tmp/mygrab.sh`: The FIRST `echo` uses `>` (single arrow), which CREATES a new file (or completely overwrites an existing one) with the given text as content. We're writing the "shebang" line (`#!/bin/bash`), which tells the system "interpret this file using bash."
- `echo '...' >> /tmp/mygrab.sh`: The SUBSEQUENT `echo` commands use `>>` (double arrow), which APPENDS to the end of the file instead of overwriting it. This builds our script line by line.
- Line 2 of our script: reads bandit24's password file (since the script will run AS bandit24, it has permission) and writes it into a file in `/tmp`.
- Line 3: makes that output file world-readable (`chmod 777` means everyone can read/write/execute — overkill for security, but perfectly fine for our purposes here since we just need to READ it as bandit23 afterward).

**Step 2: Make your script executable.**
```bash
chmod +x /tmp/mygrab.sh
```
- `chmod +x`: Adds the "execute" permission to the file. Without this, even a perfectly-written script can't actually be RUN — the `timeout ./$i` line in the cron script needs execute permission to work.

**Step 3: Copy your script into the cron's watched folder.**
```bash
cp /tmp/mygrab.sh /var/spool/bandit24/foo/mygrab.sh
```
- We use `cp` (copy) instead of `mv` (move) specifically BECAUSE we know the original gets deleted by the cron script after execution — copying preserves our original in `/tmp` as a backup, exactly like the level's hint warned us to do.

**Step 4: Wait, then check your result.**
Cron runs every minute, so wait a bit (60-90 seconds is safe), then:
```bash
cat /tmp/bandit24_password.txt
```
This reveals your level 24 password.

---

### 3. 🌍 Real-World & Tactical Application

Dropping executable scripts into a privileged-process-watched directory is a textbook real-world privilege escalation vector.

- **🔴 Red Team Perspective:** This EXACT pattern — "find a directory that a privileged cron job/service automatically executes files from" — is a genuine, documented privilege escalation technique catalogued extensively on sites like HackTricks and GTFOBins. Attackers specifically hunt for world-writable directories referenced in cron configurations, or watch folders monitored by automated systems, and drop malicious payloads there to get code execution as a more privileged user. This is precisely how you'd escalate from a low-privilege web shell to root on many misconfigured real servers.
- **🔵 Blue Team Perspective:** Never design automated systems that execute files from directories writable by lower-privileged users — this is a critical security anti-pattern. If a "drop folder" concept is genuinely needed, it should have STRICT permission checks (like this level actually demonstrates — the `owner` check), ideally combined with additional validation like checksums, digital signatures, or an explicit allowlist of approved scripts, not just an ownership check alone (which, as we just proved, is trivially satisfiable by the intended lower-privileged user).

---

### 4. 🚦 The Filter (Focus vs. Ignore)

- **FOCUS ON:** The full workflow of writing a script with `echo >` (create) and `echo >>` (append), making it executable with `chmod +x`, and understanding WHY the cron script's ownership check matters. This "read the automation logic, then craft input that satisfies it" methodology is universally applicable.
- **IGNORE:** Don't worry about learning a proper text editor like `vim` or `nano` for this specific level — the `echo >>` chaining method is perfectly fine for a short script like this. Save deep editor mastery for when you're writing longer, more complex scripts later.

---

### 5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)

> اللفل ده لحظة تاريخية—أول مرة تكتب سكريبت هجومي بإيدك، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** فيه برنامج بيتشغل أوتوماتيك على فترات منتظمة من الـ cron. شوف في `/etc/cron.d/` وشوف أنهي أمر بيتنفذ. ملحوظة: اللفل ده هيخليك تكتب أول سكريبت شِل ليك—خطوة كبيرة! ملحوظة تانية: السكريبت بتاعك بيتمسح بعد ما يتنفذ، فخلي نسخة منه عندك.
> - **الهدف:** المرة دي، مهمة الـ cron مش بس بتقرا سكريبت ثابت—هي بتنفذ **أي** سكريبت تلاقيه في فولدر معين، طالما عدى فحوصات معينة، وبتشغله بصفة bandit24. مهمتك إنك تكتب سكريبت خاص بيك (خبيث بس شرعي)، تحطه في المكان الصح بالصلاحيات الصح، وتسيب الـ cron ينفذه نيابة عنك، وتلقط باسورد bandit24 في العملية.
> - **الليه؟** دي محطة **ضخمة**—أول مرة فعلياً تكتب أداة هجومية بدل ما تقرا/تشغل ملفات موجودة بس. التقنية دي بالظبط (رمي سكريبت في فولدر عملية مرفوعة الصلاحيات بتنفذه أوتوماتيك) واحدة من أكتر أنماط تصعيد الصلاحيات شيوعاً في الواقع. لو فهمت اللفل ده كويس، فاهمت الفكرة الأساسية وراء عشرات الـ CVEs الحقيقية.
>
> **2. 💻 التنفيذ خطوة بخطوة**
>
> انت عامل login كـ `bandit23`. يلا نلاقي إعدادات الـ cron الأول:
>
> ```bash
> cat /etc/cron.d/cronjob_bandit24
> ```
> هيوريك حاجة زي:
> ```
> * * * * * bandit24 /usr/bin/cronjob_bandit24.sh
> ```
> نفس النمط زي قبل—بيشتغل كل دقيقة بصفة `bandit24`. يلا نقرا السكريبت ده:
>
> ```bash
> cat /usr/bin/cronjob_bandit24.sh
> ```
> المحتوى هيبان شبه كده:
> ```bash
> #!/bin/bash
> myname=$(whoami)
> cd /var/spool/$myname/foo || exit
> echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
> for i in * .*;
> do
>     if [ "$i" != "." -a "$i" != ".." ]; then
>         echo "Handling $i"
>         owner="$(stat --format "%U" "./$i")"
>         if [ "${owner}" = "bandit23" ]; then
>             timeout -s 9 60 ./$i
>         fi
>         rm -f "./$i"
>     fi
> done
> ```
>
> خليني اتتبع السكريبت ده **سطر بسطر**، بالظبط زي ما اتدربنا اللفل اللي فات—محتاج تفهمه بعمق، يلا نمشي بهدوء:
> - `myname=$(whoami)`: بما إن الـ cron بيشغل ده بصفة bandit24، المتغير `myname` بيبقى `bandit24`.
> - `cd /var/spool/$myname/foo`: السكريبت بيدخل جوه فولدر `/var/spool/bandit24/foo`. ده فولدر مشترك للرمي—فكر فيه كصندوق بريد أي حد يقدر يسيب فيه ملفات.
> - حلقة `for i in * .*;`: دي بتمر على **كل** ملف في الفولدر ده، حتى المخفي (`.*`).
> - `owner="$(stat --format "%U" "./$i")"`: لكل ملف بيلاقيه، بيتحقق **مين مالكه** باستخدام `stat` (أمر بيوري بيانات ميتا تفصيلية عن الملف، و `--format "%U"` بالتحديد بيطلب منه يطبع اسم اليوزر المالك بس).
> - `if [ "${owner}" = "bandit23" ];`: ده **البوابة الأمنية الحرجة**—السكريبت بينفذ الملف **بس** لو مالكه بالتحديد هو `bandit23` (أنت بالذات!). ده فحص أمان القصد منه منع أي حد غريب من زرع سكريبتات، بس بما إننا احنا bandit23، إحنا بالظبط اللي مسموحلنا نعدي البوابة دي.
> - `timeout -s 9 60 ./$i`: لو فحص المالك عدى، بينفذ الملف، بس ملفوف في `timeout` (اللي بيقتله بعد 60 ثانية لو علّق، باستخدام إشارة 9/SIGKILL كقتل إجباري).
> - `rm -f "./$i"`: بعد التشغيل (أو حتى لو فحص المالك فشل)، الملف بيتمسح—وده بالظبط سبب تحذير اللفل إنك تحتفظ بنسخة من سكريبتك، لإنه بينفجر بعد التنفيذ.
>
> فالخطة واضحة جداً: اكتب سكريبت، بصفة bandit23، بيقرا باسورد bandit24 ويطبعه في مكان **إحنا** (bandit23) نقدر نقراه، حطه في `/var/spool/bandit24/foo`، استنى لحد دقيقة عشان الـ cron يمسكه، واقرا النتيجة.
>
> **الخطوة 1: اكتب سكريبتك.**
> ```bash
> echo '#!/bin/bash' > /tmp/mygrab.sh
> echo 'cat /etc/bandit_pass/bandit24 > /tmp/bandit24_password.txt' >> /tmp/mygrab.sh
> echo 'chmod 777 /tmp/bandit24_password.txt' >> /tmp/mygrab.sh
> ```
> - `echo '...' > /tmp/mygrab.sh`: الـ `echo` **الأول** بيستخدم `>` (سهم واحد)، وده بينشئ ملف جديد (أو يمحي واحد موجود بالكامل) بالنص المكتوب كمحتوى. إحنا بنكتب سطر الـ "shebang" (`#!/bin/bash`)، اللي بيقول للنظام "فسّر الملف ده باستخدام باش."
> - `echo '...' >> /tmp/mygrab.sh`: أوامر الـ `echo` **اللي بعدها** بتستخدم `>>` (سهمين)، وده بيضيف في آخر الملف بدل ما يمحيه. كده بنبني السكريبت بتاعنا سطر سطر.
> - السطر التاني من السكريبت: بيقرا ملف باسورد bandit24 (بما إن السكريبت هيشتغل بصفة bandit24، عنده الصلاحية)، ويكتبه في ملف جوه `/tmp`.
> - السطر الثالث: بيخلي ملف المخرج ده قابل للقراءة من الجميع (`chmod 777` معناها الجميع يقدر يقرا/يكتب/ينفذ—مبالغ فيه أمنياً، بس تمام تماماً لغرضنا هنا بما إننا بس محتاجين نقراه بصفة bandit23 بعدين).
>
> **الخطوة 2: خلي سكريبتك قابل للتنفيذ.**
> ```bash
> chmod +x /tmp/mygrab.sh
> ```
> - `chmod +x`: بيضيف صلاحية "التنفيذ" للملف. من غيرها، حتى سكريبت مكتوب صح تماماً مش هيقدر فعلياً **يتشغل**—سطر `timeout ./$i` في سكريبت الـ cron محتاج صلاحية تنفيذ عشان يشتغل.
>
> **الخطوة 3: انسخ سكريبتك جوه فولدر الـ cron المراقب.**
> ```bash
> cp /tmp/mygrab.sh /var/spool/bandit24/foo/mygrab.sh
> ```
> - بنستخدم `cp` (نسخ) بدل `mv` (نقل) بالتحديد **لإننا عارفين** إن النسخة الأصلية هتتمسح بواسطة سكريبت الـ cron بعد التنفيذ—النسخ بيحافظ على نسختنا الأصلية في `/tmp` كنسخة احتياطية، بالظبط زي ما تلميح اللفل حذرنا نعمل.
>
> **الخطوة 4: استنى، بعدين شوف نتيجتك.**
>
> الـ cron بيشتغل كل دقيقة، فاستنى شوية (60-90 ثانية آمنة)، وبعدين:
> ```bash
> cat /tmp/bandit24_password.txt
> ```
> ده هيكشف باسورد اللفل 24 بتاعك.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
>
> رمي سكريبتات تنفيذية في فولدر مراقب من عملية مرفوعة الصلاحيات نمط تصعيد صلاحيات كلاسيكي وحقيقي.
>
> - **🔴 من منظور الـ Red Team:** النمط ده بالظبط—"دور على فولدر عملية cron/خدمة مرفوعة الصلاحيات بتنفذ ملفات منه أوتوماتيك"—تقنية تصعيد صلاحيات موثقة وحقيقية جداً موجودة بكثرة على مواقع زي HackTricks و GTFOBins. المهاجمين بيدوروا بالتحديد على فولدرات قابلة للكتابة من الجميع مذكورة في إعدادات cron، أو فولدرات مراقبة من أنظمة آلية، ويرموا حمولات خبيثة فيها عشان ياخدوا تنفيذ كود بصفة يوزر أرفع صلاحية. دي بالظبط الطريقة اللي تقدر تصعّد بيها من webshell بصلاحيات محدودة لـ root على كتير من السيرفرات الحقيقية المظبوطة غلط.
> - **🔵 من منظور الـ Blue Team:** أبداً متصمّمش أنظمة آلية بتنفذ ملفات من فولدرات قابلة للكتابة من يوزرز أقل صلاحية—ده نمط أمني خاطئ حرج. لو فكرة "فولدر رمي" محتاجة فعلاً، لازم يكون عندها فحوصات صلاحيات **صارمة** (زي ما اللفل ده فعلياً بيوريك—فحص الـ owner)، ويفضل تتجمع مع تحقق إضافي زي checksums أو توقيعات رقمية أو قائمة سماح صريحة للسكريبتات المعتمدة، مش بس فحص ملكية لوحده (اللي، زي ما أثبتنا لتونا، سهل جداً إشباعه من نفس اليوزر الأقل صلاحية المقصود).
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** الـ workflow الكامل لكتابة سكريبت بـ `echo >` (إنشاء) و `echo >>` (إضافة)، وجعله قابل للتنفيذ بـ `chmod +x`، وفهم **ليه** فحص الملكية في سكريبت الـ cron مهم. المنهجية دي "اقرا منطق الأتمتة، بعدين اصنع مدخل يشبعه" قابلة للتطبيق في كل حتة.
> - **تجاهل:** متقلقش من تعلم محرر نصوص حقيقي زي `vim` أو `nano` للفل ده بالتحديد—طريقة سلسلة `echo >>` كافية تماماً لسكريبت قصير زي ده. وفر الإتقان العميق للمحررات لما هتكتب سكريبتات أطول وأعقد بعدين.
