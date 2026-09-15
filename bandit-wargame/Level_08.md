### 🏴‍☠️ [Level 7 &rarr; Level 8]

**1. 🎯 The Objective & The "Why"**
- **The Question:** The password for the next level is stored in the file `data.txt` next to the word `millionth`.
- **The Goal:** There's a single file, `data.txt`, sitting right in your home directory. It's probably HUGE, full of thousands of lines, each containing some random word followed by some value. You need to find the ONE specific line that contains the word `millionth`, and grab the password sitting right beside it.
- **The Why:** This introduces you to arguably THE single most important tool in the entire Linux security toolkit: **`grep`**. Searching through massive amounts of text for a specific pattern is something you will do literally every single day in security work — searching log files for suspicious activity, searching source code for hardcoded credentials, searching config files for vulnerable settings. If you master nothing else from this whole wargame, master `grep`.

**2. 💻 The Execution (Step-by-Step)**
You're logged in as `bandit7`. Let's confirm the file exists:

```bash
ls -la
```
You'll see `data.txt` sitting there. Let's peek at how big/messy it is:

```bash
wc -l data.txt
```
- `wc`: Stands for "word count." 
- `-l`: Tells it to count **lines** specifically instead of words or characters.
This will show you something like `102000 data.txt` — meaning there are over a hundred thousand lines in this file. There is ABSOLUTELY no way you're scrolling through that manually with `cat` and your own eyeballs looking for one word. This is exactly why we need a smarter tool.

Enter `grep`:

```bash
grep millionth data.txt
```
- `grep`: This command's ENTIRE purpose in life is to **search through text (files or input) line-by-line, and print out only the lines that match a pattern you give it.** Think of it as Ctrl+F for the command line, except infinitely more powerful and scriptable.
- `millionth`: This is our **search pattern** — the exact word we're hunting for. `grep` will scan every single one of those 100,000+ lines and only show us the ones containing this word.
- `data.txt`: This is the file we're telling `grep` to search inside.

Run this, and instead of drowning in 100,000 lines, you get back ONE single line, something like:
```
millionth       aBcDeFgHiJkLmNoPqRsTuVwXyZ12345
```
The word `millionth` is on the left, and right next to it (separated by some spaces or tabs) is the password for level 8. Copy that value, and you're ready to move to `bandit8`.

**3. 🌍 Real-World & Tactical Application**
`grep` is not just useful, it's basically MANDATORY knowledge for anyone touching a Linux terminal professionally.

