### 🏴‍☠️ [Level 12 &rarr; Level 13]

**1. 🎯 The Objective & The "Why"**
- **The Question:** The password for the next level is stored in the file `data.txt`, which is a hexdump of a file that has been repeatedly compressed. It's recommended to create a directory under `/tmp` to work in, using `mkdir` with a hard-to-guess name or `mktemp -d`, then copy the datafile with `cp` and rename it with `mv`.
- **The Goal:** This is a multi-layered puzzle. First, you need to reverse a **hexdump** back into actual binary data. Then, that binary data turns out to be a compressed archive — but not just once, it's been compressed OVER AND OVER, multiple times, using different compression formats stacked on top of each other (like a Russian nesting doll). You need to peel back EVERY layer until you finally hit plain readable text.
- **The Why:** This teaches you file format identification and iterative decompression — skills that are constantly used in malware analysis (malware droppers frequently compress/encode payloads in multiple layers specifically to evade signature-based detection) and forensics (recovering data from disk images, memory dumps, or network captures often involves peeling back several layers of encoding).

**2. 💻 The Execution (Step-by-Step)**
You're logged in as `bandit12`. This level involves a LOT of file manipulation, and doing it directly in your home directory would get messy fast. So first, let's build ourselves a clean, private workspace:

```bash
mkdir /tmp/mywork12
cd /tmp/mywork12
```
- `mkdir`: Stands for "make directory." It creates a brand new, empty folder.
- `/tmp/mywork12`: We're creating this folder inside `/tmp`, which is a special Linux directory specifically meant for TEMPORARY files — it's writable by everyone and gets cleared out periodically. It's the perfect sandbox to mess around in without cluttering your actual home directory.
- `cd /tmp/mywork12`: We move INTO that new folder, so all our upcoming commands happen right here.

(A more "professional" alternative the level mentions is `mktemp -d`, which auto-generates a randomly-named temp folder for you — useful on shared multi-user systems where a predictable folder name could be tampered with by other users. For our purposes, a manually named folder works fine.)

Now let's bring our target file into this workspace:

```bash
cp ~/data.txt .
```
- `cp`: Stands for "copy." It duplicates a file from one location to another.
- `~/data.txt`: The `~` symbol is shorthand for "my home directory." So this points to the original `data.txt` file sitting in `bandit12`'s home folder.
- `.`: Remember from way back, this means "the current directory." So we're copying the file FROM the home directory TO right here where we're standing now.

Let's look at what we're dealing with:
```bash
cat data.txt
```
You'll see a hexdump — a bunch of lines showing hexadecimal byte values in a structured format (something like `00000000 1f 8b 08 08 ...`). This is NOT the actual file; it's a TEXT REPRESENTATION of binary data, byte-by-byte, in hex notation.

We need to reverse this back into real binary. The tool for that is `xxd`:

```bash
xxd -r data.txt file1
```
- `xxd`: A tool for creating (or reversing) hexdumps.
- `-r`: Stands for "reverse." Without this flag, `xxd` would take a normal file and CONVERT it into a hexdump. WITH `-r`, it does the opposite — it takes a hexdump (text) and reconstructs the original binary data from it.
- `data.txt`: Our input hexdump file.
- `file1`: The output filename where the reconstructed binary data gets saved.

Now let's identify what `file1` actually IS:
```bash
file file1
```
This will likely tell you something like `file1: gzip compressed data`. Ah-ha — it's a compressed archive! Let's rename it to have the proper extension (this matters because some decompression tools are picky about extensions) and then decompress it:

