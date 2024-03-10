---
layout: post
published: true
title: Structured Exception Handler (SEH) Part 1 
date: '2024-02-27'
tags:
- Windows
- Windows Internals
- Arabic-Article
- System Security
- Exploit Development
---


<div dir="rtl" markdown="1">

السلام عليكم ورحمة الله وبركاته 

في هذه المقالة باذن الله راح نناقش الـ `Windows Structured Exception Handling (SEH)` 

المقالة تلخيص لقراءات متفرقة من عدّة مصادر، كتب ومقالات، بالنسبة للكتب فهذه هي القائمة : 

</div> 

- [Buffer. Overflow. Attacks. DETECT, EXPLOIT, PREVENT](https://a.co/d/fTPXdb6)
  - Chapter 8 : Windows Buffer Overflows
- [Windows System Programming](https://a.co/d/7loSby9)
  - Chapter 4 : Exception Handling
- [Practical Malware Analysis](https://a.co/d/aeXQFn7)
  - Chapter 15 : Anti-Disassembly   


<div dir="rtl" markdown="1">
بسم الله ، لنبدأ 

# ما المقصود بالـ `Exception` و الـ `Exception Handlers` ؟ 
قبل أن نخوض في الـ `SEH` لنعرف أولًا ماذا نقصد بالـ `Exception` و الـ `Exception Handlers`


بالنسبة للـ `Exception`  ، في كتاب [Buffer. Overflow. Attacks](https://a.co/d/fTPXdb6) نجد التعريف التالي :

</div>

>  an exception is a condition that occurs outside the normal flow of a program


<div dir="rtl" markdown="1">

إذا فالـ `Exception` أو الاستثناء بالمعنى الحرفي، هي حالة معينة تحصل وقت تنفيذ البرنامج، وهذه الحالة خارج سِياق عمل البرنامج الطبيعي، وفي أغلب الأحوال يُشار للـ `Exception` بالأخطاء التي تحصل وقت تنفيذ البرنامج 

أحد الأمثلة لحالات الأخطاء الممكن حدوثها وقت تنفيذ البرنامج هي استعمال `pointer` ويكون هذا الـ `pointer` يُشير إلى قيمة `NULL` 

أو مثلًا القسمة على صفر ، مثل المثال في الكود الآتي : 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/1.png)
*  [Buffer. Overflow. Attacks](https://a.co/d/fTPXdb6) - page : 350

بالنسبة للـ `Exception Handlers` ، فهو الجزء من البرنامج المسؤول عن التعامل مع الحالة الخاصة أو الأخطاء الوارد حدوثها ( الـ `Exception` )

كتاب [Windows System Programming](https://a.co/d/7loSby9) يذكر لنا تفاصيل جميلة توضّح الغاية من وجود الـ `Exception Handlers`
</div>

> Without some form of exception handling, an unintended program exception, such as dereferencing a pointer or division by zero, will terminate a program immediately without performing normal termination processing, such as deleting temporary files. SEH allows specification of a code block, or exception handler, which can delete the temporary files, perform other termination operations, and analyze and log the exception. In general, exception handlers can perform any required cleanup operations before leaving the code block.

<div dir="rtl" markdown="1">

إذًا وجود الـ `Exception Handlers` في البرنامج يمكّننا من إجراء عمليات إضافية في حال حدوث أي أخطاء ( `Exceptions` ) ، على سبيل المثال لا الحصر : في حال حدوث أي `Exception` خلال سير عمل البرنامج، يقوم البرنامج بحذف الملفات المؤقتة التي تم إنشاءها على النظام ( `cleanup operations` ) ، أو حتى قد يقوم البرنامج بكتابة تفاصيل هذا الخطأ في السجلات الخاصة بالبرنامج ( `logs files` ) ليعود لها المبرمج لاحقًا

إذًا فالـ `Exception Handlers` بكل بساطة هو جزء من البرنامج مسؤول عن العمليات اللازم إجراءها في حال حدوث أي `Exception`

# ماهو الـ Windows Structured Exception Handling (SEH)

بعد المقدمة السابقة، نستطيع الآن الحديث عن الـ `SEH` 

هذه [المقالة](http://bytepointer.com/resources/pietrek_crash_course_depths_of_win32_seh.htm) قديمة جدًا لكنها تشعّبت في شرح الـ `SEH`  ، سنأخذ بعض التفاصيل العامة منها 

في البداية، نجد المعلومة الآتية

</div>

> Win32 structured exception handling is an operating system-provided service.


<div dir="rtl" markdown="1">

تخبرنا المقالة أن الـ `SEH` هو **جزء من نظام ويندوز** وليست ميزة مرتبطة بلغة برمجة معينة ( وهذه نقطة مهمة ؛ أنا حقيقة في بداية القراءة حول الـ `SEH` كنت أعتقد أنها ميزة مرتبطة بـ C / C++ ) 

نجد كذلك نفس المعلومة السابقة مذكورة في [موقع مايكروسوفت](https://learn.microsoft.com/en-us/windows/win32/debug/about-structured-exception-handling)

</div>

> The structured exception handling and termination handling mechanisms are integral parts of the system; they enable the system to be robust. You can use these mechanisms to create consistently robust and reliable applications.


<div dir="rtl" markdown="1"> 

إذًا، فالـ `SEH` هي ميزة أو خدمة يقدمها نظام التشغيل ويندوز لإدارة الـ `Exceptions` المحتمل حدوثها خلال تنفيذ البرنامج 

لنأخذ المثال التالي حتى نستوعب الـ `SEH` أكثر ( المثال من نفس [المقالة](http://bytepointer.com/resources/pietrek_crash_course_depths_of_win32_seh.htm) القديمة )
</div>

> Imagine I told you that when a thread faults, the operating system gives you an opportunity to be informed of the fault. More specifically, when a thread faults, the operating system calls a user-defined callback function. This callback function can do pretty much whatever it wants. For instance, it might fix whatever caused the fault, or it might play a Beavis and Butt-head .WAV file.

<div dir="rtl" markdown="1"> 
نعرف بشكل عام أن البرنامج عبارة عن عملية ( `Process` ) والعملية تحتوي `Threads` واحد على الأقل أو أكثر ( المهتم بهذه التفاصيل أرشّح له الاطلاع على كتاب [Windows Internals](https://a.co/d/ejV363j) ) 

الآن لنتصوّر أنه خلال تنفيذ البرنامج حدث خطأ ( `Exception` ) في أحد الـ `Threads` الخاصة بالبرنامج ، مع وجود الـ `SEH` كميزة في نظام التشغيل، لن يقوم نظام التشغيل بانهاء البرنامج تمامًا ( لأنه حدث خطأ في أحد الـ `Threads` ) 

لكن سيتيح نظام التشغيل ويندوز للمبرمج الفرصة لتصحيح هذا الوضع عن طريق نداء دالة مسؤولة عن التعامل مع هذا الخطأ، هذه الدالة هي ما قمنا بذكره سابقًا ، وهي الـ `Exception Handlers` أو الـ `callback function` كما هو مذكور في المثال أعلاه

قبل أن نختم هذه المقالة ، هذا تعريف آخر جميل أيضًا وجدته في هذه [المقالة](https://limbioliong.wordpress.com/2022/01/09/understanding-windows-structured-exception-handling-part-1/) ولعله يلخّص العديد من النقاط المهمة التي ذكرناها حول الـ `SEH` 

</div>

> SEH can be described as a generalized error handling mechanism supported by the Windows OS. It is an Operating System feature and not tied to any programming language. It forms part of the Windows Application Binary Interface (ABI) so it’s a contract between an application and the Windows OS.


<div dir="rtl" markdown="1">  

نكتفي بهذا القدر من المعلومات، ونكمل بقية المواضيع حول الـ `SEH` في جزء آخر باذن الله

</div>

> ℹ️ [ملاحظة]
> هذه المقالة تمّت كتابتها خلال دراسة هذه المواضيع، فكل ما تم ذكره هنا قد يحتمل الخطأ، لكن بالإمكان العودة إلى المراجع التي إستندت عليها هذه المقالة 

