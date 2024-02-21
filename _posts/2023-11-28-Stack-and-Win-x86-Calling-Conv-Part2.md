---
layout: post
published: true
title: The Stack and Windows x86 Calling Conventions Part 2
date: '2023-12-25'
tags:
- Windows
- Windows Internals
- Arabic-Article
- System Security
---


<div dir="rtl" markdown="1">

السلام عليكم ورحمة الله وبركاته 

هذه المقالة استكمال للمقالة السابقة 

أهدف في هذه المقالة لشرح النقاط الآتية : 
* Calling Conventions 
* Function's `Prologue` and `Epilogue` 

لنبدأ بسم الله 

# Calling Conventions 

في كتاب `Practical Malware Analysis` نجد التعريف التالي 


</div>

> calling conventions govern the way the function call occurs. These conventions include the order in which parameters are placed on the stack or in registers, and whether the caller or the function called (the callee) is responsible for cleaning up the stack when the function is complete.

<div dir="rtl" markdown="1">

حتى نفهم التعريف لنأخذ المثال التالي 

</div>


```c
1  #include <stdio.h>
2  
3  // define a function named MyFunction that returns the sum value
4  int MyFunction(int a, int b)
5  {
6      int sum; 
7      sum = a + b; // add a and b and store the result in sum
8      return sum; // return the result
9  }
10 
11 int main()
12 {
13     int result;
14     result = MyFunction(10, 5); // call MyFunction and store the returned value in result
15     printf("The sum is: %d\n", result); // print the result
16     return 0; // end the program
17 }
```

<div dir="rtl" markdown="1">
البرنامج جدًا بسيط لجمع رقمين وطباعتهما

دالة الـ `main` هي التي قامت بالاستدعاء ، أو بلغة أخرى نسميها الـ `Caller` 

ودالة الـ `MyFunction` هي الدالة التي تم نداءها ، أو الـ `Callee` 

نلاحظ أن دالة الـ `main` قامت أيضًا بتمرير الـ `arguments` لدالة الـ `MyFunction`

الشرح هذا قد يبدو بسيط، لكن خلفه أشياء أخرى تحدث ، مثلًا : 

كيف سيتم تمرير الـ `arguments` من دالة الـ `main` الى الـ `MyFunction` ؟ 

وأين سيتم حفظ هذه الـ `arguments` ؟ في الـ `Stack` ؟ أم في الـ `registers` ؟ 

من سيقوم باعادة الـ `Stack` لوضعه الأولي قبل نداء الدالة `MyFunction` ؟ هل ستكون دالة `MyFunction` هي المسؤولة ؟ أم دالة الـ `main` ؟

جميع هذه الأسئلة التي طرحناها يُجيب عنها الـ `Calling Conventions` 

فالـ `Calling Conventions` بشكل مختصر أشبه بالقوانين التي **تحكم** كيف سيتم نداء الدوال وكيف سيتم تمرير المتغيرات للدالة المُستدعاة ( `Callee` ) ، كذلك أين و كيف سيتم تخزين هذه المتغيرات التي تم تمريرها، وأيضًا من سيتولى عملية تنظيف الـ `Stack` وإعادته لوضعه الأولي بعد أن تُكمل الدالة المُستدعاة ( `Callee` ) عملها

# Layout of a Function in Assembly
تطرقنا في المقالة السابقة للتعليمة `CALL` وعرفنا أن هذه التعليمة في لغة أسمبلي تمكّننا من نداء دالة 

ولو عدنا للكود الخاص بنفس البرنامج سنجد السطر الآتي الذي يوضح لنا نداء الدالة `MyFunction` 


![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/x86CallsAndStack/4.png)


ولو تعمقنا في كود الأسمبلي الخاص بأي دالة، سنجد أن كود الأسمبلي الخاص بالدالة يحمل الشكل الآتي 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/x86CallsAndStack/3.png)


نلاحظ أن بداية الدالة يكون الجزء المسمى `Prologue` ونهايتها يكون الـ `Epilogue` ، فماذا نقصد بهذين المفهومين ؟ 

## Function's `Prologue` 

في نفس كتابنا الرائع `Practical Malware Analysis` نجد التعريف التالي للـ `Prologue` 


</div>

> prologue—a few lines of code at the start of the function. The prologue prepares the stack and registers for use within the function

<div dir="rtl" markdown="1">
فالــ `Prologue` هو مجموعة من تعليمات الـ Assembly والغرض الأساسي منها هو تهيئة الـ Stack والـ registers قبل أن يتم تنفيذ الـ Body الخاص بالدالة

في الصورة التالية نجد مثال على الـ `Prologue` الخاص بالدالة `MyFunction` 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/x86CallsAndStack/5.png)

نلاحظ أن الـ `Prologue` عبارة عن هذه التعليمات 



</div>

```nasm
1  ; The function prologue
2  push ebp ; Save the old base pointer
3  mov ebp, esp ; Set the new base pointer
4  sub esp, 0CCH ; Allocate 192 bytes for the local variable
```

<div dir="rtl" markdown="1">

لنبدأ بالحديث عن الـ `Prologue` بالتفصيل الآن : 

* `2` : هذه التعليمة تحفظ قيمة الـ `EBP` في الـ Stack  ، وقيمة الـ `EBP` في هذه الحالة يعود للـ `Stack Frame` الخاص بالدالة السابقة، وفي مثالنا الدالة السابقة هي الـ `main` ، والغرض من هذه الخطوة حتى نستطيع العودة للـ `Stack Frame` الخاص بالدالة `main` بعد أن تُكمل الدالة `MyFunction` عملها

