### 🏴‍☠️ [Level 16 &rarr; Level 17]

**1. 🎯 The Objective & The "Why"**
- **The Question:** The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. You first need to find which of these ports have a server listening, then determine which ones speak SSL/TLS and which don't. Only ONE server gives back the next credentials; the rest just echo back whatever you send them.
- **The Goal:** This is a full mini penetration test compressed into one level. You need to (1) scan a range of 1000 ports to find which ones are actually open/listening, (2) test each open port to determine if it requires encryption or not, and (3) try submitting your password to each candidate until you find the ONE that actually gives you something useful back — the rest are decoys that just echo your input back at you.
- **The Why:** This is your first real taste of **port scanning and service enumeration**, the literal FIRST step of ANY penetration test or red team engagement. Before you can attack anything, you need to know what's actually running and listening. `nmap` is quite possibly the single most iconic and universally-used tool in all of offensive security. If you learn nothing else from this entire wargame besides basic `nmap` usage, you've already gained something invaluable.

**2. 💻 The Execution (Step-by-Step)**
You're logged in as `bandit16`. Let's grab your current password first:

```bash
cat /etc/bandit_pass/bandit16
```

**Step 1: Scan the port range to find which ones are actually open.**

```bash
nmap -p 31000-32000 localhost
```
- `nmap`: Stands for "Network Mapper." It's THE standard tool for discovering what hosts are alive on a network and what services/ports are open on them.
- `-p 31000-32000`: The `-p` flag specifies which PORTS to scan. Instead of scanning nmap's default set of ~1000 common ports, we're telling it explicitly, "only check this exact range, from 31000 all the way to 32000," matching exactly what the level told us.
- `localhost`: The target we're scanning — in this case, the same machine we're sitting on.

Running this will take a few seconds and then output something like:
```
PORT      STATE SERVICE
31046/tcp open  unknown
31518/tcp open  unknown
31691/tcp open  unknown
31790/tcp open  unknown
31960/tcp open  unknown
```
This tells you EXACTLY which ports in that range have a service actively listening ("open") — everything else in that 1000-port range is closed/not responding. Now you have your short list of actual candidates instead of blindly guessing across 1000 possibilities.

**Step 2: Use nmap's service/version detection to guess which ones use SSL/TLS.**

We can get MORE detail with a smarter scan:
```bash
nmap -p 31000-32000 -sV localhost
```
- `-sV`: Stands for "service version detection." Instead of just telling you a port is "open," nmap actively probes it and tries to fingerprint WHAT service/software is actually running there, including detecting if it's speaking SSL/TLS.

This might show you something like:
```
31790/tcp open  ssl/unknown
31960/tcp open  echo
```
Ports flagged with `ssl/` in their service name are your clue that they need `openssl s_client` instead of plain `nc`.

**Step 3: Test each candidate port manually, one by one.**

For any port that DIDN'T show `ssl` in the scan, try plain netcat first:
```bash
nc localhost 31046
```
Type your current password and hit Enter. If the response is just your OWN password echoed straight back at you, that's a decoy — this port isn't the right one. Close it (`Ctrl+C`) and move to the next candidate.

For any port that DID show `ssl` in the scan (or if plain `nc` gives you garbled/no response), use the encrypted client instead:
```bash
openssl s_client -connect localhost:31790
```
Type your password after the handshake output settles and you see the `---` prompt.

**You repeat this process for every port on your candidate list**, systematically ruling out the ones that just echo your input, until you find the ONE port that responds with something genuinely different — actual new credentials (this level gives you BOTH a username and password for the next level, since the level description says "credentials," plural, not just a password).

Once you find the correct port and submit your password, it'll respond with something like:
```
Correct!
BANDIT17USERNAME BANDIT17PASSWORD
```
Save both values — you'll need the username too this time, since apparently it's not simply "bandit17" as expected (this level sometimes hands you a differently-named account, so pay close attention to what's actually returned).

**3. 🌍 Real-World & Tactical Application**
Port scanning with `nmap` combined with manual service probing is THE textbook opening move of every professional penetration test.

- **🔴 Red Team Perspective:** Every single engagement starts with reconnaissance, and `nmap` is the industry-standard tool for that first phase — mapping out what's alive, what ports are open, and what services/versions are running (which then get cross-referenced against known vulnerabilities/CVEs). Distinguishing encrypted vs. plaintext services (exactly like this level trains) determines your entire attack approach — you can't sniff plaintext credentials off an encrypted channel, but you absolutely can off an unencrypted one, which is why finding "the odd service out" matters so much operationally.
- **🔵 Blue Team Perspective:** Defenders run their OWN internal `nmap` scans regularly (as part of attack surface management) to catch services that shouldn't be exposed, unpatched software versions nmap can fingerprint, or forgotten "test" services left running on weird high ports that developers spun up and forgot about — exactly the kind of decoy/echo services you saw in this level. Intrusion Detection Systems (IDS) also specifically watch for the TELLTALE PATTERN of an nmap scan itself (rapid sequential connection attempts across a port range) as an early-warning sign that someone is actively reconnaissance-ing your network.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** The `nmap -p [range] [target]` syntax for targeted port scanning, and `-sV` for service detection. Also lock in the methodology: scan first (find what's alive) → identify (encrypted or not) → interact (with the right tool for each) → verify (is this the real one, or an echo decoy?). This four-step loop IS penetration testing in miniature.
- **IGNORE:** Don't worry about nmap's hundreds of other scan types and flags (`-sS`, `-sU`, `-A`, timing templates, NSE scripts) right now — that's a massive topic for dedicated nmap study later. For this level, `-p` and `-sV` are all you genuinely need.

