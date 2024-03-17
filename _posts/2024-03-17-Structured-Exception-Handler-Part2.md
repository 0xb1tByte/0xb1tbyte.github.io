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

# في حال حدوث `Exception` ، كيف يقوم نظام التشغيل بنداء الدّالة المَعنيّة ( الـ `Exception Handler` ) 

كبداية وقبل الخوض في تفاصيل الـ `SEH` ، سنحاول الاجابة على السؤال التالي ( السؤال واجابته بتفصيل من [المقالة](http://bytepointer.com/resources/pietrek_crash_course_depths_of_win32_seh.htm) القديمة التي ذكرناها سابقًا ) 

</div> 

> How does the OS know where to call when an exception occurs?
