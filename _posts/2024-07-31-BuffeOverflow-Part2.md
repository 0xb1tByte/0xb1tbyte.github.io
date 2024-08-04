---
layout: post
published: true
title: Buffer Overflow Part 2
date: '2024-07-31'
tags:
- Arabic-Article
- System Security
- Exploit Development
---

<div dir="rtl" markdown="1">

السلام عليكم ورحمة الله وبركاته 

نستكمل في هذه المقالة الحديث عن الـ `Buffer`

عرضنا في المقالة السابقة مثال على **القراءة** خارج نطاق الـ `Buffer` 

سنستعرض في هذه المقالة -إن شاء الله- مثال آخر حول عملية **الكتابة** خارج نطاق الـ `Buffer` 

بسم الله لنبدأ 

# مثال : الكتابة خارج نطاق الـ `Buffer`

المثال من نفس كتابنا السابق، كتاب [The Shellcoder's Handbook](https://amzn.eu/d/e5ihS4i) 

لدينا الكود الآتي 
</div>

```c
1:  int main () {
2:      int array[5];
3:      int i;
4:      for (i = 0; i <= 255; i++ )
5:      {
6:         array[i] = 10;
7:      }
8:  }
```

<div dir="rtl" markdown="1">
`2` : قمنا بتعريف متغير من نوع `array` ، ويفترض أن تكون سعة هذه الـ `array` هو `5` عناصر

`3` : عرّفنا متغير آخر ، وهو العدّاد ( counter ) الذي سنستخدمه في الـ `for loop`

`4` - `7` : في داخل الـ `for loop` قمنا بتخزين القيمة `10` في عناصر الـ `array`  ، لكن المشكلة في هذا الكود أننا خرجنا خارج نطاق الـ `array` 

لاحظ أن الـ `for loop` ستستمر بتنفيذ العملية `array[i] = 10;` حتى ينتفي الشرط `i <= 255` ، وكما نعرف سابقًا ، الـ `array` لا تسع إلّا `5` عناصر 

فالكود أعلاه بكل بساطة يقوم بالكتابة خارج نطاق الـ `array` ( الـ `buffer` ) 

طيّب، لنقم الآن بعمل `Compile` للكود ونرى ما سيحدث 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/BufferOverflow//3.png)

مثل المرّة السابقة، نلاحظ أن الـ `Compiler` لم يقم بطباعة أي خطأ ، لننتقل الآن لتنفيذ البرنامج 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/BufferOverflow//4.png)

في هذا المثال نلاحظ أننا حصلنا على رسالة الخطأ `Segmentation fault` ( في مثال **القراءة** خارج نطاق الـ `Buffer` من المقالة السابقة لم نواجه مشكلة في تنفيذ البرنامج ) 

وهي رسالة خطأ عامة تحتمل أسباب كثيرة ، أحد الأسباب الشائعة لهذه الرسالة هو عندما يحاول البرنامج الوصول لعنوان في الذاكرة **غير مسموح** الوصول له 

أو عندما يحاول البرنامج الوصول لعنوان ذاكرة خاطئ في الأساس ، وأحد أسباب هذه الرسالة كذلك عندما يقوم البرنامج بالإشارة ( Dereferencing ) لـ `pointer` يحمل قيمة `NULL` 

طيّب لنعود لبرنامجنا ونحاول البحث أكثر حول سبب هذا الخطأ ، سنستخدم [gdb](https://www.sourceware.org/gdb/) لهذه المُهمة

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/BufferOverflow//5.png)




</div>
