### 🏴‍☠️ [Level 11 &rarr; Level 12]

**1. 🎯 The Objective & The "Why"**
- **The Question:** The password for the next level is stored in the file `data.txt`, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions.
- **The Goal:** The file contains text that's been scrambled using a classic substitution cipher called **ROT13**. Every single letter has been shifted 13 spots down the alphabet (A becomes N, B becomes O, and so on, wrapping back around at the end). You need to reverse this shift to reveal the real password.
- **The Why:** This is your first hands-on encounter with a **substitution cipher**, one of the oldest concepts in cryptography. While ROT13 itself is laughably weak and provides zero real security (it's used more as an "obfuscation" trick than actual encryption), understanding it builds the foundation for recognizing and reversing more complex ciphers later. It also introduces you to `tr`, one of the most versatile text-transformation tools in Linux.

**2. 💻 The Execution (Step-by-Step)**
You're logged in as `bandit11`. Let's peek at the file:

```bash
cat data.txt
```
You'll see something like:
```
Gur cnffjbeq vf synxfrqcnffjbeq123
```
Notice it LOOKS like English (word lengths, spacing, sentence structure feel natural) but every word is nonsense. This "almost-English-but-not-quite" pattern is the classic fingerprint of ROT13 — it's specifically designed to be reversible with a simple shift, unlike random gibberish.

You might ask, "Why is ROT13 special compared to other shift amounts?" I'll tell you the beautiful trick behind it: the English alphabet has 26 letters. 13 is EXACTLY half of 26. This means ROT13 is its own inverse — applying it once scrambles the text, and applying the EXACT SAME operation a second time perfectly unscrambles it back to the original. You don't need a separate "decode" function; encoding and decoding are literally the same operation.

Now let's decode it using `tr`:

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

Let's break this down piece by piece, because the syntax looks intimidating at first:
- `cat data.txt`: Dumps the raw scrambled content of the file.
- `|`: Pipes that raw output directly into our next command instead of printing it first.
- `tr`: Stands for "translate" or "transliterate." Its entire job is to take a stream of text and **replace every character from one set with the corresponding character in another set**, position by position.
- `'A-Za-z'`: This is the FIRST set — "the source characters." `A-Z` means "every uppercase letter from A to Z," and `a-z` means "every lowercase letter from a to z." Combined, this represents the entire alphabet, both cases.
- `'N-ZA-Mn-za-m'`: This is the SECOND set — "what to replace each source character with," matched position-by-position with the first set. Here's the clever part: `N-ZA-M` covers uppercase N through Z, then wraps around to A through M — this is literally the alphabet rotated by 13 positions! Same logic with `n-za-m` for lowercase. So `tr` takes the letter A (1st position in the source set) and replaces it with N (1st position in the destination set), takes B and replaces it with O, and so on, all the way through the entire alphabet, for both cases.

Running this command will output the fully decoded, readable text, revealing your level 12 password embedded in a normal English sentence.

**3. 🌍 Real-World & Tactical Application**
While ROT13 itself isn't used for real security anymore, the underlying SKILL — recognizing substitution ciphers and using `tr` for character transformation — is extremely relevant.

- **🔴 Red Team Perspective:** Attackers occasionally still encounter ROT13 (or its cousins) used in CTFs, old legacy systems, forum spoiler tags, or as a lazy "security through obscurity" attempt by inexperienced developers trying to hide something (like a hardcoded API key ROT13'd inside source code). More broadly, `tr` itself is a powerhouse tool attackers use for quick text manipulation during recon — stripping unwanted characters from scraped data, converting case, or cleaning up wordlists for password cracking (`tr 'A-Z' 'a-z'` to lowercase an entire wordlist instantly, for example).
- **🔵 Blue Team Perspective:** Security teams should NEVER rely on ROT13 or any simple substitution cipher to protect actual sensitive data — it provides zero cryptographic security and can be broken instantly. If you find ROT13'd credentials in a codebase during a security audit, that's an immediate critical finding — real secrets need proper encryption (AES) or, better yet, should live in a secrets manager, never hardcoded in any form, obfuscated or not.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** Recognizing the "readable-but-wrong" pattern that signals ROT13 at a glance, and the `tr 'A-Za-z' 'N-ZA-Mn-za-m'` command structure as your go-to decoder. Also understand the beautiful math fact that ROT13 is self-reversing (13 = half of 26).
- **IGNORE:** Don't bother memorizing `tr`'s dozens of other use cases (deleting characters with `-d`, squeezing repeats with `-s`) right now — those are separate lessons for separate contexts. Focus purely on the character-mapping/rotation use case for this level.

---

> **5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)**
>
> بص ، اللفل ده هيعرفك على أقدم أنواع التشفير في التاريخ، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** الباسورد بتاع اللفل الجاي متخزن في ملف `data.txt`، وكل الحروف الصغيرة (a-z) والكبيرة (A-Z) اتزاحت 13 مكان.
> - **الهدف:** الملف فيه نص اتلخبط باستخدام تشفير كلاسيكي اسمه **ROT13**. كل حرف اتحرك 13 مكان لقدام في الأبجدية (A بتبقى N، وB بتبقى O، وهكذا، وبترجع لأول الأبجدية تاني لما توصل الآخر). لازم تعكس الإزاحة دي عشان تشوف الباسورد الحقيقي.
> - **الليه؟** ده أول تعامل عملي ليك مع **تشفير الاستبدال (substitution cipher)**، من أقدم مفاهيم التشفير. رغم إن الـ ROT13 ضعيف جداً ومش بيوفر أي أمان حقيقي (بيتستخدم كـ "تعمية" مش تشفير فعلي)، فهمك ليه بيبني الأساس اللي هتحتاجه عشان تتعرف وتفك تشفيرات أعقد بعدين. وكمان بيعرفك على `tr`، من أقوى أدوات تحويل النصوص في اللينكس.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit11`. يلا نشوف الملف:
>
> ```bash
> cat data.txt
> ```
> هتلاقي حاجة زي:
> ```
> Gur cnffjbeq vf synxfrqcnffjbeq123
> ```
> لاحظ إن شكله "يشبه الإنجليزي" (طول الكلمات، المسافات، بنية الجملة حاسة طبيعية) بس كل كلمة معناها فاضي. النمط ده "شبه إنجليزي بس مش مظبوط" هو البصمة الكلاسيكية للـ ROT13—مصمم بالتحديد إنه يكون قابل للعكس بإزاحة بسيطة، مش زي كلام عشوائي تماماً.
>
> هتقولي "ليه الـ ROT13 مميز بالذات عن أي إزاحة تانية؟" هقولك السر الجميل وراه: الأبجدية الإنجليزية فيها 26 حرف. الرقم 13 هو **بالظبط نص** 26. ده معناه إن ROT13 بيعكس نفسه—طبقه مرة بتلخبط النص، وطبقه مرة تانية بنفس العملية بالظبط بيرجعه لأصله تماماً. مش محتاج دالة "فك تشفير" منفصلة؛ التشفير وفك التشفير حرفياً نفس العملية.
>
> يلا نفك التشفير باستخدام `tr`:
>
> ```bash
> cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
> ```
>
> خليني افككها حتة حتة لإن الصيغة شكلها مرعب أول وهلة:
> - `cat data.txt`: بيطبع المحتوى الملخبط الخام بتاع الملف.
> - `|`: بيبعت المخرج الخام ده مباشرة للأمر اللي بعده بدل ما يطبعه الأول.
> - `tr`: معناها "translate" أو "transliterate" (ترجمة/تحويل). شغلانتها بالكامل إنها تاخد سلسلة نص و**تستبدل كل حرف من مجموعة أولى بالحرف المقابل ليه في مجموعة تانية**، موقع بموقع.
> - `'A-Za-z'`: دي المجموعة **الأولى**—"الحروف المصدر". `A-Z` معناها "كل حرف كابيتال من A لـ Z"، و `a-z` معناها "كل حرف سمول من a لـ z". مع بعض، دي كل الأبجدية بحالتيها.
> - `'N-ZA-Mn-za-m'`: دي المجموعة **التانية**—"استبدل بإيه كل حرف مصدر"، متطابقة موقع بموقع مع المجموعة الأولى. الجزء الذكي هنا: `N-ZA-M` بتغطي الحروف الكابيتال من N لـ Z، وبعدين بترجع لـ A لـ M—ده حرفياً الأبجدية مزاحة 13 مكان! نفس المنطق مع `n-za-m` للحروف السمول. فـ `tr` بتاخد الحرف A (الموقع الأول في المجموعة المصدر) وتستبدله بـ N (الموقع الأول في المجموعة الهدف)، وتاخد B وتستبدله بـ O، وهكذا لحد آخر الأبجدية، بالحالتين.
>
> تشغيل الأمر ده هيطلعلك النص الكامل مفكوك ومقروء، وهيوريك باسورد اللفل الثاني عشر جوه جملة إنجليزية عادية.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> رغم إن ROT13 نفسه مش بيتستخدم للأمان الحقيقي، المهارة اللي وراه—التعرف على تشفيرات الاستبدال واستخدام `tr` لتحويل الحروف—مهمة جداً.
>
> - **🔴 من منظور الـ Red Team:** الهاكرز أحياناً لسه بيقابلوا ROT13 (أو أقاربه) مستخدم في تحديات CTF، أو أنظمة قديمة، أو تاجات "سبويلر" في المنتديات، أو كمحاولة كسولة من مبرمج قليل الخبرة عايز يخبي حاجة (زي مفتاح API مكتوب ثابت في الكود ومشفر بـ ROT13). بشكل أعم، أداة `tr` نفسها قوة هائلة يستخدمها المهاجمين للتلاعب السريع بالنصوص أثناء الاستطلاع—إزالة حروف مش عايزينها من بيانات مجمعة، تحويل حالة الحروف، أو تنظيف قوايم الكلمات لكسر الباسوردات (`tr 'A-Z' 'a-z'` عشان تحول قايمة كلمات كاملة لحروف صغيرة فوراً مثلاً).
> - **🔵 من منظور الـ Blue Team:** فرق الأمن أبداً ميعتمدوش على ROT13 أو أي تشفير استبدال بسيط لحماية بيانات حساسة فعلياً—ده بيوفر صفر أمان تشفيري وممكن ينكسر فوراً. لو لقيت باسوردات مشفرة بـ ROT13 في كود المشروع أثناء تدقيق أمني، ده اكتشاف حرج فوري—الأسرار الحقيقية محتاجة تشفير حقيقي (AES) أو، أفضل من كده، تتخزن في مدير أسرار، مش مكتوبة ثابتة في أي شكل، معماة كانت أو لأ.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** التعرف على نمط "مقروء بس غلط" اللي بيدل على ROT13 من أول نظرة، وصيغة الأمر `tr 'A-Za-z' 'N-ZA-Mn-za-m'` كأداتك الأساسية لفك التشفير. وكمان افهم الحقيقة الرياضية الجميلة إن ROT13 بيعكس نفسه (13 = نص 26).
> - **تجاهل:** متضيعش وقتك تحفظ عشرات الاستخدامات التانية لـ `tr` دلوقتي (زي حذف حروف بـ `-d`، أو ضغط التكرار بـ `-s`)—دول دروس منفصلة لسياقات تانية. ركز بس على استخدام تحويل/إزاحة الحروف للفل ده.