---

> **5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)**
>
> اللفل ده هيخليك تعمل اختبار اختراق مصغر بإيدك، وهتتعرف على أشهر أداة في عالم الأمن الهجومي كله، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** بيانات الدخول (credentials) بتاعة اللفل الجاي تقدر تجيبها لما تبعت باسورد اللفل الحالي لبورت في نطاق من 31000 لـ 32000 على localhost. الأول لازم تلاقي أنهي بورتات من دول فيها سيرفر شغال وواقف يستنى، وبعدين تحدد أنهي منهم بيتكلم SSL/TLS وأنهي لأ. فيه سيرفر واحد بس هيديك بيانات الدخول الجديدة، الباقي هيرجعلك بس نفس اللي بعتهوله.
> - **الهدف:** ده اختبار اختراق مصغر كامل متضغط في لفل واحد. لازم (1) تعمل فحص لنطاق 1000 بورت عشان تلاقي أنهي منهم فعلاً مفتوح/واقف يستنى، (2) تختبر كل بورت مفتوح عشان تحدد هل محتاج تشفير ولا لأ، و(3) تجرب تبعت الباسورد بتاعك لكل مرشح لحد ما تلاقي الواحد اللي فعلاً بيرجعلك حاجة مفيدة—الباقي مصايد بس بترجعلك نفس اللي بعتهوله.
> - **الليه؟** ده أول تذوق حقيقي ليك لـ **فحص البورتات وحصر الخدمات**، وهي حرفياً **أول خطوة** في أي اختبار اختراق أو عملية Red Team. قبل ما تقدر تهاجم أي حاجة، لازم تعرف فعلاً إيه اللي شغال وواقف يستنى. `nmap` هي بشكل شبه مؤكد الأداة الأشهر والأكثر استخداماً في عالم الأمن الهجومي كله. لو مش هتتعلم حاجة تانية من اللعبة دي كلها غير استخدام `nmap` الأساسي، تكون بالفعل كسبت حاجة تستاهل.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit16`. يلا ناخد الباسورد الحالي بتاعك الأول:
>
> ```bash
> cat /etc/bandit_pass/bandit16
> ```
>
> **الخطوة 1: افحص نطاق البورتات عشان تلاقي أنهي منهم مفتوح فعلاً.**
>
> ```bash
> nmap -p 31000-32000 localhost
> ```
> - `nmap`: معناها "Network Mapper" (رسّام الشبكة). هي الأداة القياسية لاكتشاف الأجهزة الشغالة على شبكة وأنهي خدمات/بورتات مفتوحة عليها.
> - `-p 31000-32000`: الفلاج `-p` بيحدد أنهي **بورتات** هنفحصها. بدل ما نفحص مجموعة nmap الافتراضية من الألف بورت الشائعين، بنقولها صراحة، "افحص بس النطاق ده بالظبط، من 31000 لحد 32000"، بالظبط زي ما اللفل قالنا.
> - `localhost`: الهدف اللي بنفحصه—في الحالة دي، نفس الجهاز اللي واقفين عليه.
>
> تشغيل الأمر ده هياخد ثواني وبعدين يطلعلك حاجة زي:
> ```
> PORT      STATE SERVICE
> 31046/tcp open  unknown
> 31518/tcp open  unknown
> 31691/tcp open  unknown
> 31790/tcp open  unknown
> 31960/tcp open  unknown
> ```
> ده بيقولك بالظبط أنهي بورتات في النطاق ده فيها خدمة شغالة وواقفة تستنى ("open")—أي حاجة تانية في الألف بورت دول مقفولة/مش بترد. دلوقتي عندك قايمة قصيرة من المرشحين الفعليين بدل ما تخمن عشوائي على 1000 احتمال.
>
> **الخطوة 2: استخدم كشف الخدمة/الإصدار بتاع nmap عشان تخمن أنهي منهم بيستخدم SSL/TLS.**
>
> نقدر نجيب تفاصيل أكتر بفحص أذكى:
> ```bash
> nmap -p 31000-32000 -sV localhost
> ```
> - `-sV`: معناها "service version detection" (كشف إصدار الخدمة). بدل ما بس تقولك البورت "مفتوح"، nmap بتفحصه فعلياً وتحاول تعرف بالضبط أنهي خدمة/برنامج شغال هناك، وفيها كمان اكتشاف لو بيتكلم SSL/TLS.
>
> ده ممكن يوريك حاجة زي:
> ```
> 31790/tcp open  ssl/unknown
> 31960/tcp open  echo
> ```
> البورتات اللي معلم عليها `ssl/` في اسم الخدمة دي إشارتك إنهم محتاجين `openssl s_client` بدل `nc` العادية.
>
> **الخطوة 3: اختبر كل بورت مرشح يدوياً، واحد واحد.**
>
> لأي بورت **مالوش** علامة `ssl` في الفحص، جرب netcat العادية الأول:
> ```bash
> nc localhost 31046
> ```
> اكتب باسوردك الحالي ودوس Enter. لو الرد كان بس باسوردك الخاص راجعلك زي ما هو، ده معناها مصيدة—البورت ده مش الصح. اقفله (`Ctrl+C`) وانتقل للمرشح اللي بعده.
>
> لأي بورت **معلم** عليه `ssl` في الفحص (أو لو الـ `nc` العادية رجعتلك رموز مبهدلة أو مفيش رد خالص)، استخدم الكلاينت المشفر بدلها:
> ```bash
> openssl s_client -connect localhost:31790
> ```
> اكتب باسوردك بعد ما مخرجات المصافحة تستقر وتشوف البرومبت `---`.
>
> **كرر العملية دي على كل بورت في قايمة المرشحين بتاعتك**، واستبعد بشكل منظم اللي بيرجعلك بس نفس اللي بعتهوله، لحد ما تلاقي البورت الوحيد اللي بيرد بحاجة مختلفة فعلياً—بيانات دخول جديدة حقيقية (اللفل ده بيديك يوزر نيم وباسورد سوا للفل الجاي، لإن وصف اللفل بيقول "credentials" بصيغة الجمع، مش باسورد بس).
>
> بمجرد ما تلاقي البورت الصح وتبعت باسوردك، هيرد عليك بحاجة زي:
> ```
> Correct!
> BANDIT17USERNAME BANDIT17PASSWORD
> ```
> احفظ القيمتين—هتحتاج اليوزر نيم كمان المرة دي، لإن يبدو إنه مش بالبساطة "bandit17" المتوقع (اللفل ده أحياناً بيديك حساب باسم مختلف، فركز كويس في اللي فعلاً هيترجعلك).
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> فحص البورتات بـ `nmap` مع اختبار الخدمات اليدوي هي الخطوة الافتتاحية الكلاسيكية في أي اختبار اختراق احترافي.
>
> - **🔴 من منظور الـ Red Team:** كل عملية بتبدأ بالاستطلاع، و`nmap` هي الأداة القياسية في الصناعة للمرحلة الأولى دي—رسم خريطة لما هو شغال، وأنهي بورتات مفتوحة، وأنهي خدمات/إصدارات شغالة (اللي بعدين بتتقارن بثغرات معروفة/CVEs). التفرقة بين خدمة مشفرة وخدمة نص عادي (بالظبط زي ما اللفل ده بيدربك عليه) بتحدد أسلوب هجومك بالكامل—مش هتقدر تتنصت على باسوردات نص عادي من قناة مشفرة، بس أكيد هتقدر من قناة غير مشفرة، وعشان كده إيجاد "الخدمة الشاذة" مهم جداً عملياً.
> - **🔵 من منظور الـ Blue Team:** المدافعين بيشغلوا فحوصات `nmap` داخلية بتاعتهم بشكل دوري (كجزء من إدارة سطح الهجوم) عشان يمسكوا خدمات مش المفروض تكون مكشوفة، أو إصدارات برامج غير محدثة nmap بيقدر يكتشفها، أو خدمات "اختبار" منسية شغالة على بورتات عالية غريبة عملها مبرمج ونساها—بالظبط نوع خدمات الصدى/المصايد اللي شفتها في اللفل ده. أنظمة كشف التسلل (IDS) كمان بتراقب بالتحديد **النمط المميز** لفحص nmap نفسه (محاولات اتصال متتالية سريعة عبر نطاق بورتات) كإشارة إنذار مبكر إن حد عامل استطلاع نشط على شبكتك.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** صيغة `nmap -p [نطاق] [هدف]` للفحص المستهدف للبورتات، و `-sV` لكشف الخدمة. وكمان ثبّت المنهجية: افحص الأول (شوف إيه الشغال) → حدد (مشفر ولا لأ) → تفاعل (بالأداة الصح لكل واحد) → تأكد (ده الحقيقي ولا مصيدة صدى؟). الحلقة دي المكونة من 4 خطوات هي اختبار اختراق مصغر فعلياً.
> - **تجاهل:** متقلقش من مئات أنواع الفحص والفلاجات التانية بتاعة nmap (زي `-sS` و `-sU` و `-A` وقوالب التوقيت وسكريبتات NSE) دلوقتي—ده موضوع ضخم لدراسة nmap مخصصة لاحقاً. للفل ده، `-p` و `-sV` هما كل اللي محتاجه فعلاً.
