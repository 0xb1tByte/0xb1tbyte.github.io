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

## Python, Scapy, and Fuzzing

## Exploiting Linux for Penetration Testers

## Exploiting Windows for Penetration Testers




# إختبار [`GXPN`](https://www.giac.org/certifications/exploit-researcher-advanced-penetration-tester-gxpn/)

# مصادر / أدوات مُساعدة للتحضير لإختبار الـ [`GXPN`](https://www.giac.org/certifications/exploit-researcher-advanced-penetration-tester-gxpn/)
</div> 
