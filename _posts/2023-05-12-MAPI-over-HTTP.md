---
layout: post
published: true
title: MAPI over HTTP
date: '2023-05-12'
tags:
  - Exchange
  - Network
  - Arabic-Article
---

  <div dir="rtl" markdown="1">

السلام عليكم ورحمة الله وبركاته

المقالة هذه تشرح `MAPI over HTTP` ، 

على الأغلب سيصادفك هذا الـ `API` في حال تعاملت مع الـ `Exchange Server`

المقالة بإذن الله ستكون تلخيص لبعض النقاط المذكورة في [الورقة](https://interoperability.blob.core.windows.net/files/MS-OXCMAPIHTTP/%5bMS-OXCMAPIHTTP%5d-210422.pdf) الخاصة بالـ `MAPI over HTTP` ، الورقة نُشرت من قبل فريق مايكروسوفت  

أهدف لشرح النقاط الآتية :
 - مقدمة ، ماهو `MAPI` ؟
 - ماهو `MAPI over HTTP` وكيف يعمل ؟
 
 بسم الله ، لنبدأ 
 
# مقدمة ، ماهو `MAPI` ؟

بدايةً، `MAPI` اختصار لـ `Microsoft Messaging API (MAPI)`
 
في كتاب **Microsoft Windows Architecture Training (Training Kit)** عرّفت مايكروسوفت `MAPI` بالنقاط الآتية : 
 
 
  </div>
  
  > The Windows operating system includes the Messaging Application Programming Interface (MAPI) that allows information from mail, fax, documents, and other systems to be accessible to users across an entire organization. 

 > With the MAPI subsystem, you can easily add messaging features to any Windows-based application.
 

<div dir="rtl" markdown="1">
فـ `MAPI` هو `API` قامت بتطويره مايكروسوفت ضمن نظام التشغيل الخاص بها، 

ماذا يقدّم هذا الـ `API` ؟ 

يتيح هذا الـ `API` للبرامج التي تعمل ضمن نظام الويندوز أن تكون `messaging-aware` أي أن البرنامج يستطيع التعامل أو التواصل مع `Mail Server` لجلب أو معالجة البيانات

ولا نقصد هنا فقط أن يكون البرنامج `mail Client` مثل الـ `Outlook` ، الـ `Outlook` هو أحد الأمثلة الشائعة لاستخدام هذا الـ `API` ، لكن مايكروسوفت طوّرت هذا الـ `API` ليتيح لأي تطبيق يعمل ضمن بيئتها من الوصول الى الـ `Mail Server` أو كما أسمتها مايكروسوفت أيضًا بالـ`Messaging System`

وعلى ذكر الـ `Messaging System` ، في بيئة ويندوز يمثلها الـ `Exchange Server`

عرفنا الآن أن `MAPI` ماهو إلا نظام طوّرته مايكروسوفت لتتيح لتطبيقاتها أن تكون `messaging-aware` وتستطيع التواصل مع الـ `Messaging System` 

فما هو الـ `MAPI over HTTP` ؟ 

# ماهو `MAPI over HTTP` وكيف يعمل ؟

نعود الآن للورقة المعنيّة في هذه المقالة، نجد في الورقة التعريف الآتي 
 
 </div>
 
 > The Messaging Application Programming Interface (MAPI) Extensions for HTTP enable a client to access personal messaging data and directory data on a server by sending HTTP requests and receiving responses returned on the same HTTP connection.

<div dir="rtl" markdown="1">

إذًا `MAPI over HTTP` ماهو إلا إضافة أخرى طورتها مايكروسوفت حتى تتيح للتطبيقات إستخدام هذا الـ `API` لكن ضمن الـ `HTTP Protocol`

لعل الصورة التالية توضح الفكرة أكثر 


</div>

 ![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/MAPI-over-HTTP/1.png#center)
 
 
 <div dir="rtl" markdown="1">
عرفنا سابقًا أن في الـ `MAPI system` لدينا طرفان ، الـ `Messaging System` ونقصد به الـ `Exchange Server` والـ `Messaging Client` ولنأخذ الـ `Outlook` كمثال ولدينا أيضًا الـ `API` نفسه 

فكيف ستبدو الصورة في حال انتقلنا الى الـ `HTTP Protocol` ؟

لو أكملنا القراءة في الورقة سنجد المعلومة الآتية 


 </div>
 
 >  When the client initiates communication with the server, the server creates a virtual Session Context and returns a session context cookie to the client. Once established, the Session Context allows the client to perform operations on the server

 <div dir="rtl" markdown="1">
نعرف من العبارة أعلاه أن في الـ `MAPI over HTTP` تُدار الـ `Session` عبر الـ `Cookie`
 
وبمجرّد أن يُتم الـ `Client`  الـ `Authentication` مع الـ `Server` سيتمكن من إجراء العمليات التي يريدها .. فما هي هذه العمليات ؟ 

الصورة التالية تلخّص كيف يتواصل الـ `Client` مع الـ `Server` في الـ `MAPI over HTTP` 


 </div>
 
 ![2](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/MAPI-over-HTTP/2.png#center)
 
  <div dir="rtl" markdown="1">
  في البداية يجب أن نعرف أن جميع الـ `Requests` التي يرسلها الـ `Client` هي عبارة عن `POST Request` كما تم ذكره في الورقة
  
  </div>
  
  
  > All requests MUST use the POST method. POST supports uploading a request block and returning a response block.
  
  
  <div dir="rtl" markdown="1">
  
  وهذه الـ `POST Requests` تتبع الصيغة الآتية
  </div>
  
```http
    POST <Autodiscover-provided endpoint> HTTP/1.1
    Host: <URL of the host server>
    Content-Length: <length of REQUEST BODY>
    Content-Type: application/mapi-http
    X-RequestType: <?>
    X-RequestId: <id>
    X-ClientApplication:<client version>

    <REQUEST BODY>
```
  
  <div dir="rtl" markdown="1">
  
و يتواصل الـ `Client`  مع الـ `Server` كالآتي : 

1 - يبدأ الـ `Client` في البداية بإرسال `Connect Request`

هذا مثال على الـ `Connect Request`
  
  </div>


```http
    POST domain.com/mapi/emsmdb/?MailboxId=8f0a9c7c-fa0f-4b6f-bd9e-ea1c9d3b7e8c@domain.com HTTP/1.1
    Content-Type: application/mapi-http
    X-RequestType: Connect
    X-ClientInfo: {6F29FC82-34A0-40A1-A5A8-760E9D6BD9EB}:14.0.4760.1000
    X-ClientApplication: Outlook/14.0.4760.1000
    X-RequestId: {C715155F-2BE8-44E0-BD34-2960067874C8}:2
    X-RequestVersion: 2.0
    Content-Length: 32

    00000000000000000000000000000000
```

  <div dir="rtl" markdown="1">
2 - يقوم الـ `Client` بإرسال `EXECUTE Request`

يبدو هذا الـ `Request` كالآتي : 
 </div>
 
 
 ```http
    POST https://domain.com/mapi/emsmdb/?MailboxId=8f0a9c7c-fa0f-4b6f-bd9e-ea1c9d3b7e8c@domain.com HTTP/1.1
    Content-Type: application/mapi-http
    X-RequestType: Execute
    X-ClientInfo: {6F29FC82-34A0-40A1-A5A8-760E9D6BD9EB}:14.0.4760.1000
    X-ClientApplication: Outlook/14.0.4760.1000
    X-RequestId: {C715155F-2BE8-44E0-BD34-2960067874C8}:3
    X-RequestVersion: 2.0
    Content-Length: 32

    00000000000000000000000000000000
 ```
 
 
<div dir="rtl" markdown="1">
3 - وآخرًا سيقوم الـ `Client` بإرسال `Disconnect Request`  لإنهاء الإتصال مع الـ `Server`

سيكون الـ `Request` كالآتي : 
</div>


```http
    POST https://domain.com/mapi/emsmdb/?MailboxId=8f0a9c7c-fa0f-4b6f-bd9e-ea1c9d3b7e8c@domain.com HTTP/1.1
    Content-Type: application/mapi-http
    X-RequestType: Disconnect
    X-ClientInfo: {6F29FC82-34A0-40A1-A5A8-760E9D6BD9EB}:14.0.4760.1000
    X-ClientApplication: Outlook/14.0.4760.1000
    X-RequestId: {C715155F-2BE8-44E0-BD34-2960067874C8}:4
    X-RequestVersion: 2.0
    Content-Length: 32

    00000000000000000000000000000000
```

<div dir="rtl" markdown="1">
لعلك لاحظت أنه نستطيع تحديد نوع الـ `Request` بالنظر الى الـ `X-RequestType Header`

وسنجد في الـ `X-ClientApplication Header` معلومات عن الـ `Messaging Client`

نكتفي بالتلخيص الى هنا و لعلنا نكمل في مقالة أخرى إن شاء الله 


ختامًا، أود الإشارة بأن هذه المقالة تمّت كتابتها خلال دراسة هذه المواضيع، فكل ما تم ذكره هنا قد يحتمل الخطأ، لكن بالإمكان العودة إلى المراجع التي إستندت عليها هذه المقالة

وإن أصبت فمن توفيق الله وحده، وإن أخطأت فمن نفسي والشيطان

# المراجع
- [Messaging Application Programming Interface (MAPI) Extensions for HTTP](https://interoperability.blob.core.windows.net/files/MS-OXCMAPIHTTP/%5bMS-OXCMAPIHTTP%5d-210422.pdf)
- [Microsoft Windows Architecture for Developers Training Kit](https://flylib.com/books/en/2.914.1/)

</div>
