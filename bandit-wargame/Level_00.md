### 🏴‍☠️ [Level 0 &rarr; Level 1]

**1. 🎯 The Objective & The "Why"**
- **The Question:** The goal of this level is for you to log into the game using SSH. The host is `bandit.labs.overthewire.org`, on port `2220`. The username is `bandit0` and the password is `bandit0`.
- **The Goal:** Literally just prove you can open a terminal and connect to a remote server. That's it. No hacking, no tricks. This is the "hello world" of the entire wargame.
- **The Why:** Every single job in Red Teaming, Penetration Testing, DevOps, or System Administration revolves around **SSH (Secure Shell)**. You will NEVER physically touch 99% of the servers you attack or defend. You connect to them remotely, through an encrypted tunnel, and SSH is that tunnel. If you can't master this door, you can't get into the house.

**2. 💻 The Execution (Step-by-Step)**
Alright, you just opened your terminal (on Linux, Mac, or WSL/Git Bash on Windows). Here is exactly what you type:

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

Let's break this down piece by piece, because flags are useless if you don't know why they exist:
- `ssh`: This is the program itself. It stands for "Secure Shell." It tells your computer: "I want to open an encrypted communication channel with another computer."
- `bandit0@`: This is the **username**. In SSH syntax, we always structure it as `username@hostname`. Think of it like an email address: `bandit0` is *who* you are, and the part after `@` is *where* you're going.
- `bandit.labs.overthewire.org`: This is the **hostname** (the address of the server you want to connect to). It's the "house address" you are knocking on.
- `-p 2220`: This flag specifies the **Port**. You might ask, "Why do we need this? Isn't SSH always on the same port?" I'll tell you why: By default, SSH runs on Port 22. It's the "standard door" everyone knows about. Bandit intentionally moved their door to Port `2220` for security/organizational reasons (specific to this training). Without the `-p 2220` flag, your computer will try to knock on the default Port 22 door, find nobody home, and time out.

