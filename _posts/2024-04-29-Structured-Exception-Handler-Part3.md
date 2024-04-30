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



<div dir="rtl" markdown="1">

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/7.png)


نرى في الصورة أعلاه أن الـ `ExceptionList` هي أول العناصر في الـ `TIB` وهي عبارة عن `Pointer` مثل ما ناقشنا في المقالات السابقة 

لنستعرض الآن الـ `_EXCEPTION_REGISTRATION_RECORD` ونرى كيف يبدو ، بإمكاننا استخدام الـ Command الآتية للإستعلام عن الـ `_EXCEPTION_REGISTRATION_RECORD`

</div> 


```bat
dt _EXCEPTION_REGISTRATION_RECORD
```


<div dir="rtl" markdown="1">

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/8.png)


مثل ما ذكرنا سابقًا ، الـ `ExceptionList` أو الـ `_EXCEPTION_REGISTRATION_RECORD` مكوّنة من عنصرين 

هما الـ `Next` والـ `Handler` ، الأول عبارة عن `Pointer` يُشير للعنصر التالي ، والثاني عبارة عن `Pointer` كذلك لكنه يُشير للدالة المسؤولة عن التعامل مع الـ `Exception` 

للمراجعة ، الصورة التالية تمثّل ما رأيناه بعد استعراض الـ `_EXCEPTION_REGISTRATION_RECORD Structure`

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/4.png)

# تتبُّع الـ `SEH Chain`

سنرى الآن كيف بإمكاننا عمل `trace` للـ `SEH Chain`

كما ذكرنا سابقًا، نستطيع الوصول الى الـ `Exception List` عن طريق الـ `TEB` ، فأول خطوة سنقوم بها هي الاستعلام عن محتوى الـ `TEB` ( في الجزء الأول استعلمنا عن بُنيته ، الآن نستعلم عن محتواه ) 

وسنقوم تحديدًا بمعرفة العنوان ( address ) الخاص بالـ `Exception List` ، نستطيع الاستعلام عن الـ `TEB` باستخدام الـ Command الآتية : 

</div> 

```bat
!teb
```

<div dir="rtl" markdown="1">

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/9.png)

نلاحظ أن الـ `Exception List` موجودة في العنوان : `01aeff70` 

الآن، باستخدام هذا العنوان بإمكاننا الوصول لأول عنصر في الـ `Exception List`  

ومن العنصر الأول سنستطيع الوصول للعنصر الثاني ( لأن الـ `_EXCEPTION_REGISTRATION_RECORD` كما ذكرنا من قبل مكوّن من عنصرين، أحدهما الـ `Next` ، والذي يُشير للعنصر التالي في الـ `List` ) 

ومن العنصر الثاني سنستطيع الوصول للعنصر الثالث، وهكذا حتى نصل لآخر عنصر في الـ `Exception List`

بعد معرفة عنوان الـ `Exception List` بإمكاننا الاستعلام عن أول العناصر باستخدام الـ Command التالية : 


</div> 

```bat
dt _EXCEPTION_REGISTRATION_RECORD 01aeff70
```

<div dir="rtl" markdown="1">

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/10.png)

نلاحظ أن العنصر التالي من الـ `List` موجود في العنوان `0x01aeffcc` 

وبإمكاننا الوصول للعنصر الثاني من الـ `List` باستخدام نفس الـ Command السابقة، مع تغيير العنوان للعنصر الذي نود الإستعلام عنه ، وفي حالتنا ستكون الـ Command كالآتي : 

</div> 

```bat
dt _EXCEPTION_REGISTRATION_RECORD 0x01aeffcc
```

<div dir="rtl" markdown="1">

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/11.png)

نرى من الصورة السابقة أن العنوان الخاص بالعنصر الثالث هو : `0x01aeffe4`

ونستطيع الاستعلام عنه كالآتي : 

</div> 

```bat
dt _EXCEPTION_REGISTRATION_RECORD 0x01aeffe4
```

<div dir="rtl" markdown="1">

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/12.png)

نلاحظ من الصورة أعلاه، أننا وصلنا الى آخر الـ `SEH Chain` ، لأن الـ `Next` يُشير الى العنوان `0xffffffff`

ومثل ما عرفنا في المقالات السابقة، آخر عنصر في الـ `SEH Chain` عبارة عن الدالة الخاصة بالنظام، والتي سيتم تنفيذها في حال لم يجد نظام التشغيل دالة تقوم بمعالجة الـ `Exception`

والى هنا نكتفي، 

بتوفيق الله انتهت سلسلة مقالات الـ `SEH` ، أو ربما نُكمل -إن شاء الله- في مقالات لاحقة في حال وجدنا ما يستحق الذكر 

</div> 


> ℹ️ [ملاحظة]
> هذه المقالة تمّت كتابتها خلال دراسة هذه المواضيع، فكل ما تم ذكره هنا قد يحتمل الخطأ، لكن بالإمكان العودة إلى المراجع التي إستندت عليها هذه المقالة

