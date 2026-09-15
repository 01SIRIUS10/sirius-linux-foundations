### 🏴‍☠️ [Level 18 &rarr; Level 19]

**1. 🎯 The Objective & The "Why"**
- **The Question:** The password for the next level is stored in a file `readme` in the home directory. Unfortunately, someone has modified `.bashrc` to log you out when you log in with SSH.
- **The Goal:** Normally, you'd just SSH in and casually browse around with `ls` and `cat`. But this time, the SERVER is intentionally weaponized against you — the moment your SSH session starts, a startup script kicks you out before you even get a working shell prompt. You need to find a way to run commands WITHOUT ever getting a normal interactive shell.
- **The Why:** This teaches you about **shell startup scripts** (`.bashrc`, `.bash_profile`, `.profile`) and, more importantly, how to BYPASS a broken or hostile login environment. This is a genuinely important real-world skill — sometimes you'll SSH into a server with a corrupted profile, a restricted shell, or (in a red team context) a deliberately booby-trapped account, and you need alternate ways to still execute commands.

**2. 💻 The Execution (Step-by-Step)**
Let's start from scratch. You attempt the normal login:

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220
```
You type the password, and instead of landing on a nice `bandit18@bandit:~$` prompt, you immediately see something like `Byebye!` and get disconnected right back to your own local machine. You might ask, "What just happened? Did I type the password wrong?" No — I'll tell you exactly what's going on: every time you log in via SSH, the server automatically runs certain startup files to configure your shell environment (things like setting your prompt, aliases, environment variables). One of these files — `.bashrc` — has been deliberately edited by the level designers to contain a command that immediately terminates your session with an "echo Byebye! ; exit" the SECOND you connect. Your login is technically succeeding, but the shell commits suicide instantly afterward.

So how do we get around a hostile startup script? The trick is: **we tell SSH to run a SPECIFIC command directly, instead of dropping us into an interactive shell that would trigger `.bashrc`.**

```bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
```

Let's break down why this works:
- Normally, when you run plain `ssh user@host`, SSH starts up a full interactive login shell for you — and it's THAT shell startup process which reads and executes `.bashrc`, triggering the booby trap.
- But when you provide `ssh` with an EXTRA argument after the hostname (in this case, the string `"cat readme"`), SSH interprets this completely differently: instead of giving you an interactive shell, it says "connect, authenticate, run EXACTLY this one command remotely, print its output back to me, and then disconnect." This is called a **non-interactive SSH command execution** — you never actually get a full interactive shell session, so the `.bashrc` booby trap never gets triggered at all (or even if it partially runs, the specific command you asked for still executes and its output still gets returned to you before the kill-switch fires).

When prompted, type the bandit18 password, and instead of getting kicked out, you'll see the CONTENTS of the `readme` file printed directly to your terminal — which is your level 19 password — and then the connection closes on its own (since we only asked for one single command to run, not an ongoing session).

**3. 🌍 Real-World & Tactical Application**
Non-interactive SSH command execution is a genuinely essential technique for both admins and attackers.

- **🔴 Red Team Perspective:** Attackers use this EXACT technique when they've compromised credentials for an account with a restricted, broken, or monitored shell — running `ssh user@host "malicious_command_here"` lets them execute a single payload without ever spawning a full interactive session, which can sometimes evade logging/monitoring systems specifically watching for interactive shell sessions. It's also heavily used in legitimate automation (deploying scripts across fleets of servers via tools like Ansible, which under the hood does exactly this — SSHing in and running specific commands non-interactively, at scale).
- **🔵 Blue Team Perspective:** Defenders should be aware that a hostile or misconfigured `.bashrc`/`.profile` is both a legitimate security control (used here to deliberately lock down a training account) AND a potential attack vector — if an attacker gains write access to a victim's `.bashrc`, they can plant persistence (a backdoor command that runs every single time that user logs in). Security audits should routinely check shell startup files for unauthorized modifications, and monitoring solutions should flag unusual non-interactive SSH command executions as potential signs of automated attack tooling or living-off-the-land techniques.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** The syntax `ssh user@host "command"` for running a single remote command without triggering a full interactive shell/its startup scripts. Understand WHY this works — it bypasses `.bashrc` execution entirely by not requesting an interactive session.
- **IGNORE:** Don't worry about actually trying to FIX or edit the broken `.bashrc` file itself right now (though we'll touch on that skill later) — for this specific level, the goal is simply bypassing it to grab the password, not repairing it.

---

> **5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)**
>
>   اللفل ده هيعلمك إزاي تتحايل على سيرفر مسلح ضدك، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** الباسورد بتاع اللفل الجاي متخزن في ملف `readme` في الـ home directory. للأسف، حد عدّل ملف `.bashrc` عشان يطردك بره في اللحظة اللي تعمل فيها SSH.
> - **الهدف:** عادةً، هتعمل SSH وتتفسح جوه بـ `ls` و `cat` براحتك. بس المرة دي، السيرفر متسلح ضدك عمداً—في اللحظة اللي جلسة الـ SSH بتاعتك بتبدأ، سكريبت بدء تشغيل بيطردك برة قبل ما توصل لبرومبت شِل شغال أصلاً. لازم تلاقي طريقة تنفذ بيها أوامر من غير ما توصل لشِل تفاعلي عادي خالص.
> - **الليه؟** ده بيعلمك عن **سكريبتات بدء تشغيل الشِل** (`.bashrc` و `.bash_profile` و `.profile`)، والأهم من كده، إزاي **تتحايل على بيئة تسجيل دخول معطلة أو عدائية**. دي مهارة مهمة جداً في الواقع—أحياناً هتعمل SSH لسيرفر فيه profile تالف، أو شِل مقيّد، أو (في سياق Red Team) حساب متفخخ عمداً، ولازم تلاقي طرق بديلة عشان تنفذ أوامر برضو.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> يلا نبدأ من الصفر. تحاول تعمل login عادي:
>
> ```bash
> ssh bandit18@bandit.labs.overthewire.org -p 2220
> ```
> بتكتب الباسورد، وبدل ما توصل لبرومبت لطيف زي `bandit18@bandit:~$`، بتشوف على طول حاجة زي `Byebye!` وبتتفصل رجوع لجهازك المحلي. هتقولي "ايه اللي حصل؟ كتبت الباسورد غلط؟" لأ—هقولك بالظبط اللي بيحصل: كل مرة بتعمل login عن طريق SSH، السيرفر بيشغل أوتوماتيك ملفات بدء تشغيل معينة عشان يظبط بيئة الشِل بتاعتك (حاجات زي ظبط البرومبت، الاختصارات، متغيرات البيئة). واحد من الملفات دي—`.bashrc`—اتعدل عمداً من مصممي اللفل عشان يحتوي على أمر بينهي جلستك فوراً بـ "echo Byebye! ; exit" **لحظة** ما تتصل. الـ login بتاعك بينجح تقنياً، بس الشِل بينتحر فوراً بعديها.
>
> فإزاي نتحايل على سكريبت بدء تشغيل عدائي؟ الحيلة إن: **نقول لـ SSH ينفذ أمر محدد مباشرة، بدل ما يحطنا في شِل تفاعلي كان هيشغّل الـ `.bashrc`.**
>
> ```bash
> ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
> ```
>
> خليني افكك ليه ده بيشتغل:
> - عادةً، لما تشغل `ssh user@host` عادية، الـ SSH بيبدأ شِل تسجيل دخول تفاعلي كامل ليك—وعملية بدء تشغيل الشِل دي بالتحديد هي اللي بتقرا وتنفذ `.bashrc`، وبتفجر الفخ.
> - بس لما تديلـ `ssh` argument **إضافي** بعد اسم المضيف (في الحالة دي، النص `"cat readme"`)، الـ SSH بتفسر الموضوع بشكل مختلف تماماً: بدل ما تديك شِل تفاعلي، بتقول "اتصل، اتحقق من الهوية، شغّل بالظبط الأمر ده لوحده عن بعد، اطبع مخرجه لي، وبعدين اقفل الاتصال." ده اسمه **تنفيذ أمر SSH غير تفاعلي**—انت أصلاً مبتوصلش لجلسة شِل تفاعلية كاملة، فالفخ بتاع `.bashrc` مش بيتفجر خالص (أو حتى لو اتشغل جزئياً، الأمر المحدد اللي طلبته بينفذ ومخرجه بيترجعلك قبل ما مفتاح القتل يشتغل).
>
> لما يطلب منك الباسورد، اكتب باسورد bandit18، وبدل ما تتطرد، هتشوف **محتوى ملف الـ readme** مطبوع مباشرة على التيرمينال بتاعك—وده باسورد اللفل 19 بتاعك—وبعدين الاتصال بيتقفل لوحده (لإننا طلبنا أمر واحد بس يتنفذ، مش جلسة مستمرة).
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> تنفيذ أوامر SSH غير التفاعلي تقنية أساسية فعلياً للأدمنز والمهاجمين على حد سواء.
>
> - **🔴 من منظور الـ Red Team:** الهاكرز بيستخدموا التقنية دي بالظبط لما يكونوا اخترقوا بيانات دخول لحساب عنده شِل مقيّد، أو معطل، أو مراقب—تشغيل `ssh user@host "malicious_command_here"` بيخليهم ينفذوا حمولة واحدة من غير ما يفتحوا جلسة تفاعلية كاملة، وده ممكن أحياناً يهرب من أنظمة التسجيل/المراقبة اللي بتراقب بالتحديد جلسات الشِل التفاعلية. دي كمان بتتستخدم بكثافة في الأتمتة الشرعية (نشر سكريبتات على أساطيل من السيرفرات بأدوات زي Ansible، اللي تحت الغطا بتعمل بالظبط كده—تعمل SSH وتشغل أوامر محددة بشكل غير تفاعلي، على نطاق واسع).
> - **🔵 من منظور الـ Blue Team:** المدافعين لازم يكونوا واعيين إن `.bashrc`/`.profile` عدائي أو معطل هو ضابط أمني شرعي (متستخدم هنا عمداً عشان تقفل حساب تدريبي) **وكمان** مسار هجوم محتمل—لو هاكر وصل لصلاحية كتابة على `.bashrc` بتاع ضحية، يقدر يزرع persistence (أمر باكدور بيتشغل كل مرة اليوزر ده بيعمل login). التدقيقات الأمنية لازم تفحص بشكل دوري ملفات بدء تشغيل الشِل على أي تعديلات غير مصرح بها، وحلول المراقبة لازم تعلّم على تنفيذات SSH غير تفاعلية غريبة كعلامات محتملة لأدوات هجوم آلية أو تقنيات "العيش على الأرض" (living-off-the-land).
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** صيغة `ssh user@host "command"` لتنفيذ أمر بعيد واحد من غير ما تشغل شِل تفاعلي كامل/سكريبتات بدء تشغيله. افهم **ليه** ده بيشتغل—بيتحايل على تنفيذ `.bashrc` بالكامل عن طريق إنه مش بيطلب جلسة تفاعلية أصلاً.
> - **تجاهل:** متقلقش من محاولة فعلياً **تصلح** أو تعدل ملف الـ `.bashrc` المعطل نفسه دلوقتي (رغم إننا هنلمس المهارة دي بعدين)—للفل ده بالتحديد، الهدف بس إنك تتحايل عليه عشان تاخد الباسورد، مش تصلحه.
