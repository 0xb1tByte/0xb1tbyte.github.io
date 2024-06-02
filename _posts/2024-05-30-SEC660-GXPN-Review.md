---
layout: post
published: true
title: SEC660 / GXPN Review
date: '2024-05-30'
tags:
- SANS
- GXPN
- SEC660
- Arabic-Article
---

<div dir="rtl" markdown="1">

السلام عليكم ورحمة الله وبركاته 

اجتزت بفضل الله هذا الشهر إختبار  [`GXPN`](https://www.giac.org/certifications/exploit-researcher-advanced-penetration-tester-gxpn/) ، وفي هذه المقالة إن شاء الله سأتحدث قليلًا عن : 

منهج الشهادة ( [`SEC660`](https://www.sans.org/cyber-security-courses/advanced-penetration-testing-exploits-ethical-hacking/) )

اختبار [`GXPN`](https://www.giac.org/certifications/exploit-researcher-advanced-penetration-tester-gxpn/)

مصادر / أدوات مُساعدة للتحضير لإختبار الـ [`GXPN`](https://www.giac.org/certifications/exploit-researcher-advanced-penetration-tester-gxpn/)


# منهج الشهادة ( [`SEC660`](https://www.sans.org/cyber-security-courses/advanced-penetration-testing-exploits-ethical-hacking/) )
منهج الشهادة مكوّن من الفصول الآتية : 

## Network Attacks for Penetration Testers
يُركّز هذا الفصل على الاختراقات على **مستوى الشبكة** ، وبما أن الحديث في هذا الفصل عن الشبكة، فبالطبع الهدف في عملية الاختراق هنا هي **أجهزة الشبكة**
كالـ `routers` , `switches`  و بعض **البروتوكولات والمفاهيم المتعلقة بالشبكة** مثل `NAC` , `IEEE 802.1X` , `VLANs` ، لا يعرض هذا الفصل إختراق البروتوكولات نفسها إنما إستغلال نقاط الضعف في تنفيذ هذه البروتوكولات ( Implementation ) 

كذلك يعرض هذا الفصل أمثلة على كيفية تجاوز أنظمة الـ `network access/admission control (NAC)` ، و أمثلة على اختراقات وتجاوز أنظمة الـ `captive portal` 

أنظمة الـ `captive portal` هي أول ما يستطيع المستخدم التواصل معه قبل الدخول والوصول للشبكة ، فالـ `captive portal` تتيح لنا تقييد الوصول للشبكة عن طريق سياسات ( policies ) يتم تعريفها ضمن هذه الأنظمة ، على سبيل المثال : لن يستطيع المستخدم النفاذ للشبكة قبل التأكد من تحديث نظام التشغيل ، وغيرها من السياسات التي تمكّننا من تقييد الوصول للشبكة إلا في حالة توفّر شروط معينة ، والاختراقات في هذا الجانب تتمحور حول إستغلال الضعف في تطبيق هذه السياسات ، أو وجود ثغرات فعليّة في نظام الـ `captive portal` نفسه 

كذلك يتحدّث هذا الفصل عن بعض الاختراقات المتعلّقة بالـ `VLANs` مثل الـ `VLAN Hopping / Manipulation`  ، ويناقش الفصل أيضًا بعض بروتوكولات سيسكو مثل : `DTP` , `CDP` , `HSRP`  ( معرفة هذه البروتوكولات وقراءتها في الـ Wireshark مهمة في الاختبار ) 

يتحدّث كذلك هذا الفصل بتفصيل عن أداتين متعلقة باختراقات الشبكة وهما : `Ettercap` و `Bettercap` ( في هذا الجزء توقّع في الاختبار أسئلة حول الـ `Ettercap Filters` )

أيضًا يعرض هذا الفصل اختراقات متعلّقة بالـ `IPv6` والـ Routing Protocols مثل الـ `OSPF` 

هذه كانت لمحة سريعة عن مواضيع هذا الفصل ، لمن أراد الاستزادة ، الـ [Syllabus](https://www.sans.org/cyber-security-courses/advanced-penetration-testing-exploits-ethical-hacking/) بها تفصيل أكثر ، وكذلك المعامل الخاصة بكل فصل 

## Crypto and Post-Exploitation
هذا الفصل يعرض أولًا بروتوكولات التشفير ومن ثم يناقش عملية مابعد الاختراق لأنظمة ويندوز و لينكس 

في الجزء الأول ( التشفير ) يعرض الفصل إختراقات حول ضعف في تنفيذ ( Implementation ) بروتوكولات التشفير ، ولا يناقش الفصل إختراق البروتوكول / الألقوريثم نفسه 

يعرض كذلك الفصل بعض الثغرات التي تم اكتشافها في أنظمة لم تُطبّق بروتوكولات التشفير بطريقة صحيحة ، أمثلة على هذه الاختراقات : 

Oracle Padding Attack 

POODLE Attack

KRACK Attack 

وفي الجزء الثاني من الفصل ( ما بعد الاختراق ) ، يعرض الفصل بعض الطرق لتجاوز الـ System Restrictions 

في أنظمة ويندوز تحدّث الفصل عن الـ `WDAC` والـ `SRPs` والـ `KIOSK Mode` والـ `AMSI` والـ `UAC` وفي أنظمة لينكس تحدّث الفصل عن الـ `Grsecurity` والـ `PaX` والـ `SELinux` وغيرها

وفي آخر الفصل، عرض لمشروع الـ [PowerSploit](https://github.com/PowerShellMafia/PowerSploit)  وأدواته الفرعية 


## Python, Scapy, and Fuzzing
هذا الفصل يبدأ بشرح لغة بايثون وبعض مكتباتها أو الـ  Modules مثل : sys , os , ctypes , pefile

بعد ذلك يعرض الفصل [Scapy](https://scapy.net/) وهي مكتبة في بايثون تُمكّننا من التعديل والتلاعب في الـ `Packet` لمجموعة من البروتوكولات ( فهم Scapy مهم جدًا في الاختبار )

والجزء الأخير من الفصل يتحدّث عن الـ Fuzzing ويعرض كذلك بعض أدوات الـ Fuzzing ومن الأدوات التي تم ذكرها في الفصل : 

أداة [Sulley](https://github.com/OpenRCE/sulley) وهي عبارة عن Fuzzing Engine ( فهم Sulley مهم في الاختبار ) 

أداة [BooFuzz](https://github.com/jtpereyda/boofuzz) وهي تطوير لأداة Sulley  ( بحسب تجربتي في الاختبارات التجريبية والاختبار الفعلي للمنهج لم يصادفني أي سؤال حول هذه الأداة ) 

أداة [DynamoRIO](https://github.com/DynamoRIO/dynamorio) وهي runtime code manipulation framework ( مهمة هذه الأداة في الاختبار كذلك ) 

## Exploiting Linux for Penetration Testers
يبدأ هذا الفصل بشرح الأساسيات مثل الذاكرة ولغة أسمبلي والـ Stack ( كتبت [سلسلة سابقة](https://0xb1tbyte.github.io/Stack-and-Win-x86-Calling-Conv-Part1/) عن الـ Stack ، لعلها تنفع من يدرس هذا الفصل ) 

يشرح كذلك الفصل أدوات مثل GDB و objdump ( مهمة هذه الأدوات في الاختبار ) 

بعد ذلك يناقش الفصل Linux Shellcode وكذلك windows Shellcode 

ومن ثم يشرح الفصل ثغرات الـ Buffer Overflow  ويناقش كذلك الفصل الـ ROP ( الفهم العام هنا يكفي، بكل تأكيد لن يتسع وقت الاختبار لبناء ROP ) 

وآخرًا يتحدث الفصل عن الـ Stack Protection في أنظمة لينكس وكيفية تجاوزها 

## Exploiting Windows for Penetration Testers

يبدأ الفصل هذا بشرح نظام ويندوز و مكوّناته الأساسية 

كذلك يشرح الفصل بعض المفاهيم في نظام ويندوز مثل الـ PEB , TEB  والـ SEH ( فهم هذه المفاهيم مهم جدًا في الاختبار ، كتبت [سلسلة](https://0xb1tbyte.github.io/Structured-Exception-Handler-Part1/) عن الـ SEH سابقًا، آمل أن تكون نافعة ) 

بعد ذلك يناقش الفصل الـ Exploit Mitigation Controls في نظام ويندوز ، وطريقة تجاوز بعضها 

وآخرًا يتحدّث الفصل عن الـ ROP 


# إختبار [`GXPN`](https://www.giac.org/certifications/exploit-researcher-advanced-penetration-tester-gxpn/)
هذه بعض الملاحظات السريعة عن الاختبار ، لكن الأفضل الرجوع للموقع 

الاختبار مدته ١٨٠ دقيقة تقريبًا 

عدد الأسئلة ٦٠ سؤال، يتخللها أسئلة عملية ( تقريبًا ٦ أسئلة ) 

في الأسئلة العملية يتم إعطاءك صلاحية الوصول لـ VM ( عبر المتصفح مباشرة ) 

أرشّح **وبشدة** خوض تجربة الاختبارات التجريبية ، فهي تحاكي الاختبار الحقيقي بنسبة كبيرة جدًا 

يوجد فترة استراحة ١٥ دقيقة ، بالامكان أخذها مرة واحدة ، أو على فترات 


في حالتي، خضت تجربة الاختبار Online ، التجربة لا تختلف عن اختبارات Offsec نوعًا ما من حيث فحص المكان والتأكد من الهوية ( لكن سيُطلب منك توفير **مستندين اثنين** لاثبات الهوية )

**غير مسموح** باستخدام الأقلام أو أي شيء للتدوين

كذلك **غير مسموح** باستخدام أي شاشة ثانوية ، فقط **شاشة واحدة** ( هذه نقطة مهمة لا أذكر أني قرأتها في أي مكان لكني عرفتها وقت بدء الاختبار ، فاستعدوا على هذا الأساس ) 

# مصادر / أدوات مُساعدة للتحضير لإختبار الـ [`GXPN`](https://www.giac.org/certifications/exploit-researcher-advanced-penetration-tester-gxpn/)

خلال تحضيري للاختبار وجدت [هذه الأداة](https://github.com/Ge0rg3/sans-index-creator) التي سهّلت عليّ بناء الـ Index ( لا أرشح دخول الاختبار بدون الـ Index أبدًا ) 

لكن يعيب الأداة أنها تُنشئ الـ Index بشكل تفصيلي جدًا وضخم ، في حالتي استخدمت الأداة كطريقة أوليّة ، ومن ثم قمت بالتعديل على الـ Index وإبقاء النقاط التي تهمني بعد الانتهاء من مذاكرة كل فصل 

قمت بنشر الـ Index الخاصة بي [هنا](https://github.com/0xb1tByte/GXPN-Journey) لمن أراد استخدامها

ختامًا، تمّت بفضل الله ، و أرجو أن تكون هذه النقاط والتفاصيل السابقة نافعة ومفيدة لكل من يستعد لهذه الاختبار 
</div> 