After running this, the terminal will ask:
```bash
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
Type `yes` and hit Enter. This is the server showing you its "digital ID card" (SSH key fingerprint) for the first time. You are just confirming, "Yes, I trust this ID, save it in my memory (known_hosts file) for future connections."

Finally, it asks for a password:
```bash
bandit0@bandit.labs.overthewire.org's password: 
```
Type `bandit0` (it won't show any characters on screen as you type—this is a **security feature**, so nobody looking over your shoulder can count your password length). Hit Enter. Boom, you're in.

**3. 🌍 Real-World & Tactical Application**
In a real job, you use this exact command daily to access production servers, cloud instances (AWS EC2, Azure VMs), or client machines during a penetration test.

- **🔴 Red Team Perspective:** Attackers actively scan the internet for open SSH ports (usually with tools like Nmap or Masscan). If they find one, they will try to **brute-force** it (guessing thousands of username/password combos) or exploit weak/default credentials—exactly like `bandit0:bandit0`. Never underestimate how many real corporate servers are breached simply because someone left default credentials active.
- **🔵 Blue Team Perspective:** Defenders NEVER run SSH on the default port 22 in high-security environments—they change it (like Bandit did with 2220) specifically to dodge automated bot scans that only check the default port. They also disable password authentication entirely and force SSH-Key-only login (we'll master this later), and use tools like `Fail2Ban` to auto-block IPs that fail login attempts repeatedly.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** The syntax structure `ssh user@host -p port`. Memorize this cold. You will type variations of this command literally thousands of times in your career.
- **IGNORE:** Don't worry about "fingerprint" verification errors for now, or SSH key theory (RSA/ED25519 algorithms). That's a much later, deeper topic. Right now, just focus on the connection mechanics.

---

> **5.(Sirius Notes - Egyptian Arabic)**
>
> بص  عشان تكون فاهم اللفل ده كويس، خليني اقولك هو عايز منك ايه بالظبط.
>
> **1. 🎯 الهدف و "ال ليه"**
> - **السؤال:** عايزك تعمل login على اللعبة باستخدام SSH. السيرفر اسمه `bandit.labs.overthewire.org`، والبورت (الباب) بتاعه `2220`. اليوزر نيم `bandit0` والباسورد `bandit0`.
> - **الهدف:** ببساطة يصحبي، عايزين نتأكد إنك عارف تفتح تيرمينال وتتصل بسيرفر بعيد عنك. مفيش هكر ولا حاجة، دي بس أول خطوة في السكة.
> - **ال ليه؟** هتقولي احنا بنعمل كده ليه؟ هقولك: أي شغلانة في الـ Red Teaming أو الـ Penetration Testing أو حتى الـ DevOps، أساسها إنك تتصل بسيرفرات مش موجودة قدامك في الأوضة. أنت مش هتلمس السيرفر بإيدك، هتتصل بيه عن بعد من خلال قناة مشفرة، والـ SSH هو القناة دي بالظبط. لو معرفتش تفتح الباب ده، مش هتقدر تدخل البيت خالص.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> تمام، فاتح التيرمينال؟ اكتب الأمر ده بالظبط:
>
> ```bash
> ssh bandit0@bandit.labs.overthewire.org -p 2220
> ```
>
> خليني افكك لك الأمر ده حتة حتة، عشان الفهم يبقى فاهم مش حافظ:
> - `ssh`: ده اسم البرنامج نفسه. معناه "Secure Shell"، وبيقولك "أنا عايز افتح خط اتصال مشفر مع جهاز تاني".
> - `bandit0@`: ده اليوزر نيم بتاعك. اتخيلها زي الايميل بالظبط، اللي قبل الـ `@` هو مين انت، واللي بعده هو انت رايح فين.
> - `bandit.labs.overthewire.org`: ده عنوان السيرفر، يعني "عنوان البيت" اللي انت طارقه.
> - `-p 2220`: دي الزتونة يا صحبي! هتقولي احنا محتاجينها ليه؟ هقولك: عادةً بروتوكول الـ SSH بيشتغل على بورت رقم 22، ده الباب المعروف اللي كل الناس عارفاه. لكن سيرفرات Bandit غيّرت الباب لبورت 2220 لأسباب أمنية. فلو نسيت تكتب الفلاج ده، الجهاز بتاعك هيروح يطرق على باب رقم 22 مش هيلاقي حد، وهيقولك "مفيش رد".
>
> بعد ما تكتب الأمر، هيسألك:
> ```bash
> Are you sure you want to continue connecting (yes/no/[fingerprint])?
> ```
> اكتب `yes` واضغط Enter. ده السيرفر بيوريك "بطاقة تعريفه" الرقمية لأول مرة، وانت بتقوله "تمام أنا واثق فيك، احفظها عندك في الذاكرة".
>
> بعدين هيطلب منك الباسورد:
> ```bash
> bandit0@bandit.labs.overthewire.org's password: 
> ```
> اكتب `bandit0` (هتلاقي إنك بتكتب ومفيش حروف ظاهرة على الشاشة، دي ميزة أمان عشان محدش واقف وراك يعرف طول الباسورد بتاعتك). دوس Enter، وخلاص أنت داخل.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> في شغلانتك الحقيقية، هتستخدم الأمر ده يومياً عشان توصل لسيرفرات شركات، أو أجهزة على الكلاود زي AWS و Azure.
>
> - **🔴 من منظور الـ Red Team:** الهاكرز بيعملوا سكان على الإنترنت كله عشان يدوروا على أي بورت SSH مفتوح (باستخدام أدوات زي Nmap). لو لقوا واحد، بيحاولوا يعملوا Brute-force (يعني يجربوا آلاف الباسوردات) أو يستخدموا باسوردات ضعيفة أو Default زي بتاعتنا دلوقتي `bandit0:bandit0`. صدقني، شركات كتير اتهكرت بس عشان حد نسي يغير الباسورد الافتراضي.
> - **🔵 من منظور الـ Blue Team:** فريق الحماية أبداً ما بيسيبوا الـ SSH شغال على البورت الافتراضي 22 في بيئة فيها حساسية عالية، بيغيروه (زي ما عملت Bandit ببورت 2220) عشان يهربوا من البوتات اللي بتدور بس على البورت الأساسي. كمان بيقفلوا خاصية الدخول بالباسورد نهائياً، ويخلوا الدخول بالـ SSH Key بس، وبيستخدموا أدوات زي `Fail2Ban` عشان توقف أي IP بيحاول يدخل غلط كذا مرة.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** شكل الأمر `ssh user@host -p port`. احفظه عن ظهر قلب، هتكتبه آلاف المرات في مشوارك.
> - **تجاهل:** متقلقش دلوقتي من موضوع الـ Fingerprint، أو نظرية مفاتيح الـ SSH (RSA/ED25519). دي مواضيع لاحقة وأعمق، دلوقتي ركز على ميكانيكية الاتصال بس.
