### 🏴‍☠️ [Level 9 &rarr; Level 10]

**1. 🎯 The Objective & The "Why"**
- **The Question:** The password for the next level is stored in the file `data.txt` in one of the few human-readable strings, preceded by several `=` characters.
- **The Goal:** This file is NOT plain text this time — it's likely a binary file full of garbage/gibberish (compiled data, random bytes, non-printable characters). Buried inside all that noise are a FEW actual readable text strings, and one specific one is preceded by a row of equal signs (`====`). You need to extract ONLY the human-readable parts from a mostly-unreadable file.
- **The Why:** This introduces you to the concept of **string extraction from binary data** — a technique used HEAVILY in malware analysis, reverse engineering, and forensics. Real malware samples, compiled executables, and memory dumps are mostly unreadable binary garbage to a human eye, but they almost always contain embedded readable text (URLs, error messages, hardcoded credentials, file paths). Knowing how to pull that text out efficiently is a foundational reverse-engineering skill.

**2. 💻 The Execution (Step-by-Step)**
You're logged in as `bandit9`. Let's check the file:

```bash
ls -la
```
You'll see `data.txt`. If you're tempted to just `cat` it, DON'T — remember from Level 4, dumping raw binary data straight to your terminal with `cat` can seriously mess up your terminal display. Let's confirm it's not plain text first:

```bash
file data.txt
```
This will likely tell you something like `data.txt: data` — confirming it's binary garbage, not clean ASCII text. Good, that confirms our approach needs to change.

Now we bring in the right tool for THIS specific job:

```bash
strings data.txt
```
- `strings`: This command's entire purpose is to **scan through a binary file and extract any sequence of printable characters that's long enough to be considered "real text"** (by default, it looks for sequences of 4 or more consecutive printable characters). Everything else — the actual binary noise/garbage — gets ignored and filtered out. This is EXACTLY the tool malware analysts run first on any suspicious binary file, because it's a fast way to spot clues (URLs, filenames, error messages) without needing a full disassembler.

Running `strings data.txt` alone will still show you a decent chunk of output — dozens or hundreds of readable text fragments buried in the file. We need to narrow it down further to match the level's specific clue: "preceded by several `=` characters."

So we pipe the output of `strings` into `grep`, searching for a pattern of multiple equal signs:

```bash
strings data.txt | grep "="
```
- We're piping (`|`) the extracted readable strings into `grep`, and searching for the `=` character. This will show us every readable line that contains at least one equals sign.

If that still shows a bit too much noise, we can be more specific and search for MULTIPLE consecutive equal signs, since the level said "several":

```bash
strings data.txt | grep "===="
```
This narrows it down even further, since random text is unlikely to naturally contain four equal signs in a row — but a deliberately placed marker (like `====== password ======`) absolutely will. This should leave you with just one or two lines, one of which clearly shows your level 10 password sitting right next to the row of equal signs.

**3. 🌍 Real-World & Tactical Application**
`strings` is one of the FIRST commands run in any real-world malware triage or binary analysis workflow.

