---
layout: post
published: true
title: Structured Exception Handler (SEH) Part 2
date: '2024-03-18'
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

# في حال حدوث `Exception` ، كيف يقوم نظام التشغيل بنداء الدّالة المَعنيّة ( الـ `Exception Handler` ) ؟

كبداية وقبل الخوض في تفاصيل الـ `SEH` ، سنحاول الاجابة على السؤال التالي ( السؤال واجابته بتفصيل من [المقالة](http://bytepointer.com/resources/pietrek_crash_course_depths_of_win32_seh.htm) القديمة التي ذكرناها سابقًا ) 

</div> 

> How does the OS know where to call when an exception occurs?


<div dir="rtl" markdown="1">
للإجابة على هذا السؤال يجب أن نعرف أولًا أن الـ `SEH` يعمل على مستوى الـ `Thread`  كما هو مذكور في [المقالة](http://bytepointer.com/resources/pietrek_crash_course_depths_of_win32_seh.htm) التي نناقشها

</div> 

> it's helpful to remember that structured exception handling works on a per-thread basis. That is, each thread has its own exception handler callback function

<div dir="rtl" markdown="1">
والمقصود بهذا، أن كل `Thread` يملك `SEH Chain` خاصة به، وهذه السلسلة ( سنعرف لاحقًا تفاصيلها ولماذا نعتبرها سلسلة) تحوي مجموعة الدوال المسؤولة عن التعامل مع الـ `Exceptions` الوارد حدوثها في هذا الـ `Thread` 

إذن ، في حال حدث خطأ ( `Exception` ) في أحد الـ `Threads` ، سيقوم نظام التشغيل بالرجوع لسلسلة الـ `SEH` الخاصة بهذا الـ `Thread` ويبحث في هذه السلسلة عن الدالة المسؤولة عن التعامل مع هذا الـ `Exception` 

التفصيل أعلاه يقودنا لسؤالين مهمّين ، هما : 

الأول : كيف يصل نظام التشغيل للـ `Thread` ( نقصد الوصول للمعلومات الخاصة بالـ `Thread` ) ؟

الثاني : كيف نجد تفاصيل الـ `SEH Chain` ضمن الـ `Thread` ؟


لنبدأ بالاجابة على **السؤال الأول** ، 

في أنظمة ويندوز يوجد `Structure` اسمه [`Thread Environment Block (TEB)`](https://learn.microsoft.com/en-us/windows/win32/api/winternl/ns-winternl-teb)

الـ `TEB` بشكل مُختصر يحمل معلومات عن الـ `Thread` ، وفي أنظمة الـ `32bit` نستطيع الوصول للـ `TEB` عن طريق الـ `FS register` 

لسنا في هذه المقالة بصدد مناقشة بنية الـ `TEB`  وشكله، لكن سنتطرّق للجزء الذي يهمنا من هذا الـ `Structure` وهو الـ `SEH` 
</div> 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/2.png)
*  [Windows Internals](https://a.co/d/ejV363j)  - page : 201

<div dir="rtl" markdown="1">

في الصورة أعلاه، نجد أهم المعلومات التي يحملها الـ `TEB`  ، ونجد أن أول العناصر هو الـ `Thread Information Block (TIB)` 

الـ `TIB` ماهو إلا `Structure` آخر ، وفي داخل الـ `TIB` نجد الـ `Exception List` أو الـ `SEH Chain` ( إجابة **السؤال الثاني** ) 

هذا هو الـ `structure definition` الخاص بالـ `TIB` ( المرجع هذه [المقالة](https://limbioliong.wordpress.com/2022/01/09/understanding-windows-structured-exception-handling-part-1/) وملف الـ `winnt.h` ) 



</div> 

```c
typedef struct _NT_TIB {
    struct _EXCEPTION_REGISTRATION_RECORD *ExceptionList;
    PVOID StackBase;
    PVOID StackLimit;
    PVOID SubSystemTib;
#if defined(_MSC_EXTENSIONS)
    union {
        PVOID FiberData;
        DWORD Version;
    };
#else
    PVOID FiberData;
#endif
    PVOID ArbitraryUserPointer;
    struct _NT_TIB *Self;
} NT_TIB;
typedef NT_TIB *PNT_TIB;
```

<div dir="rtl" markdown="1">

نلاحظ أن أول العناصر في الـ `TIB` عبارة عن `pointer` 

يُشير هذا الـ `pointer` الى أول عنصر من `Linked List` ، هذه الـ `Linked List` ماهي إلا الـ `Exceptions List` ( الـ `SEH Chain` ) 

لنناقش الآن بتفصيل الـ `SEH Chain`


# `SEH Chain` ( `Exceptions List` ) 

عرفنا حتى الآن كيف يستطيع نظام التشغيل الوصول للـ `SEH Chain` ، وهذه العملية تتم كالآتي :

في حال حدوث `Exception` في أحد الـ `Threads` سيقوم نظام التشغيل بالرجوع للـ `TEB` ( لأن الـ `TEB` هو المكان الذي سيجد فيه نظام التشغيل التفاصيل عن الـ `Thread` )

ويستطيع نظام التشغيل الوصول للـ `TEB` عن طريق الـ `FS register` ( في أنظمة الـ `32bit` ) 

بعد ذلك سيقوم نظام التشغيل بالوصول للـ `TIB` ، وفي الـ `TIB` سيجد نظام التشغيل الـ `pointer` الذي يُشير لأول عنصر في الـ `SEH Chain` 

الآن ، ماهي الـ `SEH Chain` وكيف يبدو شكلها ؟ 

في كتاب [Practical Malware Analysis](https://a.co/d/aeXQFn7) نجد تعريف جميل للـ `SEH Chain` 

</div> 

> The SEH chain is a list of functions designed to handle exceptions within the thread. Each function in the list can either handle the exception or pass it to the next handler in the list. If the exception makes it all the way to the last handler, then it is considered to be an unhandled exception. The last exception handler is the piece of code responsible for triggering the familiar message box that informs the user that “an unhandled exception has occurred.”


<div dir="rtl" markdown="1">

فالـ `SEH Chain` هي `list` ( بالتحديد `linked list` ) مكوّنه من مجموعة من الدّوال ( الـ `Exception Handlers` )

في حال حصول أي `Exception` سيقوم نظام التشغيل بالمرور على هذه القائمة ، ويرى أي دالة ستكون المسؤولة عن التعامل مع هذا الـ `Exception` 

في حال لم يجد نظام التشغيل أي دالة تتكفّل بالتعامل مع هذا الخطأ ، فسيتم نداء الدالة الخاصة بنظام التشغيل ( وهذه الدالة هي العنصر الأخير في الـ `Linked List` /  `SEH Chain` ) 

وعادةً هذه الدالة تقوم بانهاء العملية واظهار رسالة خطأ للمستخدم 

نجد كذلك المعلومة الآتية في كتاب [Practical Malware Analysis](https://a.co/d/aeXQFn7) : 

</div> 

> The SEH chain is a simple linked list of 8-byte data structures called EXCEPTION_REGISTRATION records


<div dir="rtl" markdown="1">

كل عنصر في الـ `Linked List` يسمى `EXCEPTION_REGISTRATION` 

والـ `EXCEPTION_REGISTRATION` سعته `8` بايت 

يسمى الـ `EXCEPTION_REGISTRATION` أحيانًا بالـ `_EXCEPTION_REGISTRATION_RECORD` مثل ماهو مُعرّف في الـ `Structure` الخاص بالـ `TIB` 

</div>


```c
typedef struct _NT_TIB {
    struct _EXCEPTION_REGISTRATION_RECORD *ExceptionList;
```

<div dir="rtl" markdown="1">

الـ `EXCEPTION_REGISTRATION` كذلك عبارة عن `Structure` وتعريفه كالآتي :

</div>


```c
struct _EXCEPTION_REGISTRATION {
   DWORD prev;
   DWORD handler;
};
```

<div dir="rtl" markdown="1">


نلاحظ أن الـ `EXCEPTION_REGISTRATION` يتكوّن من عنصرين  ( كل عنصر سِعته `4` بايت ) : 

**الأول** : `prev` ، عبارة عن `pointer` ويُشير الى العنصر السابق في الـ `Linked List`

</div>

> A pointer to the next `EXCEPTION_REGISTRATION_RECORD` in the chain

 <div dir="rtl" markdown="1">

**الثاني** : `handler` . عبارة عن `pointer` ويُشير الى الدالة المسؤولة عن التعامل مع الـ `Exception`

الصورة التالية من كتاب [Practical Malware Analysis](https://a.co/d/aeXQFn7) لعلّها توضّح الصورة وتختصر الكثير من الأفكار 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/4.png)

واحده من التفاصيل المهمة التي تم ذكرها في كتاب  [Practical Malware Analysis](https://a.co/d/aeXQFn7) المعلومة الآتية :

</div>

> This linked list operates conceptually as a stack. The first record to be called is the last record to be added to the list. The SEH chain grows and shrinks as layers of exception handlers in a program change due to subroutine calls and nested exception handler blocks. For this reason, SEH records are always built on the stack.


<div dir="rtl" markdown="1">

نعرف من النص السابق أن الـ `SEH Chain` يتم تخزينها في الـ `Stack` 

كذلك آخر عنصر يُضاف للـ `SEH Chain` هو أول عنصر سيقوم نظام التشغيل بالمرور عليه في حال حصول أي `Exception` 

الصورة التالية تُعطينا لمحة عن كيف يبدو شكل الـ `SEH Chain` في الـ `Stack` 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/SEH/5.png)


نكتفي بهذا القدر من المعلومات، ولعلنا نُكمل في مقالات قادمة -إن شاء الله- 

</div>



> ℹ️ [ملاحظة]
> هذه المقالة تمّت كتابتها خلال دراسة هذه المواضيع، فكل ما تم ذكره هنا قد يحتمل الخطأ، لكن بالإمكان العودة إلى المراجع التي إستندت عليها هذه المقالة