```bash
mv file1 file1.gz
gzip -d file1.gz
```
- `mv`: Stands for "move" (but it's also used for renaming, since renaming is technically just "moving" a file to a new name in the same location).
- `gzip -d`: Decompresses a `.gz` file. The `-d` flag means "decompress." This will produce a new file called `file1` (gzip strips the `.gz` extension automatically after decompressing).

Now check the type again:
```bash
file file1
```
It'll show ANOTHER compression format, maybe `bzip2 compressed data`, or `POSIX tar archive`, or `ASCII text` if you're finally done. **This is where the "repeatedly compressed" part comes in** — you literally repeat this exact cycle (rename with correct extension → decompress with the matching tool → check the type again) over and over, layer by layer, until `file` finally reports `ASCII text` instead of a compression format.

Here's the general pattern you'll cycle through, using whichever tool matches what `file` reports each time:

```bash
mv file1 file1.bz2 && bzip2 -d file1.bz2
```
```bash
mv file1 file1.tar && tar -xf file1.tar
```
- `bzip2 -d`: Decompresses `.bz2` files, same `-d` "decompress" logic as gzip.
- `tar -xf`: Extracts a `.tar` archive. `-x` means "extract," `-f` means "the following argument is the filename to operate on." Note: `tar` archives are just "bundled" files (like a zip without compression) — extracting one might produce a differently-named file, so always run `ls` after each `tar` extraction to see what popped out.

You just keep looping: `file [filename]` → identify the type → rename with the right extension → decompress with the matching tool (`gzip -d`, `bzip2 -d`, or `tar -xf`) → repeat. Eventually, `file` will report `ASCII text`, and running:

```bash
cat [finalfilename]
```
will reveal your level 13 password.

**3. 🌍 Real-World & Tactical Application**
Multi-layer decompression and format identification are core skills in malware analysis and digital forensics.

- **🔴 Red Team Perspective:** Malware droppers and payload delivery systems frequently wrap their actual malicious code in MULTIPLE layers of compression/encoding (gzip inside base64 inside a custom XOR obfuscation, for example) specifically to defeat simple signature-based antivirus scanning and to slow down human analysts doing manual triage. Understanding how to systematically peel back these layers (identify → convert → repeat) is exactly the mental loop a malware analyst runs through on a real sample.
- **🔵 Blue Team Perspective:** Forensic analysts recovering data from a hard drive image, a memory dump, or a suspicious network capture constantly deal with nested/layered file formats and need the exact same "identify with `file`, then use the matching tool" methodical approach. This is also why security teams build automated sandboxes (like Cuckoo Sandbox) that automatically detect and unwrap common compression/obfuscation layers before a human ever looks at a sample.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** The LOOP/CYCLE itself: `file` (identify) → rename with correct extension → decompress with the matching tool → repeat. Also master `xxd -r` for hex-to-binary conversion, and remember working in `/tmp` keeps your home directory clean. This "identify then act" methodology is more valuable than memorizing every single compression tool's syntax.
- **IGNORE:** Don't stress about memorizing EVERY flag for `gzip`, `bzip2`, and `tar` — you only really need `-d` (decompress) for the first two and `-xf` (extract file) for tar. Also don't worry about compression algorithms' internal mechanics (how gzip's DEFLATE algorithm actually works mathematically) — completely irrelevant for solving this.

---

