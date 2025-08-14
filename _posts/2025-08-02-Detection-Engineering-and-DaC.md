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

## Implementation & Validation
في هذه المرحلة يتم نقل الـ `Detection Rule` من بيئة التطوير إلى بيئة الإنتاج، بعد التأكد من نجاحها في الاختبارات السابقة. التركيز يكون على تفعيل القواعد، مع مراعاة عدم التأثير على استقرار الأنظمة. كذلك تتضمن العملية إعداد الـ `configurations` النهائية وربطها بالـ `alerting` أو الـ `ticketing systems`، وتطبيق أي `tuning` أو `thresholds` ضرورية.

يتبع ذلك الـ `validation`، والذي يتضمن مراقبة أداء الـ `Detection Rules` والـ `detection systems` في الإنتاج، مع إجراء اختبارات تقييم روتينية (`Quarterly` أو `Semi-Annual`). هذه المراجعات الدورية تساعد على تحديد ما إذا كانت القواعد ما تزال فعّالة، وتضمن توافقها مع الأهداف لبرنامج الـ `Detection Engineering`.


## Deprecation
تأتي هذه المرحلة عندما تصبح الـ `Detection Rules` غير فعّالة، أو لم تعد ذات صلة بالـ `TTP relevance` المستهدفة، أو عندما تسبب معدل عالي من الضجيج مقارنة بالفائدة، أو بسبب تغييرات في الـ `log sources` أو الـ `data inputs`. في هذه المرحلة، يتم سحب أو تعطيل القواعد التي لم تعد تخدم الغرض الأمني المطلوب، سواء بسبب تغيّر طبيعة التهديدات، أو لوجود قاعدة أحدث تحل محلها. يجب أن تتضمن العملية تسجيل سبب الإلغاء، والتأكد من عدم وجود `dependencies` على القاعدة الملغاة في عمليات أو أنظمة أخرى. وكما في أي `life cycle`، يجب أن تتضمن هذه المرحلة جمع `feedback` من فريق الـ `engineering` أو الـ `stakeholders` لتحديد الحالات التي تستدعي الـ `deprecation`.


# `Detection-as-Code (DaC)`

في أغلب فرق الأمن السيبراني، عملية إنشاء الـ `Detection Rules` تتم بشكل يدوي بالكامل. يبدأ الأمر بكتابة الـ `logic` مباشرة في الـ `SIEM` أو الـ `console`، ثم تعديل القاعدة بشكل يدوي عند الحاجة، واختبارها بشكل يدوي كذلك. كل المراحل التي تمر بها الـ `Detection Engineering` — من التصميم، التطوير، النشر، وحتى الصيانة — تتم بدون أتمتة أو توثيق منظم.

هذا الأسلوب اليدوي يخلق عدة مشاكل:
- صعوبة تتبع التغييرات أو العودة لإصدار سابق من القاعدة عند حدوث مشكلة.
- صعوبة التوسع مع تزايد عدد الـ detections أو تغير التهديدات بسرعة.
- ارتفاع احتمالية الأخطاء بسبب غياب المراجعة الآلية.
- اكتشاف المشاكل متأخرًا أثناء الحوادث الفعلية.

هنا يأتي دور `Detection-as-Code (DaC)`، وهو نهج يطبق مبادئ الـ `software engineering` على عملية تطوير وصيانة الـ `detections`. 

بدلاً من العمل المباشر في الـ `SIEM`، يتم كتابة الـ `rules` كـ `code` في بيئة تطوير منظمة، مع استخدام أدوات مثل `Git` للـ `version control`، وإجراء `code reviews`، وإضافة `automated testing` لضمان جودة القواعد قبل نشرها.  كذلك، تتم عملية نشر القواعد على الأنظمة مثل `SIEM` أو `EDR` بشكل مؤتمت، مما يتيح إمكانية نشر القاعدة على أكثر من منصة بخطوة واحدة (عادةً عن طريق سكربتات)، ويجعل التوسع أسهل بكثير. عمليات التحقق والاختبار تتم أيضًا بشكل مؤتمت، وبالإمكان إجراء اختبارات محاكاة (`Adversary Simulation`) لقياس كفاءة القواعد والتأكد من فعاليتها قبل اعتمادها في بيئة الإنتاج

هذا النهج يضمن أن عملية تطوير الـ `detections` قابلة للتوسع بشكل مستمر، ويمنح فرق الأمن ثقة أكبر بأن الـ `Detection Rules` ستؤدي وظيفتها بكفاءة، حتى مع تغير بيئة العمل أو ظهور أساليب هجومية جديدة.

## `Detection-as-Code (DaC)` Requirements

لتطبيق منهج الــ `Detection-as-Code (DaC)`، يجب توفير بعض المكونات التقنية الأساسية:

 * **Version Control Systems (مع CI Runners)**:
`Git` يُستخدم لتتبع جميع التعديلات على الـ `detection rules` وإدارة نسخ متعددة لكل قاعدة، بما في ذلك نسخ للاختبار والإنتاج. هذا يسهل الرجوع لأي نسخة سابقة عند الحاجة.

كذلك كل قاعدة جديدة تخضع أولًا لاختبارات `unit tests` مثل التحقق من الـ `syntax` ، ثم لاختبارات `integration tests` لتقييم عملها ضمن بيئة قريبة من الإنتاج.

يتم أتمتة كل هذه الاختبارات، بالإضافة إلى أتمتة عملية نشر القواعد على أكثر من `SIEM` أو `EDR` دفعة واحدة ويمكن أيضًا إجراء `adversary simulation` لاختبار فعالية القاعدة بشكل مؤتمت قبل نشرها. وتتم عادةً أتمتة هذه العمليات باستخدام `CI runners`

* **API Support:**

لتسهيل النشر المؤتمت للـ `rules` وربطها بأنظمة أخرى، مثل الـ `alerting` و`ticketing`. مثال: سكربت `API` ينشر قاعدة جديدة على أكثر من `SIEM` في نفس الوقت.

* **Use Case Syntax**:
  
تحديد الصياغة الصحيحة للـ `rules` يضمن قراءة واضحة وصيانة سهلة. مثال: قواعد مكتوبة بلغة `KQL` أو `Sigma` لتوحيد الأسلوب عبر الفريق.

* **Testing Instrumentation**:
  
أدوات الاختبار تمكن الفريق من التأكد من أن كل قاعدة تعمل بشكل صحيح، وتشمل اختبارات `unit` و`integration` وأحيانًا `adversary simulation` لمحاكاة هجمات واقعية.

* **Secrets Management:**
  
إدارة آمنة للمفاتيح أو بيانات الاتصال الحساسة، لضمان عدم تضمينها مباشرة في الكود والحفاظ على أمان البيئات.

</div>
