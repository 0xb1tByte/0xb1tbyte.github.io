---
layout: post
published: true
title: The Stack and Windows x86 Calling Conventions Part 1
date: '2023-11-28'
tags:
- Windows
- Windows Internals
- Arabic-Article
- System Security
---


<div dir="rtl" markdown="1">

السلام عليكم ورحمة الله وبركاته

هذه المقالة والمقالة القادمة -بمشيئة الله- سلسلة بسيطة لتلخيص قراءات متفرّقة حول الـ `Win x86 Calling Conventions & the Stack` في الكتب التالية : 

[Computer Security: A Hands-On Approach](https://a.co/d/8n1K5hR)

[Practical Binary Analysis](https://a.co/d/bHER3zs)

[Practical Malware Analysis](https://a.co/d/6vgWg0v)

لنبدأ بسم الله

# Program Memory Layout 
قبل أن نعرف الـ `Stack` لنعرف في البداية كيف تبدو الذاكرة الخاصة بالبرامج

نعرف أن العملية ( `Process` ) الخاصة بأي برنامج تحتاج لذاكرة ، هذه الذاكرة تتكوّن من الآتي ( نتحدث هنا عن الـ `user mode` ) 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/x86CallsAndStack/1.png)

[Computer Security: page 58 ](https://a.co/d/8n1K5hR)

بالنظر للصورة أعلاه نلاحظ أن الذاكرة الخاصة بالعملية تتكوّن من الآتي : 
* `Text Segment` : هذه المساحة من الذاكرة مخصّصة لتخزين الـ `executable code` الخاص بالبرنامج
* `Data Segment` : هذا المساحة مخصّصة لتخزين المتغيّرات التي يتم تعريفها كـ `static/global` ويتم إعطاءها قيَم ( `initialized` )
* `BSS Segment` : هذه المساحة مخصّصة لتخزين المتغيّرات التي يتم تعريفها كـ `static/global` لكن لم يتم تعريف القيَم الخاصة بهذه المتغيّرات ( `uninitialized`)
* `HEAP` :  مساحة من الذاكرة ديناميكية متاحة للمُبرمج لاستخدامها، وتتم ادارتها بأحد هذه الدوال `malloc`, `calloc`, `realloc` , `free` وغيرها
* `Stack` : محور حديثنا في هذه المقالة، وبشكل مختصر جدًا جدًا هي مساحة من الذاكرة مخصّصة للدوال الخاصة بالبرنامج

حتى نستوعب الجزء النظري، لنأخذ هذا البرنامج البسيط 
</div>

```c
1.	int x = 100;  // In Data segment
2.	int main()
3.	{
4.		int a = 2;  // In Stack 
5.		float b 2.5; // In Stack 
6.		static int y; // In BSS
7.	
8.		// Allocate memory on Heap
9.		int *ptr = (int *) malloc (2*sizeof(int));
10.	
11.		// values 5 and 6 stored on heap 
12.		ptr [O] = 5; // In Heap
13.		ptr [1] = 6; // In Heap
14.	
15.		free(ptr); 
16.		return 1;
17.	
18.	}
```


<div dir="rtl" markdown="1">

* `1` : نلاحظ أن المتغير `x` عبارة عن `global variable` وتم إعطاءه قيمة، بالتالي هذا المتغيّر سيكون مخزن في الـ `Data Segment`
* `4` & `5` : نلاحظ أن المتغيرات `a` و `b` عبارة عن متغيرات تم تعريفها داخل الدالة `main` وليست من نوع `static` وهذه المتغيرات تحمل قيَم ، بالتالي هذه المتغيرات سنجدها في الـ `Stack`
* `6` : نلاحظ أن المتغير `y` تم تعريفه كـ `static` لكنه لا يخزن أي قيمة ، بالتالي سنجد هذا المتغير في الـ `BSS Segment`
* `9` : هذا السطر يحوي أفكار مهمة وهي :

1 - نلاحظ أننا عرفنا `pointer variable` باسم `ptr` ونلاحظ أنه تم تعريفه في داخل الدالة ( `local variable` ) بالتالي سيتم تخزين الـ `ptr` في الـ `Stack` 

2 - نلاحظ أيضًا أنه قمنا بحجز مساحة ( `two integers` ) في الذاكرة باستخدام الدالة `malloc`، المساحة التي تم حجزها ستكون في الـ `Heap` والمتغير `ptr` يُشير لها ( لكنّه مُخزّن في الـ `Stack` ) 

* `12` : نلاحظ أننا أعطينا قيمة `5` للعنصر الأول في الـ `array` ( `ptr[0]` )  ، وسيتم تخزين الـ `ptr[0]` في الـ `Heap`  ( سيتم التخزين في الـ `Heap` لأن الـ `ptr` الذي قمنا بتعريفه في السطر رقم `9` يشير لمساحة في الـ `Heap` )
* `13` : هذا السطر مثل السطر السابق، لكننا قمنا بتعريف قيمة للعنصر الثاني في الـ `array` (`ptr[1]` ) وسيتم التخزين في الـ `Heap` كذلك 


بعد المقدمة البسيطة، لننتقل الآن للحديث عن الـ `Stack` 

# ماهو الـ `Stack` ؟ 

كالعادة سنبدأ بالجانب النظري أولًا ثم سننتقل لاحقًا للجانب العملي - الكود 

في كتاب `Practical Binary Analysis` نجد التعريف الآتي للـ `Stack`

> The stack is a memory region reserved for storing data related to function calls, such as return addresses, function arguments, and local variables. On most operating systems, each thread has its own stack. The stack gets its name from the way it’s accessed. Rather than writing values at random places in the stack, you do so in a last-in-first-out (LIFO) order. That is, you can write values by pushing them to the top of the stack and remove values by popping them from the top.

وفي كتاب `Computer Security` نجد الآتي :
> Stack is used for storing data used in function invocations. A program executes as a series of function calls. Whenever a function is called, some space is allocated for it on the stack for the execution of the function.

وفي الكتاب الرائع `Practical Malware Analysis` نجد تعريف مُختصر وشامل جدًا وهو : 
> Memory for functions.

لنبدأ من التعريف الأخير كونه الأبسط ونتشعّب في بقية الأفكار 

يخبرنا كتاب `Practical Malware Analysis` أن الـ `Stack` ماهو إلا مساحة في الذاكرة مخصصة للدوال ( `Functions` ) 

جميل ، طيب ماذا نعني بمساحة خاصة بالدوال ؟ 

نعرف أن البرنامج ماهو إلا سلسلة من تنفيذ مجموعة من الأوامر، وهذه الأوامر يتم تنظيمها وتعريفها في الدوال 

فبحسب التعريف، نستطيع التخيّل أن الـ `Stack` مساحة من الذاكرة وكل دالة في البرنامج تستخدم الجزء الخاص بها من الـ `Stack` (وهذا الجزء الخاص بكل دالة يسمى الـ `Stack frame` ، سنناقشه لاحقًا )

نعرف أيضًا أنه عند نداء أي دالة قد يتم تمرير مجموعة من الـ `Arguments` لها ، وهذه الـ `Arguments` أيضًا سيتم تخزينها في الـ `Stack` ، لأن كما ذكرنا سلفًا ، الـ `Stack` ماهو إلا مساحة في الذاكرة مُخصّصة للدوال وكل ما يرتبط بها 

كذلك في الدوال لدينا ما يُعرف بالـ `Local Variables` وهي المتغيرات التي يتم تعريفها في داخل الدالة ، هذه المتغيرات سنجدها كذلك في الـ `Stack`

نعرف كذلك أن الدالة بعد انتهاء عملها ستنفّذ التعليمة الأخيرة وهي الـ `return` والتي ستعيدنا لمكان النداء وبعد ذلك يستكمل البرنامج عمله في تنفيذ **السطر التالي** ، العنوان الخاص بالسطر التالي الذي يتبع نداء الدالة سيتم تخزينه كذلك في الـ `Stack` 

للتلخيص نستطيع القول أن الـ `Stack` يُخزّن القيَم التالية الخاصة بالدالة ( هذه ليست كل الأشياء التي يتم تخزينها في الـ `Stack` -كما سنعرف لاحقًا-، لكننا هنا نتحدث فقط عن الأمور المرتبطة بالدوال ) 
* `Arguments` : القيَم التي يتم تمريرها للدالة
* `Return Address` : العنوان الخاص بالسطر الذي يتبع نداء الدالة
* `Local Variables` : المتغيرات التي يتم تعريفها داخل الدالة

عرفنا الآن تقريبًا ما يتم تخزينه في الـ `Stack` ويكون مرتبط بالدوال ، لنعرف الآن كيف يعمل الـ `Stack`

## Stack Layout :
بشكل عام، الـ `Stack` هو أحد الـ `Data Structures` في الحاسب، أي عبارة عن خوارزمية يتم اتباعها لتنظيم وتخزين البيانات في الذاكرة ، لننظر للصورة التالية التي تلخّص لنا شكل الـ `Stack` 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/x86CallsAndStack/2.png)

[Practical Malware Analysis : page 78](https://a.co/d/6vgWg0v)

من الصورة أعلاه نستطيع القول أن الـ `Stack` ماهو إلا مجموعة من الـ `Stack Frames` وكل `Stack Frame` يكون مرتبط بدالة معينة إن صح القول 

تخزين البيانات في الـ `Stack` يتم بشكل متتالي، أي بمعنى أن العناوين متتالية، وكذلك في الـ `Stack` يتم تخزين البيانات بمنهجية `LIFO - Last in, First out` ، فلو أردنا تخيّل شكل العناصر التي يتم تخزينها في الـ `Stack` ستكون كأنها كومة من الكتب **متكدّسة** فوق بعضها

ولن نستطيع الوصول لآخر عنصر، بل يجب إزالة الكتاب في أعلى القائمة حتى نصل للكتاب الذي يليه، وهكذا .. سنناقش العمليات التي يتم إجراءها على الـ `Stack` بتفصيل لاحقًا لكن هذه لمحة سريعة 

يوجد أيضًا `registers` خاصة مرتبطة بالـ `Stack` وهي : 
* `ESP` :

١ - الـ `Stack Pointer` ويُشير إلى **آخر** عنصر تم إضافته إلى الـ `Stack` أو بعبارة أخرى يُشير الى الـ `Stack Top`

٢ - تتغيّر قيمة الـ `ESP` مع كل عملية **إضافة** ( `PUSH` ) أو **حذف** ( `POP` ) عنصر من الـ `Stack`

* `EBP` :

١ - الـ `Base Pointer` ويشير الى بداية الـ `Stack Frame` الخاص بالدالة

٢ - يكون الـ `EBP` **ثابت** خلال الـ `Stack Frame` الحالي

٣ - بما أن الـ `EBP` يكون ثابت خلال الـ `Stack Frame` الحالي فيتم استخدام الـ `EBP` لأجل **الوصول للقيم الأخرى** (`anchor` ) ضمن الـ `Stack Frame` 

كذلك الـ `Stack` يكبر بشكل **عكسي** ( في أغلب أنظمة التشغيل هذه هي الخوارزمية الشائعة، لكن بالامكان أن يتم تنفيذ الـ `Stack` بالشكل الآخر - تنمو العناوين بشكل تصاعدي ) أي بمعنى أنه كلما تم إضافة عنصر للـ `Stack` فسيحمل هذا العنصر عنوان في الذاكرة **أصغر** من العنصر الذي يسبقه

وبما أننا أتينا بذكر العمليات على الـ `Stack` لننتقل لمناقشة العمليات التي يتم إجراءها على الـ `Stack` و تؤثر في بنيته 

## Stack Instructions: 
نتحدّث هنا بالطبع عن عمليات `Assembly` تؤثر على الـ `Stack` وبنيته 
### `PUSH` 

هذه العملية تسمح لنا **إضافة** عنصر في الـ `Stack` وكما عرفنا سابقًا، سيتم إضافة هذا العنصر في الـ `Stack Top`

الرسم البسيط التالي يوضّح لنا كيف تؤثر هذا العملية على الـ `ESP` ، وكما ذكرنا سابقًا **مع كل إضافة أو حذف** ستتغيّر قيمة الـ `ESP` 

في حالة الـ `PUSH` سيُشير الـ `ESP` الى عنوان **أصغر** ، لأنه كما عرفنا سابقًا الـ `Stack` ينمو بشكل **عكسي**

</div>

```c
|           | <- High address
|    ...    |
|-----------| <- ESP before PUSH EAX
|    EAX    | <- Value of EAX register pushed onto the stack
|-----------| <- ESP after PUSH EAX
|    ...    |
|-----------| <- Low address
```

<div dir="rtl" markdown="1">
### `POP`
  
هذه العملية تسمح لنا **حذف** عنصر من الـ `Stack` ، ويتم حذف **آخر عنصر تمّت إضافته** والذي يكون موجود في الـ `Stack Top`

تؤثر هذه العملية أيضًا على قيمة الـ `ESP` و سيُشير الـ `ESP` الى عنوان **أكبر** من العنوان المخزّن سابقًا في الـ `ESP`

</div>


```c
|           | <- High address
|    ...    |
|-----------| <- ESP after POP EBX
|    EBX    | <- Value of EBX register popped from the stack
|-----------| <- ESP before POP EBX
|    ...    |
|-----------| <- Low address
```

<div dir="rtl" markdown="1">
### `CALL`

</div>

```c
|           | <- High address
|    ...    |
|-----------| <- ESP before CALL FOO
| ret addr  | <- Return address pushed onto the stack
|-----------| <- ESP after CALL FOO
|    ...    |
|-----------| <- Low address
```

<div dir="rtl" markdown="1">
### `RET`

</div>

```c
|           | <- High address
|    ...    |
|-----------| <- ESP after RET
| ret addr  | <- Return address popped from the stack
|-----------| <- ESP brfore RET
|    ...    |
|-----------| <- Low address
```


<div dir="rtl" markdown="1">
  
</div>
