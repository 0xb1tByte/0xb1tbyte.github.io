---
layout: post
published: true
title: Structured Exception Handler (SEH) Part 3
date: '2024-04-29'
tags:
- Windows
- Windows Internals
- Arabic-Article
- System Security
- Exploit Development
---


<div dir="rtl" markdown="1">

السلام عليكم ورحمة الله وبركاته 

في المقالات السابقة ناقشنا الـ `SEH` من جوانب نظرية بتفصيل 

في هذه المقالة سنرى الـ `SEH` من الجانب العملي ، سنعتمد بشكل أساسي على الـ [`windbg`](http://www.windbg.org/)

بسم الله لنبدأ 

# نظرة على الـ `Structures`

ذكرنا في المقالات السابقة `Structures` عدة ، مثل الـ `TEB`  والـ `TIB` والـ `_EXCEPTION_REGISTRATION_RECORD` 

سنرى الآن كيف بإمكاننا النظر بتفصيل للتعريف ( Definition ) الخاص بكل `Structures` عبر الـ `windbg` 

سنستخدم الـ [`dt`](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/dt--display-type-) Command 

مثل المذكور في المرجع الخاص بالـ Command ، الـ [`dt`](https://learn.microsoft.com/en-us/windows-hardware/drivers/debuggercmds/dt--display-type-) تتيح لنا الاستعلام عن المتغيرات أو الـ `Data Types` والـ `Structures` وغيرها 

</div> 

> The dt command displays information about a local variable, global variable or data type. This can display information about simple data types, as well as structures and unions.

<div dir="rtl" markdown="1">

</div> 
