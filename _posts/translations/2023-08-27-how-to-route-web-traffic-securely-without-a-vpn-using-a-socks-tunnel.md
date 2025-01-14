---
layout: translation
author: Othman Alikhan
published: true
lang: ar
title-ar: كيفية توجه حركة مرور الويب بشكل آمن بدون VPN باستخداك تفق SOCKS
title-en: How To Route Web Traffic Securely Without a VPN Using a SOCKS Tunnel
original-author: Michael Holley
original-date: 2020-10-26
original-article: >-
  https://www.digitalocean.com/community/tutorials/how-to-route-web-traffic-securely-without-a-vpn-using-a-socks-tunnel
category: translation
tags:
  - cybersecurity
  - sysadmin
  - linux
  - ubuntu
  - digitalocean
---

![pizza](https://www.digitalocean.com/_next/static/media/intro-to-cloud.d49bc5f7.jpeg)


## جدول المحتويات


- [المقدمة](#المقدمة)
- [1. المتطلبات الأساسية](#prerequisites)
- [2. الخطوة الأولى - إعداد النفق (macOS/Linux)](#step1a)
- [3. الخطوة الأولى - إعداد النفق (Windows)](#step1b)
- [4. الخطوة الثاني - تكوين Firefox لاستخدام النفق](#step2)
- [5. الخطوة الثالثة - إرجاع الوكيل في Firefox](#step3)
- [6. الخطوة الخامسة - استكشاف الأخطاء وإصلاحها: المرور عبر جدار الحماية](#step5)
- [7. الخاتمة](#conclusion)


<h3 id="introduction" lang="ar" dir="rtl">المفدمة</h3>

في نقطة معينة, يمكنك تجد نفسك على شبكة غير آمن أو عندها جدار آمن محدودة بالزيادة وتحتاج أن ولا واحد ينظر إلى حركة مرورك. حل واحد هو استخدام VPN, ولكن عدة VPNs تحتاج برنامج خاص على جهازك, ولا عندك القدرة أن تثبتها. لكن, لو تريد فقط أن تحمي تصفحك للويب, هناك بديل سريع, مجانًا, ومفيد: وكيل نفق SOCKS 5.

وكيل ‏SOCKS هي عبارة عن نفق مشفر يتم البرامج المكوِّن توجيه المرور إليه, ثم, من طرف الخادم, الوكيل يوجه المرور إلى الإنترنت العام. غير مشابه لVPN, وكيل SOCKS تحتاج إعداد على كل برنامج على الجهاز المستخدم, ولكن يمكنك تعد برامج بدون أي برنامج مستخدم خاص لو البرامج قادر على استخدام وكيل SOCKS. في طرف الخادم, تحتاج فقط إعداد SSH.

في هذه المدونة ستستخدم خادم Ubuntu 20.04 (ولكن أي نوع Linux يمكنك أن تستخدمه عن طريق SSH ستعمل), ومتصفح الويب Firefox كتطبيق عميل. في نهاية هذه المدونة ستكون جاهز للتصفح ماقع آمنًا عبر النفق SSH المشفر.


<h3 id="prerequisites" lang="ar" dir="rtl">المتطلبات الأساسية</h3>

- [خادم Linux عليه Ubuntu 20.04](https://www.digitalocean.com/products/linux-distribution/ubuntu/) (ستعمل أنواع Linux أخرى) مع مستخدم `sudo` وصول عبر SSH. لتعد هذا, يمكنك [أن تتبع المدونة لنا عن إعداد خادم Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04). [أذهب هنا لبناء تفقك على قطرة DigitalOcean](https://www.digitalocean.com/products/linux-distribution/ubuntu/)
- تطبيق للتكوين مع وكيل SOCKS, مثلًا المصفح الويب Firefox.
- للمستخدمين Windows, تحتاج إضافيًا إما PuTTY أو Windows Subsystem for Linux (WSL).

‏PuTTY مستخدم لإعداد نفق وكيل لمستخدمين Windows. مستخدمين macOS أو Linux عندهم الأدوات لإعداد النفق مثبت سابقًا.


<h3 id="step1" lang="ar" dir="rtl">الخطوة الأولى (macOS/Linux) - إعداد النفق</h3>

على جهازك المحلي, قم بإنشاء [مفتاح SSH](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2) إذا ما قمت بإنشاء قطرتك (Droplet) مع واحد جاهز. بمجرد إنشاء مفتاح, تأكد أن الجانب العام مضيف إلى الملف `authorized_keys` على قطرتك الSSH. ثم قم بفتح تطبيق موجه الأوامر لإنشاء نفق SSH مع وكيل SOCKS متمكن.

قم بإعداد النفق بهذا الأمر:

```sh
ssh -i ~/.ssh/id_rsa -D 1337 -f -C -q -N sammy@your_domain
```

شرح والوسيطات:i

‏- `-i`: المسار إلى مفتاح SSH التي ستستخدم للوصول إلى المضيف
‏- `-D`: يخبر SSH أن نحتاج نفق SOCKS على رقم منفذ معين (يمكنك أن تختار رقم بين `1025` إلى `65536`)
‏- `-f`: تفرغ العملية إلى الخلف
‏- `-C`: تضغط البيانات قبل الإرسال
‏- `-q`: استخدم الوضع الهادئ
‏- `-N`: يخبر SSH أن لا يرسل أمر لما ينشأ النفق

تأكد أن تبدل `sammy@your_domain` مع مستخدمك `sudo` وعنوان IP لخادمك/اسم المجال.

بمجرد تدخيل الأمر, فوريًا ستنتقل إلى مطالبة الأمر مرة أخرى دون أي علامة عن نجاح أو فشل; هذا عادي.

تأكد أن النفق يعمل مع هذا الأمر

```sh
ps aux | grep ssh
```

سترى سطر مع هذا الإخراج:

```md
Output
sammy    14345   0.0  0.0  2462228    452   ??  Ss    6:43AM   0:00.00 ssh -i ~/.ssh/id_rsa -D 1337 -f -C -q -N sammy@your_domain
```

يمكنك تخرج من تطبيقك الموجه الأوامر وسيبقى النفق مفتوحًا. هذا لأن استخدمنا وسيطة `-f` التي تضع جلسة SSH إلى الخلف.

> ملاحظة: لتنتهي نفقك تحتاج أن تمسك الPID عن طريق `ps`. في مثالنا, الPID هي `14345`. ثم نحن نستخدم الأمر `kill 14345`. نوضح الانتهاء في الخطوة الثالثة.


<h3 id="step1a" lang="ar" dir="rtl">الخطوة الأولى (Windows) - إعداد النفق</h3>

افتح [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)

لو ما ثبته إلى الحين, نزل PuTTY وأحفظه في أي مكان ترغبه. PuTTY لا يطلب حقوق مدير (admin) للتثبيت; فقط تزيل ال`.exe` وتشغيله.

اكمل الخطوات التالي لإعداد النفق:

1. من القسم `Section`, قم بإضافة اسم المضيف (أو عنوان IP) لخادمك, والمنفذ SSH (عادة 22)

![](https://assets.digitalocean.com/articles/socks5/wXDz8J7.png)

2. في اليسار, أذهب إلى: Connection > SSH > Tunnels
3. ادخل أي رقم منفذ مصدري بين `1025` إلى `65536`, مثلًا `1337`

![](https://assets.digitalocean.com/articles/socks5/ZLPgf4V.png)

4. اختار الزر المتغير
5. انقر على الزر `Add`
6. ارجع إلى `Session` على اليسار
7. ضيف اسم تحت `Saved Sessions` وانقر الزر `Save`
8. الآن انقر على الزر `Open` لإجراء الاتصال
9. ادخل اسم المستخدم ل`sudo` وكلمة السر للخادم لتسجيل الدخول

يمكنك تصغر نافذة PuTTY الآن, ولكن لا تغلقها. اتصالك الSSH مفروض يكون مفتوح:

> ملاحظة: يمكنك أن تحفظ اسم مستخدمك `sudo` ‏(sammy) ومفتاح الSSH في نفس الجلسة [باتباع تعليمات PuTTY SSH Key](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-keys-with-putty-on-digitalocean-droplets-windows-users). ثم ما تحتاج أن تدخل اسم المستخدم وكلمة السر كل مرة تفتح اتصال.


<h3 id="step2" lang="ar" dir="rtl">الخطوة الثاني - تكوين Firefox لاستخدام النفق</h3>

نظرًا لأن لديك نفق SSH, الوقت مناسب لتكوين Firefox لاستخدام هذا النفق. تذكر أنه لكي يعمل نفق SOCKS 5, تحتاج تطبيق محلي قادر أن ينفذ النفق; Firefox عندها هذا القدرة:

> هذا الخطوة نفسها على Windows, macOS, وLinux.

تأكد أن عندك رقم المنفذ الذي استخدمته في الأمر SSH; في أمثلتنا نحن استخدمنا `1337`.

(الخطوات التالي تم إجراءها مع Firefox إصدار 80 ولكن المفروض تشتغل مع أي إصدار, لكن المواقع للخيرات ربما تكون مختلفة.)

افتح Firefox.

1. في الزاوية في اليمنى العليا, انقر على أيقونة الهمبرغر للوصول إلى قائمة Firefox.
2. انقر على قائمة `Preferences` أو `Options`
3. قم بالتمرير إلى الأسفل وتحت `Network Settings` اختار الزر `Settings...`
4. تحت العنوان `Configure Proxy Access to the Internet` اختار `Select proxy configuration`
5. لل`SOCKS Host` ادخل `localhost` أو `127.0.0.1` وللمنفذ, استخدم المنفذ الخاص لنفقتك, `1337`
6. قريب من الأسفل, حدد المربع `Proxy DNS when using SOCKS v5`
7. انقر الزر `OK` للحفظ وأغلق إعداداتك

الآن, افتح تبويب آخر وأبدأ تصفح الويب. ستكون مستعد للتصفح الآمن خلال تفقك SSH. البيانات الراجعة من الموقع مشفر. إضافيًا, لأنك حددت الاختيار `Proxy DNS`, يتم تشفير عمليات بحث DNS بحيث الISP لك لم يرون ما ترى أو أين ذهبت للحصول إليه.

للتأكد أنك تستخدم وكيل, ارجع إلى `Network Settings` في Firefox وادخل رقم منفذ آخر وحفظ الإعدادات. الآن لو تجرب تصفح الويب, ستحصل رسالة خطأ: `The proxy server is refusing connections`. هذا يثبت أن Firefox يستخدم الوكيل وليس الاتصال الافتراضي فقط. بالبدالة, يمكنك أن تذهب إلى موقع يبين الIP العام, مثل [ipecho.net](https://ipecho.net/), والIP الراجع هي الIP لقطرتك SSH لأنها تمثل وكيلة لك الآن.


<h3 id="step3" lang="ar" dir="rtl">الخطوة الثالثة - إرجاع الوكيل في Firefox</h3>

عندما تخلص من احتياج إلى خصوصية تفق SSH, ارجع إلى الإعدادات وكيل الشبكة في Firefox. انقر على الزر الاختيار `Use system proxy settings` وانقر `OK`. الآن Firefox لا يتم استخدام النفق SOCKS ويمكننا أن نغلق النفق أيضًا. يمكنك ترك النفق مفتوح لتمكين وتعطيل الوكيل في Firefox حسب الرغبة ولكن لو تركت نفقك خاملاً لفترة طويلة, يمكن يغلق بنفسه.

<h4 lang="ar" dir="rtl">إغلاق النفق (macOS/Linux)</h4>

النفق التي أنشأناها سابقًا على جهازنا المحلي ذهبت إلى الخلف, لذلك إغلاق نافذة وجه الأوامر التي استخدمتها لفتح النفق لم تنتهي النفق. لتنهي النفق تحتاج أن نحدد المعرف العملية (Process ID; PID) باستخدام الأمر `ps`, ثم ننتهي العملية باستخدام الأمر `kill`.

لنبحث عن جميع العمليات `ssh` العاملة على جهازنا:

```sh
ps aux | grep ssh
```

أبحث عن السطر الذي يشبه الأمر الذي دخلته سابقًا لإنشاء النفق. هنا مخرج متماثل: 

```md
Output
sammy    14345   0.0  0.0  2462228    452   ??  Ss    6:43AM   0:00.00 ssh -i ~/.ssh/id_rsa -D 1337 -f -C -q -N sammy@your_domain
```

من بداية السطر, في أحد العمودان الأولى, يوجد رقم مكون من 3-5 أرقام. هذا الPID. الPID الممثل فوق هو `14345`.

الآن عند معرفة الPID, يمكنك أن تستخدم الأمر `kill` لإنهاء النفق. استخدم رقم PID لك لما تقتل العملية:

```sh
kill 14345
```

<h4 lang="ar" dir="rtl">إغلاق النفق (Windows)</h4>

اغلق نافذة PuTTY التي استخدمتها لإنشاء النفق. هذا كل شيء.


<h3 id="step4a" lang="ar" dir="rtl"> الخطوة الرابعة - إنشاء اختصارات للاستخدام المتكرر</h3>

لأنظمة macOS أو Linux, يمكننا إنشاء اسم مستعار (alias) أو إنشاء برنامج نصي لإنشاء النفق لنا بشكل سريع. فيما يلي طريقتان لأتمتة عملية النفق:

> هذه الطرق المختصرة كلاهما تحتاج بدون كلمة سر/بدون عبارة مرور إلى الخادم.



<h4 lang="ar" dir="rtl">برنامج نصي BASH قابلة للنقر</h4>

لو ترغب أيقونة ذلك, لما تنقر, ستبدأ النفق, نمكن ننشأ برنامج نصي BASH صغير ليفعل المهنة. البرنامج النصي يعدد النفق ثم يتم بتشغيل Firefox, ولكن لا تزال تحتاج أن تضيف إعدادات الوكيل يدويًا في Firefox في المرة الأولى.

على macOS, الملف الثنائي Firefox التي نمكن أن نطلقه من الوجه الأوامر هو داخل Firefox.app. على افتراض التطبيق في المجلد `Applications`, الملف الثنائي يكون موجود في `/Applications/Firefox.app/Contents/MacOS/firefox`.

على نظام Linux, لو ثبت Firefox عبر مستودع أو لو مثبت من قبل, سيكون موقعها في `/usr/bin/firefox`. يمكنك دائمًا تستخدم الأمر `which firefox` لتجد أين يتواجد Firefox على جهازك لو ما كان في الموقع الافتراضي.

في هذا البرنامج النصي, بدل مسار الملف إلى Firefox مع واحد مناسب لجهازك. قد تحتاج أيضًا لتعدِّل السطر SSH لينعكس الأمر الناجح الذي استخدمته سابقًا لإنشاء النفق.

باستخدام محرر النصوص مثل `nano`, انشأ ملف جديد:

```sh
nano ~/socks.sh
```

قم بإضافة السطور التالي:

```sh
# ملف: socks.sh

#!/bin/bash -e
ssh -i ~/.ssh/id_rsa -D 1337 -f -C -q -N sammy@`your_domain`
/Applications/Firefox.app/Contents/MacOS/firefox &
```

- بدِّل `1337` مع رقم المنفذ من رغبتك (يتطابق مع الذي دخلته في Firefox)
- بدِّل `sammy@your_domain` مع المستخدم SSH واسم المضيف أو IP
- بدِّل `/Applications/Firefox.app/Contents/MacOS/firefox` مع مسار ملف إلى الثنائي Firefox لجهازك

احفظ برنامج النصي. ل`nano`, اكتب `CONTROL + o`, ثم للخروج, اكتب `CONTROL + X`.

اجعل البرنامج النصي قابلًا للتنفيذ, بحيث لما تنقر عليه مرتين, ستقوم بالتنفيذ. من الموجه الأوامر, استخدم الأمر `chmod` لإضافة الحقوق للتنفيذ:

```sh
chmod +x /path/to/socks.sh
```

على macOS, قد يجب عليك أن تعمل خطوة إضافيًا لتقول لmacOS أن الملف .sh يجب أن ينفذ كبرنامج وأنه لا يفتح بمحرر النصوص. لتعمل هذا, انقر باليمين على الملف `socks.sh` واختار `Get Info`.


حدد موقع القسم `Open with:` وإذا كان المثلث الكشف لا يشير إلى الأسفل, انقر عليه لترى القائمة المنسدلة. Xcode يمكن يكون التطبيق الافتراضي.

![dropdown-menu](https://assets.digitalocean.com/articles/socks5/8TJ7dvX.png)

غيره إلى Terminal.app. لو Terminal.app ليس معرض, اختار `Other`, ثم اذهب إلى Applications > Utilities > Terminal.app (قد تحتاج أن تحدد القائمة المنسدلة `Enable` إلى `Recommended Applications` ّإلى `All Applications`.

لتفتح وكيلك SOCKS الآن, انقر مرتين على الملف `socks.sh`. البرنامج النصي ستفتح نافذة موجه الأوامر, بدأ الاتصال SSH, وينطلق Firefox. لو رغبت, يمكنك تغلق النافذة لموجه الأوامر في هذه المرحلة. طالمًا أنك احتفظت إعدادات الوكيل في Firefox, يمنكن تبدأ تصفح فوق اتصالك الآمن:

> هذا البرنامج النصي يساعدك بإنشاء الوكيل سريعًا, ولكن لا زال أن تفعل يدويًا الخطوات المذكورة فوق لعثور عملية SSH وقتله لما تنتهي.


<h4 lang="ar" dir="rtl">اسم مستعار في موجه الأوامر</h4>

لو وجدت نفسك في موجه الأوامر غالبًا وتريد أن تنشأ النفق, يمنك أن تنشئ اسم مستعار في موجه الأوامر ليفعل العمل لك.

أصعب شيء في إنشاء اسم مستعار هو معرفة مكان حفظ أمر الاسم المستعار.

أنواع Linux مختلفة وإصدارات macOS تحفظ الأسماء المستعار في أماكن مختلفة. أفضل مكان للفص هو بحث عن أحد الملفات التالي وبحث `alias` لترى أين الأسماء المستعار الأخرى محفوظة. الاحتمالات تضمن:

- ~/.bashrc
- ~/.zshrc
- ~/.bash_aliases
- ~/.bash_profile
- ~/.profile

بمجرد تحديد موقع الملف الصحيح, قم بإضافة الاسم المستعار التي في الأسفل إلى أي ملف عندك, أو إضافة في نهاية الملف. في المثال الأسفل, نحن نستخدم الاسم المستعار `firefox` لإقامة النفق الSOCKS, ولكن يمكنك أن تستخدم أي كلمة ترغبه كاسم مستعار:

```sh
# ملف: .bashrc

alias firesox='ssh -i ~/.ssh/id_rsa -D 1337 -f -C -q -N sammy@your_domain && /Applications/Firefox.app/Contents/MacOS/firefox &'
```

- بدِّل `1337` مع رقم المنفذ من رغبتك (يتطابق مع الذي دخلته في Firefox)
- بدِّل `sammy@your_domain` مع المستخدم SSH واسم المضيف أو IP
- بدِّل `/Applications/Firefox.app/Contents/MacOS/firefox` مع مسار ملف إلى الثنائي Firefox لجهازك

أسماء المستعار لك محملة فقط لما تنشأ موجه الأوامر جديد, لذلك قم بإغلاق موجه الأوامر لك وأبدأ واحد جديد. الآن لما تكتب:

```sh
firesox
```

هذه الاسم المستعار يجهز نفقك, ثم يطلق Firefox لك ويرجع إلى موجه الأوامر. تأكد Firefox لا زال محدد للاستخدام الوكيل. يمكنك الآن تصفح آمنًا.


<h3 id="step5" lang="ar" dir="rtl"> الخطوة الخامسة - استكشاف الأخطاء وإصلاحها: المرور عبر جدار الحماية </h3>

لو اتصالك شغال, فأمورك سليم ولا تحتاج أن تقرأ أكثر. ولكن, لو اكتشفت أنك لا نمكن تتصل بSSH إلى الخارج بسبب جدار حماية محدود, فأن في احتمال منفذ `22`, الذي يحتاج إلى إنشاء نفق, يتم منعه. لو تمكن تحكم بالإعدادات خادم الوكيل SSH (مع وصول الجذر إلى قطرة DigitalOcean, يمكنك أن تفعل هذا), تقدر تحدد SSH لاستماع إلى منفذ غير من `22`.

أي منفذ يمكنك أن تستخدم لم يكون ممنوع؟

غالبًا منافذ مفتوحة تضمن 80 (مرور الويب العام) و443 (TLS, مرور الويب الآمن).

لو خادم ٍٍSSH لك لم يخدم محتوى الويب, يمكننا أن نخبر SSH أن تستخدم أحد هذه المنافذ للويب للاتصال بدلًا من المنفذ الافتراضي `22`. `443` أفضل اختيار لأنه يتوقع أن يحتوي مرور مشفر على هذا المنفذ, ومرورنا SSH يكون مشفر.

من موقع بدون الجدار الحماية, SSH إلى قطرة DigitalOcean التي تستخدمها للوكيل أو استخدم الموجه الأوامر المبني في الداخل في لوحة التحكم على DigitalOcean.

قم بتحرير إعدادات SSH لخادمك:

```sh
sudo nano /etc/ssh/sshd_config
```

انظر إلى سطر `Port 22`.

يمكننا أن نبدل `22` كله أو نضيف منفذ ثاني لاستماع SSH عليه. نحن نختار SSH أن تسمع على منافذ عديدة, لذلك سنضيف سطر جديد تحت `Port 22` يقرأ `Port 443`. هنا مثال:

```sh
ملف: sshd_config

...
Port 22
Port 443

...
```

أعد تشغيل SSH ليعيد تحميل تكوين SSH التي غيرته. حسب نوع جهازك, الاسم لخادم SSH الخفي يمكن مختلف, ولكن احتمال يكون `ssh` أو `sshd`. لو أحدهما ما تشتغل, جرب الآخر:

```sh
sudo service ssh restart
```

للتأكد أن منفذ SSH لك الجديد يشتغل, افتح موجه الأوامر جديد (لا تغلق الحالي حتى الآن, احتياطًا لو تقفل نفسك بالغلط) وSSH باستخدام المنفذ الجديد:

```sh
ssh sammy@your_domain -p 443
```

لو نجحت, يمكنك الآن تسجيل الخروج من هذان موجه الأوامر وفتح نفقك SSH باستخدام المنفذ الجديد:

```sh
ssh -i ~/.ssh/id_rsa -D 1337 -f -C -q -N sammy@your_domain -p 443
```

الإعدادات لFirefox تكون نفس الشيء لأنها لا تعتمد على المنفذ SSH, فقط المنفذ للنفق (`1337` فوق).


<h3 id="conclusion" lang="ar" dir="rtl">الخاتمة</h3>


في هذا العصر الحديث هناك طرق عديدة للتصفح آمنًا لما يحتمل أن تكون على شبكة معادية, كwifi لمحل قهوة عام. في معظم الحالات, لو قادر أن تستخدم VPN لتأمن وتحمي كل مرورك فاستخدامه هو أفضل. ولكن وجود نفق SOCKS سيعطيك الأمانة التي تحتاج عند تصفحك الويب لما لم تقدر أو توثق VPN. نفق SOCKS سريع للإعداد والاستخدام في حالة ضرورية, وعندك كل التحكم عليه. هم اختيار ممتاز للتصفح الآمن.