> **5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)**
>
> بص  ، اللفل ده هيخليك تحس إنك بتفك دمية روسية متداخلة، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** الباسورد بتاع اللفل الجاي متخزن في ملف `data.txt`، وده عبارة عن hexdump لملف اتضغط أكتر من مرة. متنصح تعمل فولدر تحت `/tmp` تشتغل فيه، باستخدام `mkdir` باسم صعب التخمين أو `mktemp -d`، وبعدين تنسخ ملف البيانات بـ `cp` وتعيد تسميته بـ `mv`.
> - **الهدف:** ده لغز متعدد الطبقات. الأول لازم تعكس **hexdump** لبيانات ثنائية حقيقية. وبعدين البيانات الثنائية دي هتطلع أرشيف مضغوط—بس مش مرة واحدة، ده اتضغط مرة ورا مرة، بصيغ ضغط مختلفة فوق بعض (زي الدمية الروسية المتداخلة). لازم تقشر كل طبقة لحد ما توصل أخيراً لنص عادي مقروء.
> - **الليه؟** ده بيعلمك التعرف على أنواع الملفات وفك الضغط بشكل متكرر—مهارات بتتستخدم باستمرار في تحليل المالوير (المالوير بيضغط/يشفر حمولته بطبقات متعددة عمداً عشان يهرب من الكشف بالتوقيعات) والتحليل الجنائي (استرجاع بيانات من صور أقراص أو ذاكرة أو التقاطات شبكة غالباً بيتطلب تقشير عدة طبقات ترميز).
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit12`. اللفل ده فيه تلاعب كتير بالملفات، وعملها في الـ home directory بتاعتك مباشرة هيبوظ المكان بسرعة. فالأول، يلا نبني مساحة عمل نضيفة وخاصة:
>
> ```bash
> mkdir /tmp/mywork12
> cd /tmp/mywork12
> ```
> - `mkdir`: معناها "make directory". بتعمل فولدر جديد فاضي.
> - `/tmp/mywork12`: بنعمل الفولدر ده جوه `/tmp`، وهو فولدر خاص في اللينكس مخصص للملفات **المؤقتة**—كل الناس تقدر تكتب فيه وبيتم مسحه بشكل دوري. ده المكان المثالي عشان تلعب فيه من غير ما تبوظ الـ home directory الحقيقي بتاعك.
> - `cd /tmp/mywork12`: بندخل جوه الفولدر الجديد ده، عشان كل الأوامر الجاية تحصل هنا بالظبط.
>
> (بديل "أكثر احترافية" ذكره اللفل هو `mktemp -d`، اللي بيعمل فولدر مؤقت باسم عشوائي أوتوماتيك ليك—مفيد في أنظمة متعددة المستخدمين لإن اسم فولدر متوقع ممكن حد تاني يلعب فيه. لأغراضنا، فولدر باسم يدوي هيشتغل تمام.)
>
> دلوقتي يلا نجيب الملف المستهدف لمساحة العمل دي:
>
> ```bash
> cp ~/data.txt .
> ```
> - `cp`: معناها "copy". بتنسخ ملف من مكان لمكان تاني.
> - `~/data.txt`: الرمز `~` اختصار لـ "الـ home directory بتاعي". فده بيشير للملف الأصلي `data.txt` الموجود في فولدر `bandit12`.
> - `.`: افتكر من زمان، دي معناها "الفولدر الحالي". فبنعمل نسخ للملف من الـ home directory لهنا، المكان اللي واقفين فيه دلوقتي.
>
> يلا نشوف احنا قدامنا ايه:
> ```bash
> cat data.txt
> ```
> هتشوف hexdump—مجموعة سطور بتوري قيم بايتات هيكسيديسيمال بشكل منظم (حاجة زي `00000000 1f 8b 08 08 ...`). ده مش الملف الفعلي؛ ده **تمثيل نصي** للبيانات الثنائية، بايت بايت، بترميز هيكس.
>
> محتاجين نعكس ده رجوع لبيانات ثنائية حقيقية. الأداة لده هي `xxd`:
>
> ```bash
> xxd -r data.txt file1
> ```
> - `xxd`: أداة لعمل (أو عكس) الـ hexdumps.
> - `-r`: معناها "reverse" (عكسي). من غير الفلاج ده، `xxd` هتاخد ملف عادي وتحوله لـ hexdump. **مع** `-r`، هي بتعمل العكس—بتاخد hexdump (نص) وتعيد بناء البيانات الثنائية الأصلية منه.
> - `data.txt`: ملف الـ hexdump اللي بندخله.
> - `file1`: اسم الملف الناتج اللي هيتخزن فيه البيانات الثنائية المعاد بناؤها.
>
> دلوقتي يلا نتعرف على `file1` ده فعلاً ايه:
> ```bash
> file file1
> ```
> غالباً هيقولك حاجة زي `file1: gzip compressed data`. آه، ده أرشيف مضغوط! يلا نعيد تسميته باللاحقة الصحيحة (ده مهم لإن بعض أدوات فك الضغط دقيقة بخصوص اللاحقات) وبعدين نفك ضغطه:
>
> ```bash
> mv file1 file1.gz
> gzip -d file1.gz
> ```
> - `mv`: معناها "move" (نقل)، بس بتستخدم كمان لإعادة التسمية، لإن إعادة التسمية تقنياً هي "نقل" الملف لاسم جديد في نفس المكان.
> - `gzip -d`: بتفك ضغط ملف `.gz`. الفلاج `-d` معناها "decompress" (فك ضغط). ده هينتج ملف جديد اسمه `file1` (gzip بتشيل اللاحقة `.gz` أوتوماتيك بعد فك الضغط).
>
> دلوقتي شوف النوع تاني:
> ```bash
> file file1
> ```
> هيوريك صيغة ضغط **تانية**، ممكن `bzip2 compressed data`، أو `POSIX tar archive`، أو `ASCII text` لو خلصت أخيراً. **هنا بالظبط بيجي جزء "اتضغط مرة ورا مرة"**—بتكرر حرفياً نفس الدورة دي (إعادة تسمية باللاحقة الصحيحة → فك ضغط بالأداة المناسبة → شوف النوع تاني) مرة ورا مرة، طبقة ورا طبقة، لحد ما `file` أخيراً تقولك `ASCII text` بدل صيغة ضغط.
>
> الطريقة العامة اللي هتكررها، بالأداة اللي تناسب ما `file` بتقوله كل مرة:
>
> ```bash
> mv file1 file1.bz2 && bzip2 -d file1.bz2
> ```
> ```bash
> mv file1 file1.tar && tar -xf file1.tar
> ```
> - `bzip2 -d`: بتفك ضغط ملفات `.bz2`، نفس منطق `-d` بتاع gzip.
> - `tar -xf`: بتستخرج أرشيف `.tar`. `-x` معناها "extract" (استخراج)، `-f` معناها "الحاجة اللي بعدي هي اسم الملف اللي هنشتغل عليه". لاحظ: أرشيفات `tar` هي بس "لَمّ" ملفات مع بعض (زي zip بس من غير ضغط)—استخراجها ممكن ينتج ملف باسم مختلف، فدايماً شغل `ls` بعد كل استخراج `tar` عشان تشوف طلع إيه.
>
> كمّل تكرر: `file [اسم الملف]` → حدد النوع → أعد التسمية باللاحقة الصح → فك الضغط بالأداة المناسبة (`gzip -d` أو `bzip2 -d` أو `tar -xf`) → كرر. في النهاية، `file` هتقولك `ASCII text`، وتشغيل:
>
> ```bash
> cat [الاسم النهائي]
> ```
> هيوريك باسورد اللفل الثالث عشر.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> فك الضغط متعدد الطبقات والتعرف على أنواع الملفات مهارات أساسية في تحليل المالوير والتحليل الجنائي الرقمي.
>
> - **🔴 من منظور الـ Red Team:** أنظمة توصيل المالوير غالباً بتلف الكود الخبيث الفعلي بطبقات **متعددة** من الضغط/الترميز (gzip جوه base64 جوه تعمية XOR مخصصة مثلاً) بالتحديد عشان تهزم فحص الأنتي فايرس البسيط المعتمد على التوقيعات وتبطئ المحللين البشريين وهما بيعملوا فرز يدوي. فهمك إزاي تقشر الطبقات دي بشكل منظم (حدد → حول → كرر) هي بالظبط الحلقة الذهنية اللي محلل المالوير بيمر بيها على عينة حقيقية.
> - **🔵 من منظور الـ Blue Team:** محللين الأدلة الجنائية اللي بيسترجعوا بيانات من صورة قرص صلب، أو تفريغ ذاكرة، أو التقاط شبكة مشبوه، دايماً بيتعاملوا مع صيغ ملفات متداخلة/متعددة الطبقات ومحتاجين بالظبط نفس الأسلوب المنهجي "حدد بـ `file`، بعدين استخدم الأداة المناسبة". ده كمان سبب بناء فرق الأمن لبيئات اختبار آلية (زي Cuckoo Sandbox) بتكتشف وتفك طبقات الضغط/التعمية الشائعة أوتوماتيك قبل ما إنسان يشوف العينة أصلاً.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** الحلقة/الدورة نفسها: `file` (حدد) → أعد التسمية باللاحقة الصحيحة → فك الضغط بالأداة المناسبة → كرر. وكمان اتقن `xxd -r` لتحويل الهيكس لبيانات ثنائية، وافتكر إن الشغل جوه `/tmp` بيخلي الـ home directory بتاعك نضيف. منهجية "حدد ثم تصرف" دي أهم من حفظ صيغة كل أداة ضغط لوحدها.
> - **تجاهل:** متتوتّرش من حفظ كل فلاج بتاع `gzip` و `bzip2` و `tar`—محتاج بس `-d` (فك ضغط) للاتنين الأولانيين و `-xf` (استخراج ملف) لـ tar. وكمان متقلقش من الآليات الداخلية لخوارزميات الضغط (إزاي خوارزمية DEFLATE بتاعة gzip بتشتغل رياضياً)—مش مهم خالص لحل اللفل ده.
