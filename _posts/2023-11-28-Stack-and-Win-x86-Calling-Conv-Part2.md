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
* Function's Stack Frame

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
3  // define a function named MyFunction
4  void MyFunction(int a, int b) 
5  {
6      int sum; // declare one integer variable
7      sum = a + b; // add a and b and store the result in sum
8      printf("The sum is: %d\n", sum); // print the result
9  }
10 
11 int main()
12 {
13     MyFunction(10, 5); 
14     return 0; // end the program
15 }
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


</div>
