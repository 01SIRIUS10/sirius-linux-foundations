### 🏴‍☠️ [Level 32 &rarr; Level 33]

**1. 🎯 The Objective & The "Why"**
- **The Question:** After all this git stuff, it's time for another escape. Good luck!
- **The Goal:** You SSH into bandit32 with your password, and instead of landing in a normal prompt, you're dropped into something bizarre — a shell where EVERY SINGLE LETTER you type gets automatically converted to UPPERCASE the moment you press it. Since Linux commands are case-sensitive and virtually all of them are lowercase, you literally cannot type a normal command like `ls` or `cat` — anything you type becomes `LS` or `CAT`, which don't exist as valid commands. You need to find a way to break out of this uppercase-only prison.
- **The Why:** This is a genuinely clever lesson about **shell variable expansion vs. literal typed input** — understanding that some things in a shell aren't "text you typed" that gets filtered/transformed, but rather VALUES the shell computes and substitutes internally, bypassing whatever text-transformation filter is watching your keystrokes. This distinction between "what you type" and "what the shell evaluates" is a subtle but powerful concept in shell exploitation and sandbox escapes.

---

### 2. 💻 The Execution (Step-by-Step)

You SSH into bandit32 normally:

```bash
ssh bandit32@bandit.labs.overthewire.org -p 2220
```
Enter the bandit32 password. Instead of a normal prompt, you'll notice that anything you try to type gets displayed back in ALL CAPS. Try typing something simple like `ls` and watch it show up as `LS` — and since there's no command called `LS` (commands are case-sensitive in Linux), you'll just get an error like `LS: command not found`.

You might ask, "How do I run ANY command if everything I type gets uppercased before it's even processed?" I'll tell you exactly why there's still a way out: this uppercase-converting wrapper is almost certainly implemented as a program that reads your KEYSTROKES and transforms the TEXT before treating it as a command. But there's something it CANNOT transform: **shell variables that get expanded/substituted by the shell ITSELF, internally, as a computed value — not as literal characters you typed.**

The magic variable we're going to use is `$0`.

```
$0
```

