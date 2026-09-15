### 🏴‍☠️ [Level 26 &rarr; Level 27]

**1. 🎯 The Objective & The "Why"**
- **The Question:** Good job getting a shell! Now hurry and grab the password for bandit27!
- **The Goal:** You've just broken out of a restricted `showtext` environment into a real bash shell running as `bandit26`. Now you need to figure out HOW `bandit26` — a user who's normally SUPPOSED to be locked into that restricted shell — can escalate to `bandit27`. The hint "hurry" is deliberate: there's likely a time-limited or race-condition element here, or simply a SUID binary you need to find and use immediately before you get kicked out again.
- **The Why:** This level reinforces that gaining a shell isn't the END goal — it's just a STEPPING STONE. In real engagements, popping a shell is only step one; you then need to enumerate your NEW environment for the next opportunity to escalate further. This "land, look around, escalate" loop is the core rhythm of every real penetration test.

---

### 2. 💻 The Execution (Step-by-Step)

Picking up right where the last level left off — you've just typed `!bash` inside the `more` pager and dropped into a live shell as `bandit26`. Let's immediately look around before anything can interrupt us:

```bash
ls -la
```
- We run this the instant we land, because we genuinely don't know how stable this shell session is — the underlying account is DESIGNED to kick people out, so "hurry" isn't just flavor text, it's a real warning.

In bandit26's home directory, you'll spot a binary, typically named something like `bandit27-do`. Let's check its permissions:

```bash
ls -la bandit27-do
```
You'll see that telltale `s` in the permission string again (`-rwsr-x---`), meaning this is a **SUID binary**, and checking further, you'll find it's owned by `bandit27`. Exactly like Level 19's `bandit20-do`, this program is designed to let bandit26 run a command with bandit27's elevated privileges.

Let's run it bare first to see its usage instructions:

```bash
./bandit27-do
```
It'll likely print something like:
```
Run a command as another user.
Example: ./bandit27-do id
```

Now we use it to read bandit27's password file directly:

```bash
./bandit27-do cat /etc/bandit_pass/bandit27
```
- `./bandit27-do`: We're invoking the SUID program (which runs with bandit27's privileges due to the SUID bit).
- `cat /etc/bandit_pass/bandit27`: The command we're passing to it — reading a file that only bandit27 (or root) would normally be allowed to read.

This immediately prints the level 27 password to your screen, since the program executes that `cat` command as bandit27, and bandit27 absolutely has permission to read its own password file.

---

### 3. 🌍 Real-World & Tactical Application

This level is a perfect illustration of the "shell is just the beginning" mentality that defines real offensive security work.

