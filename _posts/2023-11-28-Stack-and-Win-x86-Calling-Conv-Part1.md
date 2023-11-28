---
layout: post
published: true
title: The Stack and Windows x86 Calling Conventions Part 1
date: '2023-11-28'
tags:
- Windows
- Windows Internals
- Arabic-Article
- System Security
---


<div dir="rtl" markdown="1">

السلام عليكم ورحمة الله وبركاته

هذه المقالة والمقالة القادمة -بمشيئة الله- سلسلة بسيطة لتلخيص قراءات متفرّقة حول الـ `Win x86 Calling Conventions & the Stack` في الكتب التالية : 

[Computer Security: A Hands-On Approach](https://a.co/d/8n1K5hR)

[Practical Binary Analysis](https://a.co/d/bHER3zs)

[Practical Malware Analysis](https://a.co/d/6vgWg0v)

لنبدأ بسم الله

# Program Memory Layout 
قبل أن نعرف الـ `Stack` لنعرف في البداية كيف تبدو الذاكرة الخاصة بالبرامج

نعرف أن العملية ( `Process` ) الخاصة بأي برنامج تحتاج لذاكرة ، هذه الذاكرة تتكوّن من الآتي ( نتحدث هنا عن الـ `user mode` ) 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/x86CallsAndStack/1.png)

[Computer Security: page 58 ](https://a.co/d/8n1K5hR)

بالنظر للصورة أعلاه نلاحظ أن الذاكرة الخاصة بالعملية تتكوّن من الآتي : 
* `Text Segment` : هذه المساحة من الذاكرة مخصّصة لتخزين الـ `executable code` الخاص بالبرنامج
* `Data Segment` : هذا المساحة مخصّصة لتخزين المتغيّرات التي يتم تعريفها كـ `static/global` ويتم إعطاءها قيَم ( `initialized` )
* `BSS Segment` : هذه المساحة مخصّصة لتخزين المتغيّرات التي يتم تعريفها كـ `static/global` لكم لم يتم تعريف القيَم الخاصة بهذه المتغيّرات ( `uninitialized`)
* `HEAP` :  مساحة من الذاكرة ديناميكية متاحة للمُبرمج لاستخدامها، وتتم ادارتها بأحد هذه الدوال `malloc`, `calloc`, `realloc` , `free` وغيرها
* `Stack` : محور حديثنا في هذه المقالة، وبشكل مختصر جدًا جدًا هي مساحة من الذاكرة مخصّصة للدوال الخاصة بالبرنامج


</div>
