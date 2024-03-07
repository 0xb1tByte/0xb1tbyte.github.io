---
layout: post
published: true
title: Structured Exception Handler (SEH)
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

بالنسبة للـ `Exception Handlers` ، فهو الجزء من البرنامج المسؤول عن التعامل مع الحالة الخاصة أو الأخطاء الوارد حدوثها ( الـ `Exception` )

كتاب [Windows System Programming](https://a.co/d/7loSby9) يذكر لنا تفاصيل جميلة توضّح الغاية من وجود الـ `Exception Handlers`
</div>

> Without some form of exception handling, an unintended program exception, such as dereferencing a pointer or division by zero, will terminate a program immediately without performing normal termination processing, such as deleting temporary files. SEH allows specification of a code block, or exception handler, which can delete the temporary files, perform other termination operations, and analyze and log the exception. In general, exception handlers can perform any required cleanup operations before leaving the code block.

<div dir="rtl" markdown="1">
