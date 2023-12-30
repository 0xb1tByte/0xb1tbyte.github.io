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

* `2` :
* `3` :
* `4` :
  
</div>






<div dir="rtl" markdown="1">

## Function's `Epilogue` 
عرّف كتاب `Practical Malware Analysis` الـ `Epilogue` كالآتي :


</div>

> an epilogue at the end of a function restores the stack and registers to their state before the function was called.


<div dir="rtl" markdown="1">

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


* `2` :
* `3` :
* `4` :


نكتفي بهذا القدر في هذه المقالة، وسنكمل الحديث عن بقية المواضيع في مقالات لاحقة بمشيئة الله ، وإن أصبت فمن توفيق الله وحده.

ولمن أراد الاستزادة حول هذه المواضيع أرشّح وبشدة الاطلاع على سلسلة هذه المقالات : 


</div>

- [Cracking Assembly](https://medium.com/@sruthk/cracking-assembly-function-prolog-and-epilog-in-x86-cb3c3461bcd3)
- [x86 Assembly and Call Stack ](https://textbook.cs161.org/memory-safety/x86.html)



<div dir="rtl" markdown="1">

> ℹ️ [ملاحظة]
> هذه المقالة تمّت كتابتها خلال دراسة هذه المواضيع، فكل ما تم ذكره هنا قد يحتمل الخطأ، لكن بالإمكان العودة إلى المراجع التي إستندت عليها هذه المقالة 
</div>
