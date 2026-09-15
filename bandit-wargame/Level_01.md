### 🏴‍☠️ [Level 0 &rarr; Level 1]

**1. 🎯 The Objective & The "Why"**
- **The Question:** The password for the next level is stored in a file called `readme` located in the home directory.
- **The Goal:** You need to find that file, view its content, and extract the password inside it to log in as `bandit1`.
- **The Why:** This is your introduction to basic Linux navigation. As an attacker who just gained access to a server (a "shell"), the FIRST thing you do is orient yourself: "Where am I? What files exist here? What can I read?" This is called **enumeration**, and it's arguably the most important skill in all of hacking. 90% of hacking is just looking around carefully.

**2. 💻 The Execution (Step-by-Step)**
You've just SSH'd into `bandit0`. Now let's find that file.

**Step 1: See what's around you.**
```bash
ls
```
`ls` stands for "list." It lists all the files and folders in your current location. You should see one item: `readme`.

You might ask, "Why not use `ls -la`?" Good question—let's use it to be thorough:
```bash
ls -la
```
- `-l`: Stands for "long listing." This shows detailed info: permissions, owner, size, and modification date of each file, instead of just the bare name.
- `-a`: Stands for "all." By default, `ls` hides "hidden" files (any file/folder starting with a dot `.`, like `.bashrc`). The `-a` flag forces it to show EVERYTHING, hidden or not. In hacking, hidden files often contain juicy secrets, so always check with `-a`.

**Step 2: Confirm your location (optional but good practice).**
```bash
pwd
```
This stands for "Print Working Directory." It just tells you the full path of where you currently are (should be `/home/bandit0`). You want to always know your coordinates before you start moving around.

**Step 3: Read the file's content.**
```bash
cat readme
```
- `cat` stands for "concatenate," but in practice, it's just used to **dump the raw content of a file straight onto your screen**. It's the simplest, fastest way to read a text file.
- This will spit out the password for level 1 on your screen.

**Step 4: Save that password somewhere (CRITICAL habit).**
Open a text editor on your OWN local computer (Notepad, VSCode, whatever) and paste that password with a label: `Level 1 password: [Password_Appears_Here]`. If you don't save it, and you close your terminal, you'll have to redo this step.

**Step 5: Log in as the next user.**
```bash
ssh bandit1@bandit.labs.overthewire.org -p 2220
```
When it asks for the password, paste/type the password you found in the `readme` file (NOT "bandit1" this time—that's the whole point, passwords aren't usernames anymore from here on).

**3. 🌍 Real-World & Tactical Application**
This exact `ls` + `cat` combo is used thousands of times a day by every sysadmin, developer, and hacker on the planet.

- **🔴 Red Team Perspective:** Once an attacker lands a shell on a compromised machine (through any exploit), the next move is ALWAYS to look for "loot"—config files, `.bash_history` files, readme files, `.env` files with API keys, or SSH keys left lying around by lazy admins. This level trains that exact muscle memory: "check what's in front of you before doing anything fancy."
- **🔵 Blue Team Perspective:** Never, EVER leave credentials in plaintext files on a server, even for "testing purposes." Developers/Admins often leave passwords in readme.txt, config.php, or notes.txt files thinking "nobody will find it"—but this is one of the top causes of real breaches. Secrets should live in encrypted vaults (like HashiCorp Vault or AWS Secrets Manager), not plain text files.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** `ls`, `cat`, and the habit of checking `pwd` when you land somewhere new. Master these three, they're your bread and butter for the entire wargame.
- **IGNORE:** `file`, `du`, and `find` are mentioned in the tools list for this level but you genuinely don't need them here since there's only ONE obvious file. Don't waste time exploring them yet—they'll get their spotlight in later levels where they're actually necessary.

---

