### 🏴‍☠️ [Level 13 &rarr; Level 14]

**1. 🎯 The Objective & The "Why"**
- **The Question:** The password for the next level is stored in `/etc/bandit_pass/bandit14` and can only be read by user `bandit14`. This time, instead of getting a password directly, you get a private SSH key that you need to use to log into the next level.
- **The Goal:** You've been handed an SSH **private key file** sitting in your home directory. You need to use that key to authenticate as `bandit14` via SSH — no password typing involved this time. Once you're logged in as `bandit14`, you'll be able to read the password file directly since permissions will allow it for that specific user.
- **The Why:** This is your first real-world introduction to **key-based SSH authentication**, which is THE standard, professional way servers are accessed in the real world — far more secure than passwords. Nearly every production server, cloud instance, and CI/CD pipeline uses SSH keys instead of passwords. If you don't master this, you cannot function as a real sysadmin, DevOps engineer, or penetration tester.

**2. 💻 The Execution (Step-by-Step)**
You're logged in as `bandit13`. Let's see what we've been given:

```bash
ls -la
```
You'll see a file, probably named `sshkey.private`. Let's peek at its content just to understand what an SSH private key even looks like:

```bash
cat sshkey.private
```
You'll see a block of text starting with something like `-----BEGIN OPENSSH PRIVATE KEY-----` and ending with `-----END OPENSSH PRIVATE KEY-----`, with a bunch of scrambled Base64-looking gibberish in between. This is a cryptographic private key — think of it as a physical house key, except mathematically generated. Whoever holds this exact key can prove their identity to any server that has the matching "public key" installed, without ever needing to type a password.

Now let's use it to connect. The `ssh` command has a specific flag for supplying a key file:

```bash
ssh -i sshkey.private bandit14@localhost
```
Let's break this down:
- `ssh`: Same command we've used since Level 0.
- `-i`: Stands for "identity file." This tells `ssh`, "Don't ask me for a password — instead, use THIS specific key file to prove who I am." This is the core mechanic of key-based authentication.
- `sshkey.private`: The path to our private key file.
- `bandit14@localhost`: We're connecting AS user `bandit14`. And notice we use `localhost` instead of the usual `bandit.labs.overthewire.org` — that's because we're already physically sitting on the Bandit server (having SSH'd in as bandit13), so we can just connect to "this same machine" using `localhost`, which always refers to your own current machine.

**You might hit an error here**, and this level SPECIFICALLY warns you to read error messages carefully. A very common one looks like:
```
Permissions 0644 for 'sshkey.private' are too open.
It is required that your private key files are NOT accessible by others.
```

You might ask, "Why does SSH care about file permissions on my OWN key?" I'll tell you exactly why: SSH is designed with a strict security philosophy — if your private key file can be read by OTHER users on the same system (not just you), then SSH considers that key COMPROMISED/untrustworthy, because anyone else could've copied it and could now impersonate you. It REFUSES to use a key with overly permissive permissions, as a built-in safety mechanism.

The fix is to lock down the file's permissions so ONLY you (the owner) can read/write it:

```bash
chmod 600 sshkey.private
```
- `chmod`: Stands for "change mode" — it's the command used to modify file permissions in Linux.
- `600`: This is a permission code in octal notation. Breaking it down: the first digit (6) applies to the OWNER, the second digit (0) applies to the GROUP, and the third digit (0) applies to EVERYONE ELSE. A value of `6` means "read + write" permission (4 for read + 2 for write = 6), and `0` means "no permission at all." So `600` translates to: "Owner can read and write this file. Nobody else — not the group, not the world — can do anything with it." This is exactly the strict, private permission SSH demands for key files.

Now retry the connection:
```bash
ssh -i sshkey.private bandit14@localhost
```
This time, it should connect successfully without asking for a password — the key itself proves your identity.

Once logged in as `bandit14`, you can now read the level 14 password directly:
```bash
cat /etc/bandit_pass/bandit14
```
Since you're literally logged in AS `bandit14` now, the file's permissions allow you to read it.

**3. 🌍 Real-World & Tactical Application**
SSH key-based authentication is the backbone of secure remote access in virtually every professional tech environment.