- **🔴 Red Team Perspective:** After landing a shell on a target, attackers use `grep` constantly to hunt for gold in massive files: `grep -r "password" /var/www/` to find hardcoded credentials in web app source code, `grep "Failed password" /var/log/auth.log` to check if their own brute-force attempts got logged (and clean up traces), or `grep -i "api_key"` across config files to steal API tokens. `grep` combined with `find` is basically the backbone of manual post-exploitation looting.
- **🔵 Blue Team Perspective:** Defenders and SOC analysts use `grep` religiously when doing incident response — searching gigabytes of log files for a specific suspicious IP address, a malicious user-agent string, or a specific error pattern indicating an attack (like `grep "UNION SELECT" access.log` to hunt for SQL injection attempts). Nearly every SIEM query language (like Splunk's SPL) is conceptually built on the same "find lines matching a pattern" logic that `grep` pioneered.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** The basic `grep pattern filename` syntax — this is the 20% of `grep` that gets used 80% of the time. Also remember `wc -l` as a quick way to gauge "how big is this file" before deciding your approach.
- **IGNORE:** Don't get overwhelmed by the massive list of tools mentioned for this level (`sort`, `uniq`, `strings`, `base64`, `tr`, `tar`, `gzip`, `bzip2`, `xxd`) — those are hints for LATER levels in the wargame, not needed here. This specific challenge is solved with `grep` alone. Don't waste time researching all of them right now.

---

> **5. 🇪🇬  (Sirius Notes - Egyptian Arabic)**
>
> بص  ، اللفل ده هيعرفك على أهم أداة في عالم اللينكس بالكامل، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** الباسورد بتاع اللفل الجاي متخزن في ملف اسمه `data.txt` جنب كلمة `millionth`.
> - **الهدف:** فيه ملف واحد، `data.txt`، موجود في الـ home directory بتاعك. غالباً ضخم جداً، فيه آلاف السطور، كل سطر فيه كلمة عشوائية وجنبها قيمة. لازم تدور على السطر المحدد اللي فيه كلمة `millionth`، وتاخد الباسورد الموجود جنبها.
> - **الليه؟** ده بيعرفك على واحدة من أهم أدوات الأمن السيبراني بالكامل: **`grep`**. البحث في كميات ضخمة من النصوص عن نمط معين حاجة هتعملها فعلياً كل يوم في شغل الأمن—بتدور في ملفات اللوج على نشاط مشبوه، بتدور في الكود المصدري على باسوردات مكتوبة بشكل ثابت، بتدور في ملفات الإعدادات على أعدادات فيها ثغرات. لو مش هتحفظ حاجة تانية من اللعبة دي كلها، احفظ `grep`.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit7`. يلا نتأكد إن الملف موجود:
>
> ```bash
> ls -la
> ```
> هتلاقي `data.txt` موجود. يلا نشوف حجمه وفوضاه قد ايه:
>
> ```bash
> wc -l data.txt
> ```
> - `wc`: معناها "word count."
> - `-l`: بتقوله يعد **السطور (lines)** بالتحديد مش الكلمات أو الحروف.
> هتلاقي حاجة زي `102000 data.txt`—يعني الملف فيه أكتر من مية ألف سطر. مفيش طريقة خالص إنك تعمل سكرول عليهم يدوي بـ `cat` وتدور بعينك على كلمة واحدة. عشان كده بالظبط محتاجين أداة أذكى.
>
> يدخل `grep`:
>
> ```bash
> grep millionth data.txt
> ```
> - `grep`: شغلانة الأمر ده بالكامل في الحياة إنه **يدور في النص (ملفات أو إدخال) سطر بسطر، ويطبع بس السطور اللي متطابقة مع نمط انت حددهولها.** فكر فيها كأنها Ctrl+F لسطر الأوامر، بس أقوى بكتير وممكن تتحكم فيها.
> - `millionth`: ده **نمط البحث** بتاعنا—الكلمة اللي بندور عليها بالظبط. `grep` هيفحص كل سطر من المية ألف سطر دول وهيوري لنا بس السطور اللي فيها الكلمة دي.
> - `data.txt`: ده الملف اللي بنقول لـ `grep` يدور جواه.
>
> شغل الأمر ده، وبدل ما تغرق في مية ألف سطر، هتاخد سطر واحد بس، حاجة زي:
> ```
> millionth       aBcDeFgHiJkLmNoPqRsTuVwXyZ12345
> ```
> كلمة `millionth` على الشمال، وجنبها على طول (متفصولة بمسافات أو تابات) هيكون الباسورد بتاع اللفل الثامن. انسخ القيمة دي، وأنت جاهز تنتقل لـ `bandit8`.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> `grep` مش بس مفيدة، هي أساسية بشكل شبه إجباري لأي حد بيتعامل مع تيرمينال لينكس بشكل احترافي.
>
> - **🔴 من منظور الـ Red Team:** بعد ما الهاكر يوصل لشِل على هدف، بيستخدم `grep` باستمرار عشان يدور على كنوز في ملفات ضخمة: `grep -r "password" /var/www/` عشان يلاقي باسوردات مكتوبة ثابتة في كود تطبيق ويب، أو `grep "Failed password" /var/log/auth.log` عشان يشوف لو محاولات الـ brute-force بتاعته اتسجلت (ويمسح آثاره)، أو `grep -i "api_key"` في ملفات الإعدادات عشان يسرق مفاتيح API. كومبو `grep` مع `find` أساساً هو عمود شغل النهب بعد الاختراق يدوياً.
> - **🔵 من منظور الـ Blue Team:** المدافعين ومحللي الـ SOC بيستخدموا `grep` باستمرار وهما بيعملوا استجابة للحوادث—بيدوروا في جيجابايتات من ملفات اللوج على IP مشبوه معين، أو نص user-agent خبيث، أو نمط خطأ معين بيدل على هجوم (زي `grep "UNION SELECT" access.log` عشان يدوروا على محاولات SQL injection). تقريباً كل لغة استعلام SIEM (زي SPL بتاعة Splunk) مبنية مفاهيمياً على نفس منطق "دور على سطور متطابقة مع نمط" اللي `grep` بدأه.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** الصيغة الأساسية `grep نمط اسم_الملف`—دي الـ 20% من `grep` اللي بتتستخدم 80% من الوقت. وكمان افتكر `wc -l` كطريقة سريعة تقيس بيها "الملف ده حجمه قد ايه" قبل ما تقرر أسلوبك.
> - **تجاهل:** متتخضش من القايمة الضخمة من الأدوات المذكورة للفل ده (`sort` و `uniq` و `strings` و `base64` و `tr` و `tar` و `gzip` و `bzip2` و `xxd`)—دول تلميحات للفلات **جاية** في اللعبة، مش محتاجينهم هنا. التحدي ده بيتحل بـ `grep` لوحدها بس. متضيعش وقتك تبحث عنهم كلهم دلوقتي.