> **5. 🇪🇬 كبسولة سيريوس (Sirius Notes - Egyptian Arabic)**
>
> بص ، اللفل ده أول خطوة حقيقية في عالم الهاكينج، خليني اوريك.
>
> **1. 🎯 الهدف و "ال ليه"**
> - **السؤال:** الباسورد بتاع اللفل الجاي متخزن في ملف اسمه `readme` وموجود في الـ home directory بتاعك.
> - **الهدف:** لازم تدور على الملف ده، تفتحه، وتشوف الباسورد جواه عشان تعمل بيه login كـ `bandit1`.
> - **الليه؟** ده أول تعريف ليك بالتنقل الأساسي في نظام اللينكس. أي هاكر لسه داخل جديد على سيرفر (يعني معاه Shell)، أول حاجة بيعملها إنه يوجه نفسه: "أنا فين؟ فيه ملفات إيه هنا؟ أقدر أقرا إيه؟" الحركة دي اسمها **Enumeration** وهي أهم مهارة في الهاكينج كله. 90% من الهاكينج بس إنك تبص كويس حواليك.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل SSH دلوقتي على `bandit0`. يلا ندور على الملف.
>
> **الخطوة 1: شوف اللي حواليك.**
> ```bash
> ls
> ```
> `ls` معناها "list"، بتوريك كل الملفات والفولدرات في المكان اللي انت فيه. هتلاقي حاجة واحدة اسمها `readme`.
>
> هتقولي ليه ما استخدمناش `ls -la`؟ سؤال جامد، تعالى نستخدمه:
> ```bash
> ls -la
> ```
> - `-l`: معناها "long listing"، بتوريك تفاصيل زي الصلاحيات والمالك والحجم وتاريخ آخر تعديل، مش بس اسم الملف عاري.
> - `-a`: معناها "all"، الـ `ls` عادةً بتخبي أي ملف اسمه بيبدأ بنقطة `.` (زي `.bashrc`). الفلاج ده بيجبرها توري كل حاجة، مخفية أو مش مخفية. في الهاكينج، الملفات المخفية غالباً فيها أسرار، فدايماً استخدم `-a`.
>
> **الخطوة 2: تأكد من مكانك (مش إجباري بس عادة كويسة).**
> ```bash
> pwd
> ```
> معناها "Print Working Directory"، بس بتقولك إنت فين بالظبط (هيقولك `/home/bandit0`). عايز دايماً تعرف احداثياتك قبل ما تتحرك.
>
> **الخطوة 3: اقرا محتوى الملف.**
> ```bash
> cat readme
> ```
> - `cat` أصلها "concatenate"، بس في الاستخدام العملي بنستخدمها عشان **نطبع محتوى الملف على الشاشة على طول**. أسهل وأسرع طريقة تقرا بيها ملف نصي.
> - الأمر ده هيطلعلك الباسورد بتاع اللفل الجاي على الشاشة.
>
> **الخطوة 4: احفظ الباسورد ده في مكان (عادة مهمة جداً).**
> افتح أي محرر نصوص على جهازك انت (Notepad أو VSCode أو أي حاجة) واحفظ الباسورد ده وعلّم عليه: `Level 1 password: [Password_Appears_Here]`. لو مافيش، وقفلت التيرمينال، هتضطر تعمل الخطوات دي تاني.
>
> **الخطوة 5: اعمل لوجين باليوزر الجديد.**
> ```bash
> ssh bandit1@bandit.labs.overthewire.org -p 2220
> ```
> لما يطلب منك الباسورد، حط الباسورد اللي لقيته في ملف الـ `readme` (مش "bandit1" المرة دي—دي بقى الفكرة، من هنا وبعدين الباسورد مش نفس اسم اليوزر).
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> كومبو `ls` و `cat` ده بيتستخدم آلاف المرات كل يوم من أي مبرمج أو أدمن أو هاكر في الكوكب كله.
>
> - **🔴 من منظور الـ Red Team:** لما هاكر يوصل لشِل على جهاز مخترق (من خلال أي ثغرة)، الخطوة اللي بعدها دايماً إنه يدور على "الكنز"—ملفات إعدادات، ملف `.bash_history`، ملفات readme، ملفات `.env` فيها مفاتيح API، أو حتى SSH keys ناسيها أدمن كسول. اللفل ده بيدرب بالظبط العضلة دي: "شوف اللي قدامك الأول قبل ما تعمل أي حركة فاضية".
> - **🔵 من منظور الـ Blue Team:** أبداً متسيبش باسوردات في ملفات نصية عادية على سيرفر، حتى لو "للتجربة بس". كتير من المبرمجين والأدمنز بيسيبوا الباسورد في readme.txt أو config.php فاكرين "محدش هيلاقيها"—لكن دي من أكتر أسباب الاختراقات الحقيقية. الأسرار لازم تتخزن في خزنات مشفرة زي HashiCorp Vault أو AWS Secrets Manager، مش في ملفات نص عادية.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** `ls` و `cat`، وعادة التأكد من `pwd` كل ما توصل لمكان جديد. اتقن الثلاثة دول، هيبقوا أساس شغلك في اللعبة كلها.
> - **تجاهل:** الأدوات زي `file` و `du` و `find` مذكورة في قايمة الأدوات بس مش محتاجها هنا فعلياً لإن فيه ملف واحد واضح بس. متضيعش وقتك تستكشفهم دلوقتي، هيبقى ليهم دور في لفلات جاية لما فعلاً نحتاجهم.
