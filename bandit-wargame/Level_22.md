### 🏴‍☠️ [Level 21 → Level 22]

**1. 🎯 The Objective & The "Why"**
- **The Question:** A program is running automatically at regular intervals from `cron`, the time-based job scheduler. Look in `/etc/cron.d/` for the configuration and see what command is being executed.
- **The Goal:** Instead of manually finding and running an executable, you need to investigate a **scheduled task system** (cron) that's automatically running commands behind the scenes on a timer. You need to read the cron configuration to figure out WHAT script is being run, and then examine THAT script to find where it's putting your next password.
- **The Why:** `cron` is the beating heart of Linux automation — nearly every server on Earth has scheduled tasks running (backups, log rotations, health checks, cleanup scripts). Understanding how to read cron configurations is critical because attackers use cron for **persistence** (planting a backdoor that re-executes itself even after a reboot), and defenders need to know how to audit cron jobs to spot malicious ones.

---

### 💻 The Execution (Step-by-Step)
You're logged in as `bandit21`. Let's go straight to where cron configurations live:

```bash
ls -la /etc/cron.d/
```
- `/etc/cron.d/`: This is a standard Linux directory specifically meant for holding cron job configuration files. Think of it as a "scheduling calendar" folder — every file inside describes a task, WHEN it should run, and WHO should run it as.

