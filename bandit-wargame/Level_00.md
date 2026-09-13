# 🏴‍☠️ Bandit Level 0

**Target:** Log into the game using SSH with the provided credentials.
**Concepts Tested:** SSH syntax, non-standard ports, basic networking concepts.

---

### 1. 🎯 The Attack Vector
The objective is to establish a remote connection to the target server using SSH (Secure Shell). SSH is an encrypted network protocol used to operate network services securely over an unsecured network.

The standard syntax is `ssh username@hostname`. We are provided with:
*   **Username:** `bandit0`
*   **Hostname:** `bandit.labs.overthewire.org`

By default, SSH connects to port 22. However, the target server uses port `2220`. A port (ranging from 1 to 65535) acts like an "apartment number" directing traffic to a specific service, while the IP/Hostname is the "building". To specify a custom port, the `-p` flag is required.

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# Enter the password when prompted: bandit0
2. 🧗‍♀️ The Struggle
A critical detail for beginners in Linux is case sensitivity. Typing a single letter in uppercase when it should be lowercase will cause the command to fail. Additionally, understanding why the connection fails without the -p flag requires knowing the difference between default and custom service ports.

3. 🛡️ Red Team Takeaway
This level simulates Initial Access in a corporate penetration test. Gaining entry with valid credentials is just the first step. The ultimate goal of any offensive operation is to elevate privileges until you reach sudo (root access)—meaning you own the machine and control all commands. Furthermore, changing default ports (like 22 to 2220) is a common "security through obscurity" tactic used by sysadmins to evade automated scanners.

4. 🇪🇬 كبسولة سيريوس (Sirius Notes)
بص يصحبي اولا لما نيجي نقرأ السؤال ده ف هو هنا ان الهدف بتاع الليلفل ده انك تخش ع التيرمنال بتاعك بب حاجه اسمها ssh مش تنسي ابدا ان عندك اللينكس حساس للحروف ف انت لو كتبت الحروف كابتيال مش هيشتغل طيب هتقولي يعني اي sshده هقولك هو ان تتحكم في سيرفر عن بعد يعني معاك اكسيس ليه يعني هو هنا قالك اليوزر نيم والباسورد اللي هتخش بيهم انت مش هتقدر تخش الحساب الا لو معاك دول زي لما تشتغل في اي شركه انت لو خدت اكسيس هتخش بعدين انت تخترق سيرفر وقتها كل هدفك انت تعلي صلاحيتك انك تتحكم انت وتكون sudo يعني اي sudo انك صاحب الحساب اللي معاك كل الاوامر ده هدفنا في اي عملية اختراق اننا نوصل للsudo هتقولي تمام انا مفروض ابدا ب ده ازاي هقولك اول حاجه بعد م تفتح التيرمنال هتبدا ب اداه التحكم عن بعد اللي هو اختصار ل secure shell يعني اداة التحكم عن بعد الامنة يعني برتوكول شبكة مشفر وظيفتها اللي هنستخدمها هنا نتصل عن بعد بطريقة امنه عبر شبكات غير امنه
ssh  username@hostname
هتقولي اي اليوزر نيم والهوست نيم هنا هقولك ده تبع الشركه او المكان اللي انت مستدفه وهنا هو مديني The username is bandit0
وهتلاقي في اعلي الصفحة بالاخضر SSH Information
Host: bandit.labs.overthewire.org
يبقي كده احنا استفدنا اي
ssh bandit@bandit.labs.overthewire.org
هتقولي مهو مديني في الصفحة Port: 2220 اي البورت ده الـ Port (المنفذ) في شبكات الكمبيوتر هو عبارة عن رقم رقمي (من 1 إلى 65535) يعمل مثل "بوابة" أو "رقم شقة" داخل مبنى كبير (والذي هو جهاز الكمبيوتر أو الخادم).
بينما يُستخدم عنوان الـ IP لتحديد جهاز الكمبيوتر نفسه على الشبكة، يُستخدم الـ Port لتوجيه البيانات إلى البرنامج أو الخدمة المحددة التي تعمل داخل هذا الجهاز.
طيب هتقولي انا اعرف ان الافتراضي 22 ف ليه هنا مديني 2220  ؟ هقولك سؤال رائع هو بيغيره عشان الحماية
يبقي وصلنا للامر الاخير اللي هنكتبه وهو
ssh bandit0@bandit.labs.overthewire.org -p 2220
هتقولي اي -p دي هقولك طول ما البورت اتغير من الافتراضي اللي هو 22 وقتها لازم نحط -p وبس يصحبي
هيقولك دخل الباسورد هتدخل الباسورد الموقع ادهولك
وبس يصحبي كده كسبت الجولة الاولي 🥳🥳🥳🥳🥳🥳
