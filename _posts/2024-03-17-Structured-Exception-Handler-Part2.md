---
layout: post
published: true
title: Structured Exception Handler (SEH) Part 2
date: '2024-03-17'
tags:
- Windows
- Windows Internals
- Arabic-Article
- System Security
- Exploit Development
---


<div dir="rtl" markdown="1">

السلام عليكم ورحمة الله وبركاته 

المقالة هذه إستكمال للمقالة السابقة ، 

أخذنا لمحة عامة عن الـ `SEH` في المقالة السابقة ، في المقالة هذه -بإذن الله- سنتحدّث بتفصيل عن الـ `SEH` 

لنبدأ بسم الله 

# في حال حدوث `Exception` ، كيف يقوم نظام التشغيل بنداء الدّالة المَعنيّة ( الـ `Exception Handler` ) ؟

كبداية وقبل الخوض في تفاصيل الـ `SEH` ، سنحاول الاجابة على السؤال التالي ( السؤال واجابته بتفصيل من [المقالة](http://bytepointer.com/resources/pietrek_crash_course_depths_of_win32_seh.htm) القديمة التي ذكرناها سابقًا ) 

</div> 

> How does the OS know where to call when an exception occurs?


<div dir="rtl" markdown="1">
للإجابة على هذا السؤال يجب أن نعرف أولًا أن الـ `SEH` يعمل على مستوى الـ `Thread`  كما هو مذكور في [المقالة](http://bytepointer.com/resources/pietrek_crash_course_depths_of_win32_seh.htm) التي نناقشها

</div> 

> it's helpful to remember that structured exception handling works on a per-thread basis. That is, each thread has its own exception handler callback function

<div dir="rtl" markdown="1">
والمقصود بهذا، أن كل `Thread` يملك `SEH Chain` خاصة به، وهذه السلسلة ( سنعرف لاحقًا تفاصيلها ولماذا نعتبرها سلسلة) تحوي مجموعة الدوال المسؤولة عن التعامل مع الـ `Exceptions` الوارد حدوثها في هذا الـ `Thread` 

إذن ، في حال حدث خطأ ( `Exception` ) في أحد الـ `Threads` ، سيقوم نظام التشغيل بالرجوع لسلسة الـ `SEH` الخاصة بهذا الـ `Thread` ويبحث في هذه السلسلة عن الدالة المسؤولة عن التعامل مع هذا الـ `Exception` 

التفصيل أعلاه يقودنا لسؤالين مهمّين ، هما : 
* الأول : كيف يصل نظام التشغيل للـ `Thread` ( نقصد الوصول للمعلومات الخاصة بالـ `Thread` ) ؟
* الثاني : كيف نجد تفاصيل الـ `SEH Chain` ضمن الـ `Thread` ؟


لنبدأ بالاجابة على **السؤال الأول** ، 

في أنظمة ويندوز يوجد `Structure` اسمه [`Thread Environment Block (TEB)`](https://learn.microsoft.com/en-us/windows/win32/api/winternl/ns-winternl-teb)

الـ `TEB` بشكل مُختصر يحمل معلومات عن الـ `Thread` ، لسنا في هذه المقالة بصدد مناقشة بنية الـ `TEB`  وشكله، لكن سنتطرّق للجزء الذي يهمنا من هذا الـ `structure` وهو الـ `SEH` 
</div> 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/2.png)
*  Windows Internals](https://a.co/d/ejV363j)  - page : 201

<div dir="rtl" markdown="1">

في الصورة أعلاه، نجد أهم المعلومات التي يحملها الـ `TEB`  ، ونجد أن أول العناصر هو الـ ` Thread Information Block (TIB)` 

الـ `TIB` ماهو إلا `Structure` آخر ، في داخل الـ `TIB` نجد الـ `Exception List` أو الـ `SEH Chain` ( إجابة **السؤال الثاني** ) 

للتلخيص، في حال حدوث `Exception` في أحد 




</div> 
