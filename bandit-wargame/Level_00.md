# 🏴‍☠️ Bandit Level 0

**Target:** Log into the game using SSH with the provided credentials.
**Concepts Tested:** SSH syntax, non-standard ports, basic networking concepts.

---
### 1. 🎯 The Attack Vector
The goal of this level is to access the terminal using a protocol called **SSH (Secure Shell)**. SSH is an encrypted network protocol that allows us to securely connect to and control a remote server over an unsecured network.

The basic syntax to establish this connection is:
`ssh username@hostname`

Based on the provided details:
*   **Username:** `bandit0`
*   **Hostname:** `bandit.labs.overthewire.org`

So the initial command looks like this: `ssh bandit0@bandit.labs.overthewire.org`

However, the page specifies **Port: 2220**. What is a port? In computer networking, a port is a numerical value (ranging from 1 to 65535) that acts like a "gate" or an "apartment number" inside a large building (which represents the server/computer). While the IP address or hostname identifies the computer itself on the network, the port directs the data to the specific service running inside it.

The default port for SSH is 22. Why did they change it to 2220 here? Excellent question—it's changed for **protection and security**. 

Whenever the port is changed from the default 22, we must explicitly specify it using the `-p` flag. 

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# Enter the provided password when prompted, and the first round is won! 🥳
2. 🧗‍♀️ The Struggle
A crucial detail to never forget: Linux is strictly case-sensitive. If you type the commands or flags in capital letters, it simply will not work. Precision is key.

3. 🛡️ Red Team Takeaway
Like working in a real corporate environment, obtaining initial access to a server is just the beginning. Once you compromise a server, your ultimate goal in any hacking operation is to elevate your privileges to become sudo. Becoming sudo means you become the absolute owner of the account, possessing the authority to execute any and all commands.

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