- **🔴 Red Team Perspective:** SSH private keys are one of the MOST valuable pieces of loot an attacker can find on a compromised machine. If an attacker finds an unprotected private key (especially one that hasn't been password-protected with a passphrase), they can use it to pivot and access OTHER servers that trust that key — this is a classic lateral movement technique in real breaches. Attackers specifically hunt for `.ssh/` folders, `id_rsa` files, and any `*.pem`/`*.key` files during post-exploitation for exactly this reason. This level also teaches you the permission-checking behavior that YOU as an attacker need to replicate/respect when reusing stolen keys.
- **🔵 Blue Team Perspective:** This exact permission check (`Permissions 0644... too open`) built into SSH is a genuine, real security control — it's why administrators enforce strict `chmod 600` policies on all private keys, and why security audits specifically flag any key files with loose permissions (readable by group or world) as critical findings. Organizations also enforce passphrase-protected keys and short-lived certificate-based SSH access (instead of long-lived static keys) precisely to reduce the blast radius if a key IS somehow stolen.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** The `ssh -i keyfile user@host` syntax for key-based login, and `chmod 600 keyfile` as the mandatory permission fix for ANY private key you ever handle. Also internalize WHY SSH enforces this (compromised key = anyone with read access can impersonate you). Reading error messages carefully (as the level explicitly hints) is a permanent professional habit.
- **IGNORE:** Don't worry about the deeper cryptographic math behind how public/private key pairs actually work (RSA algorithm internals, prime factorization, etc.) right now — that's a separate cryptography deep-dive. Just understand the practical mechanic: key file + correct permissions = passwordless authentication.

---

> **5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)**
>
> بص  ، اللفل ده هيعرفك على الطريقة الاحترافية الحقيقية اللي بيتم بيها الدخول للسيرفرات في الواقع، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** الباسورد بتاع اللفل الجاي متخزن في `/etc/bandit_pass/bandit14` وميقدرش يقراه غير اليوزر `bandit14`. المرة دي بدل ما تاخد باسورد مباشر، هتاخد مفتاح SSH خاص لازم تستخدمه عشان تعمل login للفل الجاي.
> - **الهدف:** اديناك ملف **مفتاح SSH خاص (private key)** موجود في الـ home directory بتاعك. لازم تستخدم المفتاح ده عشان تثبت هويتك كـ `bandit14` عن طريق SSH—من غير ما تكتب باسورد خالص المرة دي. بمجرد ما تعمل login كـ `bandit14`، هتقدر تقرا ملف الباسورد مباشرة لإن الصلاحيات هتسمح بكده لليوزر ده بالتحديد.
> - **الليه؟** ده أول تعريف حقيقي ليك بـ **المصادقة بالمفتاح (key-based SSH authentication)**، وهي الطريقة القياسية والاحترافية اللي بيتم بيها الوصول للسيرفرات في الواقع—أأمن بكتير من الباسوردات. تقريباً كل سيرفر إنتاج، وكل خدمة كلاود، وكل خط CI/CD بيستخدم مفاتيح SSH بدل الباسوردات. لو ما اتقنتش الحاجة دي، مش هتقدر تشتغل كأدمن حقيقي أو مهندس DevOps أو بنتستر.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit13`. يلا نشوف اديناك ايه:
>
> ```bash
> ls -la
> ```
> هتلاقي ملف، غالباً اسمه `sshkey.private`. يلا نشوف محتواه بس عشان نفهم شكل المفتاح الخاص بتاع SSH:
>
> ```bash
> cat sshkey.private
> ```
> هتشوف كتلة نص بتبدأ بحاجة زي `-----BEGIN OPENSSH PRIVATE KEY-----` وبتنتهي بـ `-----END OPENSSH PRIVATE KEY-----`، وبينهم خليط نصي شكله Base64 متلخبط. ده مفتاح تشفيري خاص—فكر فيه كأنه مفتاح بيت فعلي، بس متولد رياضياً. أي حد ماسك المفتاح ده بالظبط يقدر يثبت هويته لأي سيرفر عنده "المفتاح العام" المطابق، من غير ما يحتاج يكتب باسورد أبداً.
>
> يلا نستخدمه عشان نتصل. أمر `ssh` عنده فلاج مخصص عشان تمرر ملف مفتاح:
>
> ```bash
> ssh -i sshkey.private bandit14@localhost
> ```
> خليني افككها:
> - `ssh`: نفس الأمر اللي بنستخدمه من اللفل صفر.
> - `-i`: معناها "identity file" (ملف الهوية). ده بيقول لـ `ssh`، "متسألنيش عن باسورد—استخدم ملف المفتاح ده بالتحديد عشان تثبت هويتي." دي الآلية الأساسية للمصادقة بالمفتاح.
> - `sshkey.private`: مسار ملف المفتاح الخاص بتاعنا.
> - `bandit14@localhost`: بنتصل **كـ** يوزر `bandit14`. ولاحظ إننا بنستخدم `localhost` بدل `bandit.labs.overthewire.org` المعتادة—وده لإننا أصلاً واقفين فعلياً على سيرفر Bandit (بعد ما عملنا SSH كـ bandit13)، فنقدر بس نتصل بـ "نفس الجهاز ده" باستخدام `localhost`، اللي دايماً بيشير للجهاز الحالي بتاعك نفسه.
>
> **ممكن يظهرلك error هنا**، واللفل ده بالتحديد بينبهك إنك تقرا رسايل الأخطاء بعناية. رسالة شائعة جداً بتبقى شبه:
> ```
> Permissions 0644 for 'sshkey.private' are too open.
> It is required that your private key files are NOT accessible by others.
> ```
>
> هتقولي "ليه SSH مهتمة بصلاحيات ملف على مفتاحي أنا؟" هقولك بالظبط ليه: SSH متصممة على فلسفة أمان صارمة—لو ملف المفتاح الخاص بتاعك ممكن يتقرا من يوزرز تانيين على نفس النظام (مش انت بس)، فالـ SSH بتعتبر المفتاح ده **مخترق/غير موثوق**، لإن أي حد تاني كان ممكن ينسخه ويقدر ينتحل شخصيتك دلوقتي. هي بترفض تستخدم مفتاح صلاحياته مفتوحة زيادة، كآلية أمان مدمجة.
>
> الحل إنك تقفل صلاحيات الملف عشان انت بس (المالك) تقدر تقراه/تكتب فيه:
>
> ```bash
> chmod 600 sshkey.private
> ```
> - `chmod`: معناها "change mode" (تغيير الوضع)—الأمر المستخدم لتعديل صلاحيات الملفات في اللينكس.
> - `600`: ده كود صلاحيات بترميز الثماني (octal). لو فككناه: الرقم الأول (6) بيخص **المالك**، الرقم التاني (0) بيخص **الجروب**، والرقم الثالث (0) بيخص **أي حد تاني**. القيمة `6` معناها صلاحية "قراءة + كتابة" (4 للقراءة + 2 للكتابة = 6)، و `0` معناها "مفيش صلاحية خالص". فـ `600` معناها: "المالك يقدر يقرا ويكتب في الملف ده. محدش تاني—لا الجروب ولا العالم كله—يقدر يعمل أي حاجة بيه." ده بالظبط الصلاحية الصارمة والخاصة اللي SSH بتطلبها لملفات المفاتيح.
>
> دلوقتي جرب الاتصال تاني:
> ```bash
> ssh -i sshkey.private bandit14@localhost
> ```
> المرة دي، المفروض يتصل بنجاح من غير ما يسألك عن باسورد—المفتاح نفسه بيثبت هويتك.
>
> بمجرد ما تعمل login كـ `bandit14`، تقدر دلوقتي تقرا باسورد اللفل الرابع عشر مباشرة:
> ```bash
> cat /etc/bandit_pass/bandit14
> ```
> بما إنك فعلاً عامل login كـ `bandit14` دلوقتي، صلاحيات الملف بتسمحلك تقراه.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> المصادقة بمفتاح SSH هي عمود الوصول الآمن عن بعد في تقريباً كل بيئة تقنية احترافية.
>
> - **🔴 من منظور الـ Red Team:** مفاتيح SSH الخاصة من أثمن الكنوز اللي ممكن الهاكر يلاقيها على جهاز مخترق. لو المهاجم لقى مفتاح خاص مش محمي (خصوصاً واحد مش محمي بـ passphrase)، يقدر يستخدمه عشان يتحرك ويوصل لسيرفرات **تانية** بتثق في المفتاح ده—دي تقنية كلاسيكية للحركة الجانبية (lateral movement) في اختراقات حقيقية. المهاجمين بيدوروا بالتحديد على فولدرات `.ssh/` وملفات `id_rsa` وأي ملفات `*.pem`/`*.key` أثناء ما بعد الاختراق لنفس السبب ده بالظبط. اللفل ده كمان بيعلمك سلوك فحص الصلاحيات اللي انت كمهاجم لازم تحترمه/تكرره لما تعيد استخدام مفاتيح مسروقة.
> - **🔵 من منظور الـ Blue Team:** فحص الصلاحيات ده بالظبط (`Permissions 0644... too open`) المدمج في SSH هو ضابط أمان حقيقي وفعلي—عشان كده الأدمنز بيفرضوا سياسات `chmod 600` صارمة على كل المفاتيح الخاصة، وعشان كده التدقيقات الأمنية بتعلم على أي ملف مفتاح صلاحياته مفتوحة (قابل للقراءة من الجروب أو العالم) كاكتشاف حرج. المؤسسات كمان بتفرض مفاتيح محمية بـ passphrase ووصول SSH بشهادات قصيرة العمر (بدل مفاتيح ثابتة طويلة العمر) بالتحديد عشان تقلل حجم الضرر لو المفتاح اتسرق بأي شكل.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** صيغة `ssh -i keyfile user@host` للدخول بالمفتاح، و `chmod 600 keyfile` كإصلاح صلاحيات إجباري لأي مفتاح خاص هتتعامل معاه في حياتك. وكمان استوعب **ليه** SSH بتفرض ده (مفتاح مخترق = أي حد عنده صلاحية قراءة يقدر ينتحل شخصيتك). قراءة رسايل الأخطاء بعناية (زي ما اللفل بيلمح صراحة) عادة احترافية دائمة.
> - **تجاهل:** متقلقش من الرياضيات التشفيرية العميقة وراء إزاي أزواج المفاتيح العامة/الخاصة بتشتغل فعلياً (تفاصيل خوارزمية RSA، تحليل الأعداد الأولية، الخ) دلوقتي—ده موضوع تشفير منفصل وأعمق. بس افهم الآلية العملية: ملف مفتاح + صلاحيات صحيحة = مصادقة من غير باسورد.
