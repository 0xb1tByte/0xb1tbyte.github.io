---
layout: post
published: true
title: Detection Engineering and Detection as Code
date: '2025-08-02'
tags:
- Arabic-Article
- Blue Team
- Detection Engineering
- Detection as Code
---

<div dir="rtl" markdown="1">


السلام عليكم ورحمة الله وبركاته 

في هذه المقالة سأشارك تلخيصًا لقراءات متفرّقة حول `Detection Engineering` و `Detection as Code (DaC)` ، مستندة إلى كتابين رئيسيين:

- [Practical Threat Detection Engineering](https://amzn.eu/d/dEUdSvt) 
- [Automating Security Detection Engineering](https://amzn.eu/d/beqUiqx)

الهدف هو تبسيط وفهم كيفية بناء قواعد الكشف (`Detection Rules`) وإدارتها بشكل فعال ضمن فرق الأمن السيبراني.

بسم الله لنبدأ 
# Detection Engineering

في مقال [About Detection Engineering – cyb3rops](https://cyb3rops.medium.com/about-detection-engineering-44d39e0755f0)، يُعرَّف `Detection Engineering` ببساطة على أنه:
تحويل المعلومات المتعلقة بالتهديدات (`Threat Intelligence`) إلى `Detections` قابلة للتنفيذ، بحيث تصبح بيانات التهديدات شيئًا عمليًا يمكن للنظام الأمني استخدامه للكشف عن نشاطات مشبوهة أو ضارة.


</div>

> Detection engineering transforms information about threats into detections.

<div dir="rtl" markdown="1">

بينما كتاب [Practical Threat Detection Engineering](https://amzn.eu/d/dEUdSvt)  يقدم منظورًا أكثر شمولية، فيصف `Detection Engineering` بأنها مجموعة عمليات (`Processes`) تبدأ من جمع متطلبات الكشف (`Detection Requirements`)، مرورًا بتجميع الـ`Telemetry` من مصادر البيانات (`Logs, EDR Data, Network Traffic`)، ثم تنفيذ وصيانة الـ`Detection Logic`، وانتهاءً بالتحقق المستمر من فعالية قواعد الكشف (`Detection Rules Effectiveness`).

</div>

> Detection engineering definition
> Detection engineering can be defined as a set of processes that enable potential threats to be detected within an environment. These processes encompass the end-to-end
> life cycle, from collecting detection requirements, aggregating system telemetry, and implementing and maintaining detection logic to validating program effectiveness.

<div dir="rtl" markdown="1">
  
الدمج بين التعريفين يوضح أن الـ ``Detection Engineering`` ليست مجرد خطوة واحدة، بل عملية تبدأ من **فهم التهديد وتحويله إلى قاعدة كشف (`Detection Rule`)**، ثم تطويرها وصيانتها بشكل مستمر لتبقى فعالة ضد التطور المستمر لأساليب المهاجمين (TTPs).

</div>