- `$0`: In any shell script or interactive shell session, this special built-in variable ALWAYS holds the name/path of the CURRENTLY RUNNING shell/program itself (for example, `/bin/bash` or `sh`). Critically, when you type `$0` and hit Enter, you're not typing the LETTERS "b-a-s-h" that could get intercepted and uppercased — you're typing the special CHARACTERS `$`, `0` — and the SHELL ITSELF looks up what that variable equals and substitutes it in, AFTER whatever uppercase-transformation happens to your raw keystrokes (or in some implementations, the variable expansion simply isn't touched by whatever character-remapping trick this level's uppercase shell uses, because it's evaluated as a shell built-in rather than treated as plain command text).

Type `$0` and press Enter. This causes the shell to spawn a brand NEW instance of itself (typically `/bin/bash` or `sh`) — and this NEW shell process is a genuinely normal, non-broken shell, completely free of the uppercase-transformation wrapper that was plaguing your previous session.

From this fresh shell, you can now type completely normal, lowercase commands:
```bash
whoami
```
This should confirm you're bandit32, but now in a WORKING shell. Let's grab the level 33 password the standard way:

```bash
cat /etc/bandit_pass/bandit33
```
Since bandit32 has permission to read its own password file (wait, actually this level's actual mechanic often has you needing to check `/etc/bandit_pass/bandit33` directly since escaping the uppercase shell alone grants you a normal environment as bandit32, and from there the password file is directly readable), this reveals your level 33 password directly.

---

### 3. 🌍 Real-World & Tactical Application

The `$0` trick exemplifies a whole CATEGORY of sandbox/restriction bypass techniques based on exploiting the gap between "user input filtering" and "shell-level evaluation."

- **🔴 Red Team Perspective:** Restricted environments (custom shells, kiosks, limited command interpreters used in CTFs and sometimes in real poorly-secured jump hosts) often implement their restrictions by filtering/transforming TEXT INPUT, but forget that shells have MANY built-in variables and expansion mechanisms (`$0`, `$PATH`, command substitution `$(...)`, backticks, environment variable expansion) that don't behave like plain typed characters. Red teamers specifically probe for these gaps when facing any kind of restricted shell or input sanitization system — the core question is always "is this filter operating on my literal keystrokes, or on the shell's FINAL evaluated command?" because those are often two very different things.
- **🔵 Blue Team Perspective:** If you're building ANY kind of restricted execution environment, filtering or transforming raw text input is a fundamentally WEAK security boundary — real sandboxing needs to happen at a deeper level (proper `chroot` jails, containers with locked-down syscalls via seccomp, dedicated restricted shell implementations like `rbash` combined with a strict `PATH`, or better yet, not relying on shell-level restriction at all). This level is a fun educational example of exactly why naive "filter the input" security controls are trivially bypassable by anyone who understands how shell expansion actually works under the hood.

---

### 4. 🚦 The Filter (Focus vs. Ignore)

- **FOCUS ON:** The core insight that `$0` (and similar shell-evaluated expressions) bypasses TEXT-level filters because the shell substitutes its VALUE rather than treating it as literal typed characters. This "filter operates on input text, not on evaluated shell semantics" gap is the single most important takeaway — remember it as a general escape PATTERN, not just a one-off trick.
- **IGNORE:** Don't worry about memorizing every other special shell variable (`$1`, `$#`, `$@`, etc.) right now — those matter more for actual script-writing later. For THIS level, `$0` specifically (because it evaluates to a shell path you can then "become") is the star of the show.

---

### 5. 🇪🇬   سيريوس (Sirius Notes - Egyptian Arabic)

> اللفل ده هيوريك حيلة ذكية جداً في الهروب من بيئة مقيدة، خليني اوريك.
>
> **1. 🎯 الهدف و "الليه"**
> - **السؤال:** بعد كل الحكاية دي بتاعة git، جه وقت هروب تاني. بالتوفيق!
> - **الهدف:** بتعمل SSH لـ bandit32 بباسوردك، وبدل ما توصل لبرومبت عادي، بتتحط في حاجة غريبة—شِل بيحول **كل حرف** بتكتبه لحروف كابيتال أوتوماتيك لحظة ما تدوسه. بما إن أوامر اللينكس حساسة لحالة الأحرف ومعظمها حروف صغيرة، عملياً مش هتقدر تكتب أمر عادي زي `ls` أو `cat`—أي حاجة تكتبها بتتحول لـ `LS` أو `CAT`، اللي مش موجودين كأوامر صحيحة. لازم تلاقي طريقة تكسر بيها السجن ده اللي كل حاجة فيه كابيتال بس.
> - **الليه؟** ده درس ذكي جداً عن **تفسير متغيرات الشِل مقابل الإدخال المكتوب حرفياً**—فهمك إن بعض الحاجات في الشِل مش "نص كتبته انت" بيتفلتر/يتحول، لكنها قيم **الشِل نفسه بيحسبها ويستبدلها داخلياً**، متخطياً أي فلتر تحويل نص بيراقب ضغطاتك على الكيبورد. الفرق بين "اللي انت كاتبه" و"اللي الشِل بيقيمه" مفهوم دقيق بس قوي جداً في استغلال الشِل والهروب من الصناديق المعزولة (sandbox escapes).
>
> **2. 💻 التنفيذ خطوة بخطوة**
>
> بتعمل SSH لـ bandit32 عادي:
>
> ```bash
> ssh bandit32@bandit.labs.overthewire.org -p 2220
> ```
> اكتب باسورد bandit32. بدل برومبت عادي، هتلاحظ إن أي حاجة تحاول تكتبها بتتعرض رجوع بحروف كابيتال كلها. جرب تكتب حاجة بسيطة زي `ls` وشوفها بتظهر كـ `LS`—وبما إنه مفيش أمر اسمه `LS` (الأوامر حساسة لحالة الأحرف في اللينكس)، هتاخد error زي `LS: command not found`.
>
> هتقولي "أشغل أي أمر إزاي لو كل حاجة بكتبها بتتحول لكابيتال قبل حتى ما تتعالج؟" هقولك بالظبط ليه لسه فيه مخرج: الغلاف ده اللي بيحول لكابيتال شبه مؤكد إنه برنامج بيقرا **ضغطاتك على الكيبورد** ويحول **النص** قبل ما يعامله كأمر. بس فيه حاجة مش هيقدر يحولها: **متغيرات الشِل اللي بتتفسر/تتستبدل بواسطة الشِل نفسه، داخلياً، كقيمة محسوبة—مش كحروف حرفية كتبتها انت.**
>
> المتغير السحري اللي هنستخدمه هو `$0`.
>
> ```
> $0
> ```
>
> - `$0`: في أي سكريبت شِل أو جلسة شِل تفاعلية، المتغير الخاص المدمج ده **دايماً** بيحمل اسم/مسار الشِل/البرنامج **الشغال حالياً** نفسه (مثلاً، `/bin/bash` أو `sh`). المهم جداً، لما تكتب `$0` وتدوس Enter، انت مش بتكتب **الحروف** "b-a-s-h" اللي ممكن تتلقط وتتحول لكابيتال—انت بتكتب **الرموز** الخاصة `$` و `0`—و**الشِل نفسه** بيدور على قيمة المتغير ده ويستبدلها، **بعد** أي تحويل كابيتال بيحصل على ضغطاتك الخام (أو في بعض التطبيقات، توسيع المتغير ببساطة مش بيتلمس من أي حيلة إعادة تخطيط حروف اللفل ده بيستخدمها، لإنه بيتقيّم كـ shell built-in بدل ما يتعامل كنص أمر عادي).
>
> اكتب `$0` ودوس Enter. ده بيخلي الشِل يفتح **نسخة جديدة** من نفسه (غالباً `/bin/bash` أو `sh`)—والعملية الجديدة دي شِل عادي وحقيقي، خالص تماماً من غلاف تحويل الكابيتال اللي كان مبهدل جلستك السابقة.
>
> من الشِل النظيف ده، دلوقتي تقدر تكتب أوامر عادية تماماً بحروف صغيرة:
> ```bash
> whoami
> ```
> ده المفروض يأكد إنك bandit32، بس دلوقتي في شِل شغال. يلا ناخد باسورد اللفل 33 بالطريقة القياسية:
>
> ```bash
> cat /etc/bandit_pass/bandit33
> ```
> بما إن bandit32 عنده صلاحية يقرا ملف باسورده هو (بالظبط ميكانيكية اللفل ده غالباً محتاجة تفحص `/etc/bandit_pass/bandit33` مباشرة بما إن الهروب من شِل الكابيتال لوحده بيديك بيئة عادية بصفة bandit32، ومن هناك ملف الباسورد قابل للقراءة مباشرة)، ده هيكشف باسورد اللفل 33 بتاعك مباشرة.
>
> **3. 🌍 التطبيق الواقعي والتكتيكي**
>
> حيلة `$0` بتمثل فئة كاملة من تقنيات تجاوز الصناديق المعزولة/القيود المعتمدة على استغلال الفجوة بين "فلترة إدخال اليوزر" و"تقييم مستوى الشِل".
>
> - **🔴 من منظور الـ Red Team:** البيئات المقيدة (شِلات مخصصة، أكشاك، مفسرات أوامر محدودة مستخدمة في تحديات CTF وأحياناً في jump hosts حقيقية مؤمنة بشكل ضعيف) غالباً بتنفذ قيودها عن طريق فلترة/تحويل **نص الإدخال**، بس بتنسى إن الشِلات عندها متغيرات وآليات توسيع مدمجة كتير (`$0`، `$PATH`، استبدال الأوامر `$(...)`، backticks، توسيع متغيرات البيئة) اللي مش بتتصرف زي حروف مكتوبة عادية. الـ Red Teamers بالتحديد بيفحصوا الفجوات دي لما يقابلوا أي نوع من الشِل المقيد أو نظام تنقية إدخال—السؤال الأساسي دايماً "الفلتر ده شغال على ضغطاتي الحرفية، ولا على الأمر النهائي اللي الشِل قيّمه؟" لإن دول غالباً حاجتين مختلفتين جداً.
> - **🔵 من منظور الـ Blue Team:** لو بتبني أي نوع من بيئة تنفيذ مقيدة، فلترة أو تحويل نص الإدخال الخام حد أمني **ضعيف من الأساس**—العزل الحقيقي لازم يحصل على مستوى أعمق (سجون `chroot` صحيحة، حاويات بـ syscalls مقفولة عن طريق seccomp، تطبيقات شِل مقيدة مخصصة زي `rbash` مع `PATH` صارم، أو الأفضل، عدم الاعتماد على تقييد مستوى الشِل خالص). اللفل ده مثال تعليمي ممتع بالظبط لسبب إن ضوابط أمان ساذجة زي "فلتر الإدخال" سهل تجاوزها من أي حد فاهم إزاي توسيع الشِل بيشتغل فعلياً تحت الغطا.
>
> **4. 🚦 الفلتر (ركز/تجاهل)**
> - **ركز على:** الفكرة الأساسية إن `$0` (وتعبيرات مشابهة يقيّمها الشِل) بتتخطى الفلاتر على **مستوى النص** لإن الشِل بيستبدل **قيمته** بدل ما يعامله كحروف مكتوبة حرفياً. الفجوة دي "الفلتر شغال على نص الإدخال، مش على دلالة الشِل المقيّمة" هي أهم استنتاج—افتكرها كنمط هروب عام، مش مجرد حيلة لمرة واحدة.
> - **تجاهل:** متقلقش من حفظ كل متغير شِل خاص تاني (`$1`، `$#`، `$@`، الخ) دلوقتي—دول أهم لكتابة سكريبتات فعلية بعدين. للفل ده بالتحديد، `$0` بالتحديد (لإنه بيتقيّم لمسار شِل تقدر بعدين "تتحول" ليه) هو نجم العرض.