لعل الرسم التالي يوضح العملية بشكل أكبر : 
![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/x86CallsAndStack/7.png)

  
* `3` : بعد أن قمنا بحفظ قيمة الـ `EBP` الخاص بالدالة السابقة ( `Caller` ) في الـ `Stack` ، الآن نستطيع تحديث قيمة الـ `EBP` لبدء `Stack Frame` جديد ( الخاص بالدالة `MyFunction` ) ، و التعليمة في السطر `3` هي التي تقوم بهذا ، وهذه التعليمة بشكل مختصر تقوم بنسخ الـ `ESP` الى الـ `EBP` ، أي كأننا قمنا بتحريك الـ `EBP` ، الرسم التالي يلخص هذه الخطوة

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/x86CallsAndStack/8.png)


* `4` : في هذه التعليمة قمنا بطرح `0CCH` من قيمة الـ `ESP` ، والغرض من هذه الخطوة هو اتاحة مساحة للمتغيرات الخاصة بالدالة ( local variables ) ، أو بتعبير آخر قمنا بتحريك الـ `ESP` ، ولاحظ أننا قمنا بالطرح على الرغم من أننا نريد إضافة المتغيرات الى الـ Stack Frame ، والسبب في الطرح لأن الـ Stack ينمو بشكل عكسي كما عرفنا في المقالة السابقة ، فكلما تمت إضافة عناصر للـ Stack ، العنصر الأخير سيحمل عنوان أصغر من العنصر السابق ، الرسم التالي لعله يلخّص هذه العملية 


![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/x86CallsAndStack/9.png) 
</div>






<div dir="rtl" markdown="1">

## Function's `Epilogue` 
عرّف كتاب `Practical Malware Analysis` الـ `Epilogue` كالآتي :


</div>

> an epilogue at the end of a function restores the stack and registers to their state before the function was called.


<div dir="rtl" markdown="1">

الـ `Epilogue` هو العملية العكسية للـ `Prologue` 

فالغرض الأساسي من الـ `Epilogue` هو إعادة الـ Stack والـ registers للوضع الأولي بعد أن تُكمل الدالة المُستدعاة ( `Callee` ) عملها 

ولو أردنا رؤية الـ `Epilogue` في الكود الخاص بالدالة `MyFunction` سنجد الآتي :

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/x86CallsAndStack/6.png)

إذًا فالـ `Epilogue` مكوّن من العمليات الآتية : 


</div>

```nasm
1  ; The function epilogue
2  mov esp, ebp ; Restore the stack pointer
3  pop ebp ; Restore the base pointer
4  ret ; Return to the caller
```

<div dir="rtl" markdown="1">
لنبدأ بالتفصيل الآن : 


* `2` : في هذه التعليمة قمنا بنسخ قيمة الـ `EBP` الى الـ `ESP` ، أي كأننا قمنا بتحريك الـ `ESP` للأعلى وأصبح يشير لنفس العنوان الموجود في الـ `EBP` ، أو إن صح التعبير كأننا انتهينا من الـ Stack Frame الحالي ، الرسم التالي يلخّص هذه العملية

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/x86CallsAndStack/10.png)  

* `3` : في هذه التعليمة قمنا بعمل `POP` للقيمة الأخيرة في الـ Stack وتخزينها في الـ `EBP` ، والقيمة الأخيرة هي الـ `EBP` السابق ( انظر للسطر `2` ، الـ `ESP` يشير الى هذه القيمة ) ، وللتلخيص الغرض من هذه الخطوة هو إسترجاع الـ `EBP` الخاص بالدالة السابقة  ( `Caller` ) ، الرسم التالي يوضح هذه الخطوة

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/x86CallsAndStack/11.png)  

* `4` : تطرقنا في المقالة السابقة الى التعليمة `RET` وعرفنا أنها تقوم بحذف القيمة الأخيرة في الـ Stack وتخزّنها في الـ `EIP register` ، وفي هذه المرحلة ، القيمة الحالية في الـ Stack هو العنوان التالي لنداء الدالة `MyFunction` ، بتعبير آخر ، هذه التعليمة تُعيد التحكم للدالة `main` حتى يستكمل البرنامج عمله ، الرسم التالي لعله يلخص هذه العملية 


![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/x86CallsAndStack/12.png)  



نكتفي بهذا القدر في هذه المقالة، وسنكمل الحديث عن بقية المواضيع في مقالات لاحقة بمشيئة الله ، وإن أصبت فمن توفيق الله وحده.

ولمن أراد الاستزادة حول هذه المواضيع أرشّح وبشدة الاطلاع على سلسلة هذه المقالات : 


</div>

- [Cracking Assembly](https://medium.com/@sruthk/cracking-assembly-function-prolog-and-epilog-in-x86-cb3c3461bcd3)
- [x86 Assembly and Call Stack ](https://textbook.cs161.org/memory-safety/x86.html)



<div dir="rtl" markdown="1">

> ℹ️ [ملاحظة]
> هذه المقالة تمّت كتابتها خلال دراسة هذه المواضيع، فكل ما تم ذكره هنا قد يحتمل الخطأ، لكن بالإمكان العودة إلى المراجع التي إستندت عليها هذه المقالة 
</div>