- **🔴 Red Team Perspective:** After landing ANY foothold — whether through a web shell, a phishing payload, or (like here) a restricted account breakout — the very first thing an experienced attacker does is enumerate for privilege escalation vectors: SUID binaries, writable cron jobs, sudo misconfigurations, kernel exploits. Automated tools like `linpeas.sh` or `LinEnum.sh` exist specifically to speed-run this exact enumeration process the moment you get ANY shell, because time on target often matters (the longer you're there, the higher the chance of detection).
- **🔵 Blue Team Perspective:** This reinforces WHY defense-in-depth matters — even if an attacker breaks out of one restricted account, the NEXT layer of security (proper SUID auditing, principle of least privilege on custom "do" scripts, monitoring for unusual process execution chains) should ideally stop or at least detect the escalation attempt. Security teams should specifically flag any custom SUID wrapper scripts (like these `*-do` binaries) as high-priority audit targets, since they're explicitly designed to cross privilege boundaries.

---

### 4. 🚦 The Filter (Focus vs. Ignore)

- **FOCUS ON:** The reflex of running `ls -la` IMMEDIATELY upon landing any new shell, and recognizing the recurring `s` permission bit pattern for SUID binaries as your instant next move. This level is really just testing if you internalized the Level 19/20 lesson well enough to apply it instantly without hand-holding.
- **IGNORE:** Don't overthink the "hurry" wording as some complex timing puzzle — in practice, the shell from the `more` breakout is stable enough to work calmly through these steps; it's more of a narrative nudge than a literal ticking clock.

---

### 5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)

> اللفل ده هيثبتلك إن الوصول لشِل مش نهاية الحدوتة—ده مجرد نقطة انطلاق، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** مبروك وصلت لشِل! دلوقتي استعجل وهات باسورد bandit27!
> - **الهدف:** لسه واصل من الشِل المقيد `showtext` لشِل باش حقيقي شغال بصفة `bandit26`. دلوقتي لازم تعرف إزاي `bandit26`—يوزر المفروض يكون محبوس في الشِل المقيد ده—يقدر يصعّد لـ `bandit27`. كلمة "استعجل" مقصودة: غالباً فيه عنصر محدود بالوقت أو race condition، أو ببساطة برنامج SUID لازم تلاقيه وتستخدمه بسرعة قبل ما تتطرد تاني.
> - **الليه؟** اللفل ده بيأكد إن الوصول لشِل مش **الهدف النهائي**—ده مجرد **حجر انطلاق**. في العمليات الحقيقية، فتح شِل هو بس الخطوة الأولى؛ بعدين لازم تحصر بيئتك **الجديدة** عشان تلاقي الفرصة الجاية للتصعيد أكتر. حلقة "انزل، اتفرج، صعّد" دي هي الإيقاع الأساسي لأي اختبار اختراق حقيقي.
>
> **2. 💻 التنفيذ خطوة بخطوة**
>
> هنكمل من حيث اللفل اللي فات وقف—انت لسه كتبت `!bash` جوه الـ pager `more` ونزلت شِل شغال بصفة `bandit26`. يلا نشوف اللي حوالينا فوراً قبل ما أي حاجة تقاطعنا:
>
> ```bash
> ls -la
> ```
> - بنشغل ده في اللحظة اللي بننزل فيها، لإننا فعلياً مش عارفين جلسة الشِل دي مستقرة قد إيه—الحساب الأساسي **متصمم** يطرد الناس، فكلمة "استعجل" مش مجرد كلام حماسي، هي تحذير حقيقي.
>
> في فولدر bandit26 الرئيسي، هتلاقي برنامج تنفيذي، غالباً اسمه شبه `bandit27-do`. يلا نفحص صلاحياته:
>
> ```bash
> ls -la bandit27-do
> ```
> هتشوف نفس الحرف `s` المميز في سلسلة الصلاحيات تاني (`-rwsr-x---`)، يعني ده **برنامج SUID**، وبفحص أكتر، هتلاقيه مملوك لـ `bandit27`. بالظبط زي `bandit20-do` بتاع اللفل 19، البرنامج ده مصمم عشان يسمح لـ bandit26 يشغل أمر بصلاحيات bandit27 المرفوعة.
>
> يلا نشغله عاري الأول عشان نشوف تعليمات استخدامه:
>
> ```bash
> ./bandit27-do
> ```
> غالباً هيطبع حاجة زي:
> ```
> Run a command as another user.
> Example: ./bandit27-do id
> ```
>
> دلوقتي نستخدمه عشان نقرا ملف باسورد bandit27 مباشرة:
>
> ```bash
> ./bandit27-do cat /etc/bandit_pass/bandit27
> ```
> - `./bandit27-do`: بنستدعي برنامج الـ SUID (اللي بيشتغل بصلاحيات bandit27 بفضل بت الـ SUID).
> - `cat /etc/bandit_pass/bandit27`: الأمر اللي بنمرره ليه—بنقرا ملف عادةً بس bandit27 (أو root) مسموحله يقراه.
>
> ده هيطبع باسورد اللفل 27 مباشرة على شاشتك، لإن البرنامج بينفذ أمر الـ `cat` ده بصفة bandit27، وbandit27 أكيد عنده صلاحية يقرا ملف باسورده هو.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
>
> اللفل ده مثال مثالي على عقلية "الشِل مجرد بداية" اللي بتميز شغل الأمن الهجومي الحقيقي.
>
> - **🔴 من منظور الـ Red Team:** بعد ما تحصل على أي موطئ قدم—سواء عن طريق webshell، أو حمولة فيشينج، أو (زي هنا) كسر من حساب مقيد—أول حاجة هاكر محترف بيعملها إنه يحصر مسارات تصعيد الصلاحيات: برامج SUID، مهام cron قابلة للكتابة، إعدادات sudo خاطئة، ثغرات كيرنل. أدوات آلية زي `linpeas.sh` أو `LinEnum.sh` موجودة بالتحديد عشان تسرّع عملية الحصر دي بمجرد ما توصل لأي شِل، لإن الوقت على الهدف غالباً مهم (كل ما تقعد أكتر، كل ما فرصة اكتشافك تزيد).
> - **🔵 من منظور الـ Blue Team:** ده بيأكد **ليه** الدفاع متعدد الطبقات مهم—حتى لو هاكر كسر من حساب مقيد واحد، الطبقة **الجاية** من الأمان (تدقيق SUID صحيح، مبدأ أقل صلاحية على سكريبتات "do" المخصصة، مراقبة سلاسل تنفيذ العمليات الغريبة) لازم توقف أو على الأقل تكتشف محاولة التصعيد. فرق الأمن لازم تعلّم بالتحديد على أي سكريبتات wrapper SUID مخصصة (زي برامج الـ `*-do` دي) كأهداف تدقيق عالية الأولوية، لإنها مصممة صراحة عشان تعدي حدود الصلاحيات.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** رد الفعل التلقائي بتشغيل `ls -la` **فوراً** بمجرد ما تنزل أي شِل جديد، والتعرف على نمط بت الصلاحية `s` المتكرر لبرامج SUID كخطوتك الجاية الفورية. اللفل ده أساساً بيختبر هل استوعبت درس اللفل 19/20 كويس بما فيه الكفاية إنك تطبقه فوراً من غير مساعدة.
> - **تجاهل:** متبالغش في التفكير في كلمة "استعجل" كأنها لغز توقيت معقد—عملياً، الشِل الناتج من كسر الـ `more` مستقر بما فيه الكفاية إنك تشتغل بهدوء على الخطوات دي؛ هي أقرب لدفعة سردية من كونها ساعة تعد فعلياً.