You'll see one or more files in there. Let's read the content of whichever one looks relevant (often it'll have a name matching this level, like `cronjob_bandit22`):

```bash
cat /etc/cron.d/cronjob_bandit22
```
The output will look something like this:
```
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh
```

Let's decode this cron syntax piece by piece, because it looks cryptic at first:
- The first FIVE fields (`* * * * *`) represent TIMING: minute, hour, day-of-month, month, and day-of-week, in that exact order. An asterisk `*` in any field means "every possible value" — so `* * * * *` altogether means "run this EVERY SINGLE MINUTE, forever, no restrictions." (In real production cron jobs, you'd typically see specific numbers here, like `30 2 * * *` meaning "run at 2:30 AM every day.")
- `bandit22`: This is the CRITICAL detail — this field specifies WHICH USER ACCOUNT the command should run AS. This is the SUID-like magic happening here: even though this cron file is readable by you (bandit21), the actual SCRIPT gets executed with bandit22's privileges, because that's what's configured here.
- `/usr/bin/cronjob_bandit22.sh`: This is the actual command/script being executed, every single minute, as user bandit22.

Now let's read the actual script being run:

```bash
cat /usr/bin/cronjob_bandit22.sh
```
The content will likely reveal something like:
```bash
#!/bin/bash
chmod 644 /etc/bandit_pass/bandit22
cat /etc/bandit_pass/bandit22 > /tmp/some_file_name.txt
chmod 644 /tmp/some_file_name.txt
```
This script (running as bandit22, remember!) is literally taking bandit22's password file, adjusting its permissions to be world-readable (`644` means owner can read/write, everyone else can only read), and DUMPING that password into a file inside `/tmp` — a directory that's readable by everyone on the system.

So all you need to do now is read that temp file directly:

```bash
cat /tmp/some_file_name.txt
```
(Replace with the exact filename the script actually specified.) This will show you the level 22 password, sitting there in plain view, thanks to a poorly-secured automated script.

---

### 🌍 Real-World & Tactical Application
Cron job auditing is a routine but critical task in real system administration and security work.

- **🔴 Red Team Perspective:** Cron is one of the MOST common persistence mechanisms attackers use after gaining access to a Linux box — planting a malicious cron entry (`* * * * * root /tmp/.hidden/backdoor.sh`) ensures their access survives reboots and periodic cleanup, since the malicious command re-executes automatically forever. Attackers also specifically look for MISCONFIGURED cron jobs like this exact scenario — a script running with elevated privileges that writes sensitive output to a world-readable location, which is a genuine, common real-world vulnerability class.
- **🔵 Blue Team Perspective:** System administrators must regularly audit `/etc/cron.d/`, `/etc/crontab`, and individual users' `crontab -l` entries for unauthorized or suspicious jobs — especially ones running as `root` or other privileged accounts, or ones pointing to scripts in world-writable locations (which an attacker could tamper with to hijack the next scheduled execution). This exact vulnerability class (privileged script leaking secrets to `/tmp`) is a classic finding in security code reviews — the fix is simple: never write sensitive data to world-readable locations, and always set proper restrictive permissions BEFORE writing sensitive content, not after.

---

### 🚦 The Filter (Focus vs. Ignore)
- **FOCUS ON:** Knowing that `/etc/cron.d/` (and `/etc/crontab`) is where system-wide scheduled tasks live, and understanding the 5-field time syntax plus the "run as user X" field. Also lock in the habit of "trace the chain" — config file points to a script, script does something, follow it step by step.
- **IGNORE:** Don't stress about memorizing every possible cron time syntax combination (specific minute ranges, step values like `*/5`, etc.) right now — you just need to recognize `* * * * *` means "every minute" for this level. Deep cron scheduling syntax mastery can come later when you're actually writing your own scheduled jobs.

---

### 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)

> اللفل ده هيعرفك على قلب الأتمتة في اللينكس كله—الـ cron، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** فيه برنامج بيتشغل أوتوماتيك على فترات منتظمة من الـ `cron`، جدول المهام المعتمد على الوقت. شوف في `/etc/cron.d/` عشان تلاقي الإعدادات وتشوف أنهي أمر بيتنفذ.
> - **الهدف:** بدل ما تدور يدوياً على برنامج تنفيذي وتشغله، لازم تحقق في **نظام مهام مجدولة** (cron) بيشغل أوامر أوتوماتيك في الخلفية على توقيت معين. لازم تقرا إعدادات الـ cron عشان تعرف أنهي سكريبت بيتشغل، وبعدين تفحص السكريبت ده عشان تلاقي فين بيحط باسوردك الجاي.
> - **الليه؟** الـ `cron` هو قلب الأتمتة في اللينكس—تقريباً كل سيرفر على الكوكب عنده مهام مجدولة شغالة (نسخ احتياطي، تدوير لوجز، فحوصات صحة، سكريبتات تنظيف). فهمك إزاي تقرا إعدادات الـ cron حاجة حرجة لإن الهاكرز بيستخدموا الـ cron لـ **الاستمرارية (persistence)** (زرع باكدور بيعيد تشغيل نفسه حتى بعد إعادة تشغيل الجهاز)، والمدافعين محتاجين يعرفوا إزاي يدققوا على مهام الـ cron عشان يكتشفوا الخبيثة منها.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit21`. يلا نروح مباشرة لمكان إعدادات الـ cron:
>
> ```bash
> ls -la /etc/cron.d/
> ```
> - `/etc/cron.d/`: ده فولدر قياسي في اللينكس مخصص لتخزين ملفات إعدادات مهام الـ cron. فكر فيه كفولدر "روزنامة جدولة"—كل ملف جواه بيوصف مهمة، وامتى المفروض تشتغل، ومين المفروض يشغلها.
>
> هتلاقي ملف أو أكتر جوه الفولدر. يلا نقرا محتوى أي واحد شكله مناسب (غالباً هيكون اسمه متطابق مع اللفل ده، زي `cronjob_bandit22`):
>
> ```bash
> cat /etc/cron.d/cronjob_bandit22
> ```
> المخرج هيبان شبه كده:
> ```
> * * * * * bandit22 /usr/bin/cronjob_bandit22.sh
> ```
>
> خليني افك صيغة الـ cron دي حتة حتة، لإنها شكلها غامض أول وهلة:
> - **الخمس خانات الأولى** (`* * * * *`) بتمثل **التوقيت**: الدقيقة، الساعة، يوم الشهر، الشهر، ويوم الأسبوع، بالترتيب ده بالظبط. علامة النجمة `*` في أي خانة معناها "أي قيمة ممكنة"—فـ `* * * * *` كلها مع بعض معناها "شغل ده كل دقيقة، للأبد، من غير أي قيود." (في مهام cron حقيقية في الإنتاج، غالباً هتشوف أرقام محددة هنا، زي `30 2 * * *` معناها "شغل الساعة 2:30 صباحاً كل يوم.")
> - `bandit22`: ده التفصيلة **الحرجة**—الخانة دي بتحدد **أي حساب يوزر** المفروض الأمر يتشغل بصفته. ده السحر الشبيه بالـ SUID اللي بيحصل هنا: رغم إن الملف ده مقروء منك انت (bandit21)، السكريبت الفعلي بيتنفذ بصلاحيات bandit22، لإن ده اللي متظبط هنا.
> - `/usr/bin/cronjob_bandit22.sh`: ده الأمر/السكريبت الفعلي اللي بيتنفذ، كل دقيقة، بصفة يوزر bandit22.
>
> يلا نقرا السكريبت الفعلي اللي بيتشغل:
>
> ```bash
> cat /usr/bin/cronjob_bandit22.sh
> ```
> المحتوى غالباً هيكشف حاجة زي:
> ```bash
> #!/bin/bash
> chmod 644 /etc/bandit_pass/bandit22
> cat /etc/bandit_pass/bandit22 > /tmp/some_file_name.txt
> chmod 644 /tmp/some_file_name.txt
> ```
> السكريبت ده (شغال بصفة bandit22، افتكر!) بيروح حرفياً ياخد ملف باسورد bandit22، يظبط صلاحياته عشان يبقى قابل للقراءة من الجميع (`644` معناها المالك يقدر يقرا ويكتب، الباقي يقدر يقرا بس)، ويطبع الباسورد ده في ملف جوه `/tmp`—فولدر مقروء من أي حد على النظام.
>
> فكل اللي محتاجه دلوقتي إنك تقرا الملف المؤقت ده مباشرة:
>
> ```bash
> cat /tmp/some_file_name.txt
> ```
> (غيّر بالاسم الفعلي اللي السكريبت حدده.) ده هيوريك باسورد اللفل 22، موجود قدامك بوضوح، بفضل سكريبت آلي مؤمّن بشكل ضعيف.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> تدقيق مهام الـ cron مهمة روتينية بس حرجة في إدارة الأنظمة والأمن الحقيقي.
>
> - **🔴 من منظور الـ Red Team:** الـ cron واحدة من أكتر آليات الاستمرارية استخداماً من المهاجمين بعد ما يوصلوا لجهاز لينكس—زرع إدخال cron خبيث (`* * * * * root /tmp/.hidden/backdoor.sh`) بيضمن إن وصولهم يفضل موجود حتى بعد إعادة التشغيل والتنظيف الدوري، لإن الأمر الخبيث بيعيد تنفيذ نفسه أوتوماتيك للأبد. المهاجمين كمان بيدوروا بالتحديد على مهام cron مظبوطة غلط زي السيناريو ده بالظبط—سكريبت شغال بصلاحيات مرفوعة وبيكتب مخرج حساس في مكان مقروء من الجميع، وده نوع ثغرة حقيقي وشائع جداً في الواقع.
> - **🔵 من منظور الـ Blue Team:** أدمنز الأنظمة لازم يدققوا بشكل دوري على `/etc/cron.d/` و `/etc/crontab` وإدخالات `crontab -l` بتاعة كل يوزر عشان يلاقوا مهام غير مصرح بها أو مشبوهة—خصوصاً اللي شغالة بصفة `root` أو حسابات مرفوعة تانية، أو اللي بتشير لسكريبتات في أماكن قابلة للكتابة من الجميع (اللي هاكر ممكن يتلاعب فيها عشان يخطف التنفيذ المجدول الجاي). نوع الثغرة ده بالظبط (سكريبت مرفوع الصلاحيات بيسرب أسرار لـ `/tmp`) اكتشاف كلاسيكي في مراجعات الكود الأمنية—الحل بسيط: أبداً متكتبش بيانات حساسة في أماكن مقروءة من الجميع، ودايماً ظبط صلاحيات مقيدة صح **قبل** ما تكتب المحتوى الحساس، مش بعده.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** معرفتك إن `/etc/cron.d/` (و `/etc/crontab`) هما مكان المهام المجدولة على مستوى النظام كله، وفهم صيغة الخمس خانات الوقتية زائد خانة "اشتغل بصفة يوزر X". وكمان ثبّت عادة "تتبع السلسلة"—ملف الإعدادات بيشير لسكريبت، السكريبت بيعمل حاجة، تابعها خطوة بخطوة.
> - **تجاهل:** متتوتّرش من حفظ كل تركيبة ممكنة لصيغة توقيت cron (نطاقات دقايق محددة، قيم خطوة زي `*/5`، الخ) دلوقتي—بس محتاج تتعرف إن `* * * * *` معناها "كل دقيقة" للفل ده. الإتقان العميق لصيغة جدولة cron ممكن ييجي لاحقاً لما فعلاً تكتب مهامك المجدولة الخاصة بيك.
