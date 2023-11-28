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

</div>