- **🔴 Red Team Perspective:** When attackers get their hands on a compiled binary (a piece of malware to study, a firmware dump from an IoT device, or a proprietary application they're trying to reverse-engineer), `strings` is step one — it's fast and can immediately reveal hardcoded IP addresses (C2 server addresses), embedded credentials, debug messages left in by lazy developers, or API endpoints, without needing heavyweight tools like IDA Pro or Ghidra right away.
- **🔵 Blue Team Perspective:** Incident responders and malware analysts run `strings` on suspicious files caught by antivirus/EDR as a first triage step to quickly assess "what does this thing potentially do or talk to" before committing to a deeper, more time-consuming reverse-engineering session. Defenders also use `strings` on firmware images and compiled binaries during security audits to catch developers who (very badly) left hardcoded passwords, private keys, or secret API tokens baked directly into a shipped binary.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** The `strings` command as your go-to for extracting readable text from binary/garbage files, and combining it with `grep` via a pipe to narrow down results based on a known pattern. This "strings + grep" combo is a real forensics workflow, not just a wargame trick.
- **IGNORE:** Don't get distracted trying to fully understand the binary structure of `data.txt` itself (what encoding, what format it technically is) — that's irrelevant here. You don't need to know WHY it's binary, just that `strings` is the correct tool to cut through binary noise.

---

> **5. 🇪🇬    (Sirius Notes - Egyptian Arabic)**
>
> بص  ، اللفل ده هيعرفك على مهارة أساسية في تحليل الملفات الثنائية، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** الباسورد بتاع اللفل الجاي متخزن في ملف `data.txt` جوه واحدة من النصوص القليلة القابلة للقراءة، ومسبوقة بمجموعة من علامات `=`.
> - **الهدف:** الملف ده مش نص عادي المرة دي—غالباً ملف ثنائي (binary) مليان زبالة/رموز غريبة (بيانات مترجمة، بايتات عشوائية، حروف مش قابلة للطباعة). جوه الضوضاء دي كلها، فيه شوية نصوص فعلاً قابلة للقراءة، وواحدة منهم بالتحديد مسبوقة بصف من علامات المساواة (`====`). لازم تستخرج بس الأجزاء المقروءة من ملف غالبيته مش مقروء.
> - **الليه؟** ده بيعرفك على مفهوم **استخراج النصوص من بيانات ثنائية**—تقنية بتتستخدم بكثافة في تحليل المالوير والهندسة العكسية والتحليل الجنائي. عينات المالوير الحقيقية والملفات التنفيذية المترجمة وصور الذاكرة غالباً زبالة ثنائية مش مقروءة للعين البشرية، بس دايماً تقريباً فيها نصوص مقروءة مدمجة (روابط، رسايل أخطاء، باسوردات مكتوبة ثابتة، مسارات ملفات). معرفتك إزاي تستخرج النص ده بكفاءة مهارة أساسية في الهندسة العكسية.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit9`. يلا نشوف الملف:
>
> ```bash
> ls -la
> ```
> هتلاقي `data.txt`. لو هيجيلك وسواس تعمله `cat` على طول، **ماتعملهاش**—افتكر من اللفل الرابع، عرض بيانات ثنائية مباشرة على التيرمينال بـ `cat` ممكن يبهدل شكل التيرمينال. يلا نتأكد إنه مش نص عادي الأول:
>
> ```bash
> file data.txt
> ```
> غالباً هيقولك حاجة زي `data.txt: data`—يعني تأكد إنه فعلاً زبالة ثنائية، مش نص ASCII نضيف. تمام، ده تأكيد إننا محتاجين نغير أسلوبنا.
>
> دلوقتي هنجيب الأداة الصح للشغلانة دي بالتحديد:
>
> ```bash
> strings data.txt
> ```
> - `strings`: شغلانة الأمر ده بالكامل إنه **يفحص ملف ثنائي ويستخرج أي سلسلة حروف قابلة للطباعة طويلة بما فيه الكفاية عشان تتعتبر "نص حقيقي"** (افتراضياً، بيدور على سلاسل من 4 حروف قابلة للطباعة متتالية أو أكتر). كل حاجة تانية—الضوضاء/الزبالة الثنائية الفعلية—بيتجاهلها ويفلترها بره. دي بالظبط الأداة اللي محللين المالوير بيشغلوها الأول على أي ملف ثنائي مشبوه، لإنها طريقة سريعة تلاقي فيها أدلة (روابط، أسامي ملفات، رسايل أخطاء) من غير ما تحتاج disassembler كامل.
>
> تشغيل `strings data.txt` لوحدها هيوريك لسه كمية معقولة من المخرجات—عشرات أو مئات القطع النصية المقروءة المدفونة في الملف. محتاجين نضيقها أكتر عشان نطابق التلميح المحدد بتاع اللفل: "مسبوقة بمجموعة من علامات `=`."
>
> فهنعمل pipe لمخرج `strings` جوه `grep`، وندور على نمط فيه علامات مساواة متعددة:
>
> ```bash
> strings data.txt | grep "="
> ```
> - بنعمل pipe (`|`) للنصوص المستخرجة جوه `grep`، وندور على حرف `=`. ده هيوريلنا كل سطر مقروء فيه علامة مساواة واحدة على الأقل.
>
> لو ده لسه فيه ضوضاء زيادة، نقدر نكون أدق ونبحث عن علامات مساواة متعددة متتالية، لإن اللفل قال "several" (عدة):
>
> ```bash
> strings data.txt | grep "===="
> ```
> ده بيضيقها أكتر، لإن أي نص عشوائي مش هيحتوي طبيعياً على أربع علامات مساواة ورا بعض—لكن علامة متعمدة (زي `====== password ======`) هتحتوي عليها فعلاً. ده هيسيبلك سطر أو اتنين بس، واحد فيهم هيوريك بوضوح باسورد اللفل العاشر جنب صف علامات المساواة.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> `strings` من أول الأوامر اللي بتتشغل في أي عملية فرز أولي لمالوير أو تحليل ملف ثنائي في الواقع.
>
> - **🔴 من منظور الـ Red Team:** لما الهاكر يمسك ملف تنفيذي مترجم (قطعة مالوير عايز يدرسها، أو نسخة firmware من جهاز IoT، أو تطبيق مغلق المصدر عايز يعمله reverse-engineer)، `strings` هي الخطوة الأولى—سريعة وممكن تكشف فوراً IP addresses مكتوبة ثابتة (سيرفرات C2)، باسوردات مدمجة، رسايل debug ناسيها مبرمج كسول، أو API endpoints، من غير ما يحتاج أدوات ثقيلة زي IDA Pro أو Ghidra على طول.
> - **🔵 من منظور الـ Blue Team:** المستجيبين للحوادث ومحللي المالوير بيشغلوا `strings` على ملفات مشبوهة اكتشفها الأنتي فايرس أو الـ EDR كخطوة فرز أولى عشان يقيموا بسرعة "الحاجة دي بتعمل إيه أو بتتكلم مع مين" قبل ما يدخلوا في جلسة هندسة عكسية أعمق وبتاخد وقت أطول. المدافعين كمان بيستخدموا `strings` على صور الـ firmware والملفات المترجمة أثناء التدقيقات الأمنية عشان يمسكوا مبرمجين (بشكل سيء جداً) سايبين باسوردات ثابتة أو مفاتيح خاصة أو توكينات API سرية داخل ملف تنفيذي متسلم للعميل.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** أمر `strings` كأداتك الأساسية لاستخراج نص مقروء من ملفات ثنائية/زبالة، ودمجها مع `grep` عن طريق pipe عشان تضيق النتايج بناءً على نمط معروف. كومبو "strings + grep" ده workflow حقيقي في التحليل الجنائي، مش بس حيلة لعبة.
> - **تجاهل:** متتشتتش وانت بتحاول تفهم بنية الملف الثنائي `data.txt` نفسه بالكامل (إيه نوع التشفير، إيه الفورمات تقنياً)—ده مش مهم هنا. مش محتاج تعرف ليه هو ثنائي، بس محتاج تعرف إن `strings` هي الأداة الصح تقطع من خلالها الضوضاء الثنائية.
