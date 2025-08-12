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

> Detection engineering definition:
> Detection engineering can be defined as a set of processes that enable potential threats to be detected within an environment. These processes encompass the end-to-end
> life cycle, from collecting detection requirements, aggregating system telemetry, and implementing and maintaining detection logic to validating program effectiveness.

<div dir="rtl" markdown="1">
  
الدمج بين التعريفين يوضح أن الـ ``Detection Engineering`` ليست مجرد خطوة واحدة، بل عملية تبدأ من **فهم التهديد وتحويله إلى قاعدة كشف (`Detection Rule`)**، ثم تطويرها وصيانتها بشكل مستمر لتبقى فعالة ضد التطور المستمر لأساليب المهاجمين (TTPs).

# Detection Life Cycle

تُعتبر الـ `Detection Life Cycle` إطارًا منهجيًا يصف المراحل التي تمر بها أي `Detection Rule` أو `Use Case` من الفكرة وحتى إيقاف الـ `Detection Rule` أو تعديلها.

بحسب كتابي [Practical Threat Detection Engineering](https://amzn.eu/d/dEUdSvt)  و [Automating Security Detection Engineering](https://amzn.eu/d/beqUiqx) ، يمكن تلخيص هذه الدورة في المراحل الآتية :

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/DaC/1.png)
* [Automating Security Detection Engineering](https://amzn.eu/d/beqUiqx)

![2](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/DaC/2.png)
* [Practical Threat Detection Engineering](https://amzn.eu/d/dEUdSvt)


## Establish Requirements / Discovery
في هذه المرحلة الأولية، تبدأ فرق الـ `Detection Engineering` بجمع متطلبات الكشف (`Detection Requirements`).

هنا يتم تحديد:

- ما هو الـ `Scope` للبرنامج؟ على سبيل المثال، بناء `Detection Rules` موجّهه لأنظمة لينكس 
- ماهي الـ `Detection Rules` التي تعتبر مفيدة ويجب تطويرها ؟
- هل لدينا بيانات وأدوات داخل البيئة تمكننا من بناء هذه الـ `Detection Rules` ؟

إضافةً لذلك، يتم وضع أولويات التنفيذ (`Implementation Priorities`)، بمعنى ماهي الـ `Detection Rules` التي يجب تنفيذها أولًا  

## Development 
بعد تحديد المتطلبات، تبدأ مرحلة تصميم وتطوير الـ `Detection Rules`

يتم هنا بناء الـ `Detection Logic` الخاص بالـ `Detection Rules` ، وبناء هذا الـ `logic` يعتمد بشكل كبير على مصادر البيانات المتاحة (`Logs, EDR Data, Network Traffic`)

## Testing
بعد بناء الـ `Detection Logic`، تأتي مرحلة الاختبارات:
هناك نوعان أساسيان من الاختبارات في هذا المجال: `Unit Tests` و `Integration Tests`.

الـ `Unit Tests` يهدف إلى التحقق من صحة الـ `syntax` الخاص بالـ `Detection Rule` لضمان عدم وجود أخطاء تمنع تنفيذه. بعد ذلك، يتم تشغيل القاعدة على `log samples` معروفة، حيث يتم تمرير عينات من الـ `logs` التي يُفترض أن تقوم القاعدة بعمل `trigger` لها، وأخرى سليمة للتأكد من عدم إطلاق تنبيه خاطئ (تقليل الـ `false positives`). هذه الطريقة تساعد على اكتشاف الأخطاء في المنطق أو في الصياغة مبكرًا وعادةً ما تُستخدم سكربتات وأدوات لأتمتة هذه الاختبارات بحيث تُنفذ تلقائيًا عند إضافة أو تعديل أي قاعدة.

أما الـ `Integration Tests` فيتضمن اختبار الـ `Detection` في بيئة `Staging` أو بيئة اختبار للـ `SIEM/EDR` تشبه بيئة الإنتاج الحقيقية، للتأكد من أن القاعدة تعمل على المنصة المستهدفة بدون أخطاء، وقياس تأثيرها على الأداء.

</div>
