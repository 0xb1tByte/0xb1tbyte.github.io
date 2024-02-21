---
layout: post
published: true
title: The Stack and Windows x86 Calling Conventions Part 3
date: '2024-01-25'
tags:
- Windows
- Windows Internals
- Arabic-Article
- System Security
---

<div dir="rtl" markdown="1">

السلام عليكم ورحمة الله وبركاته 

هذه المقالة استكمال للسلسلة السابقة ، وستكون الأخيرة ( إلا إن اكتشفنا موضوع مُقارب للمواضيع هذه ورهيب 😁 ) 

المقالات السابقة كانت تمهيد -ومقدمة لابد منها- قبل الحديث عن مواضيع المقالة الأخيرة، والنقاط التي سنناقشها في مقالتنا هذه هي : 
* C Declaration (`__cdecl`) Calling Convention
* Standard Call (`__stdcall`) Calling Convention
* Fast Call (`__fastcall`) Calling Convention

المراجع في هذه المقالة بشكل أساسي هي الكتاب الرائع `Practical Malware Analysis` والمرجع الرسمي من مايكروسوفت ، كوننا نتحدث هنا عن الـ Calling Convention في أنظمة مايكروسوفت 

لنبدأ بسم الله ،

# ملاحظة

في كتاب `Practical Malware Analysis` قبل الحديث عن تفاصيل الـ Calling Conventions نجد هذه الملاحظة المهمة

</div> 

> NOTE Although the same conventions can be implemented differently between compilers, we’ll focus on the most common ways they are used.



<div dir="rtl" markdown="1">
بايجاز، نستطيع الفهم من الملاحظة السابقة أن الـ Calling Conventions قد يختلف تنفيذها ( implementation ) من Compiler لآخر ، لذلك مثل ما ذكر الكتاب ، سنركّز في المقالة هذه على المفاهيم العامة لكل Calling Convention 

# C Declaration (`__cdecl`) Calling Convention

# Standard Call (`__stdcall`) Calling Convention

# Fast Call (`__fastcall`) Calling Convention

</div>
