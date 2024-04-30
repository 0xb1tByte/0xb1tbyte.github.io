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

سنستخدم الـ Command التالية لإستعراض الـ `TEB Structure` 

</div> 

```bat
dt nt!_TEB
```

<div dir="rtl" markdown="1">

نلاحظ في الـ Command الـ `TEB` مسبوقة بالـ `nt` وهو اسم الـ `Module` المعرّف فيه هذا الـ `Structure`

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/6.png)

مثل ما ناقشنا في المقالات السابقة، الـ `TIB`  هو أول العناصر في الـ `TEB` 

لنلقي نظرة الآن على الـ `TIB Structure` ، بإمكاننا الاستعلام عن هذا الـ `Structure` باستخدام الـ Command الآتية : 

</div> 

```bat
dt _NT_TIB
```

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/7.png)


<div dir="rtl" markdown="1">

نرى في الصورة أعلاه أن الـ `ExceptionList` هي أول العناصر في الـ `TIB` وهي عبارة عن `Pointer` مثل ما ناقشنا في المقالات السابقة 

لنستعرض الآن الـ `_EXCEPTION_REGISTRATION_RECORD` ونرى كيف يبدو ، بإمكاننا استخدام الـ Command الآتية للإستعلام عن الـ `_EXCEPTION_REGISTRATION_RECORD`

</div> 


```bat
dt _EXCEPTION_REGISTRATION_RECORD
```

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/8.png)

<div dir="rtl" markdown="1">

مثل ما ذكرنا سابقًا ، الـ `ExceptionList` أو الـ `_EXCEPTION_REGISTRATION_RECORD` مكوّنة من عنصرين 

هما الـ `Next` والـ `Handler` ، الأول عبارة عن `Pointer` يُشير للعنصر التالي ، والثاني عبارة عن `Pointer` كذلك لكنه يُشير للدالة المسؤولة عن التعامل مع الـ `Exception` 

للمراجعة ، الصورة التالية تمثّل ما رأيناه بعد استعراض الـ `_EXCEPTION_REGISTRATION_RECORD Structure`

</div> 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/4.png)
