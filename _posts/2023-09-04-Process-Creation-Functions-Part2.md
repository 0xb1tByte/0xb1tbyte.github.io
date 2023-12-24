---
layout: post
published: true
title: Process Creation Functions Part 2
date: '2023-09-04'
tags:
- Red-Teaming
- Blue Team
- Malware Development
- Arabic-Article
- Windows
- Windows Internals
---

<div dir="rtl" markdown="1">

# مقدمة 
السلام عليكم ورحمة الله وبركاته 

المقالة هذه استكمال [للسلسلة السابقة](https://0xb1tbyte.github.io/2023-08-04-Process-Creation-Functions-Part1/)

في هذا الجزء سنُلقي نظرة تفصيلية على الكود الخاص بدالة `CreateProcess` ، كيفية نداءها ، وماهي المتغيرات التي نحتاج لتعريفها ومن ثم تمريرها للدالة 

كما ذكرنا في المقالة السابقة، سنقوم باستخدام دالة `CreateProcess` لانشاء عملية `Notepad.exe`

وهذا هو الكود الذي سنقوم بمناقشته 

</div>

```c
    1 // CreateProcess.cpp : This file contains the 'main' function. Program execution begins and ends there.
    2 //
    3 
    4 #include <iostream>
    5 #include <windows.h>
    6 #include <stdio.h>
    7 int main()
    8 {
    9     // Declare variables
    10     STARTUPINFO si;
    11     PROCESS_INFORMATION pi;
    12     BOOL result;
    13     DWORD exitCode;
    14 
    15     // Initialize variables
    16     ZeroMemory(&si, sizeof(si));
    17     si.cb = sizeof(si);
    18     ZeroMemory(&pi, sizeof(pi));
    19 
    20     // Create a process using the command line argument
    21     result = CreateProcess(
    22         L"C:\\Windows\\System32\\notepad.exe", // Module name
    23         NULL, // Command line
    24         NULL, // Process handle not inheritable
    25         NULL, // Thread handle not inheritable
    26         FALSE, // Set handle inheritance to FALSE
    27         0, // No creation flags
    28         NULL, // Use parent's environment block
    29         NULL, // Use parent's starting directory
    30         &si, // Pointer to STARTUPINFO structure
    31         &pi); // Pointer to PROCESS_INFORMATION structure
    32 
    33     if (!result)
    34     {
    35         printf("CreateProcess failed (%d).\n", GetLastError());
    36         return -1;
    37     }
    38 
    39     // Wait until child process exits
    40     WaitForSingleObject(pi.hProcess, INFINITE);
    41 
    42     // Get the exit code of the child process
    43     GetExitCodeProcess(pi.hProcess, &exitCode);
    44     printf("Child process exited with code %d.\n", exitCode);
    45 
    46     // Close process and thread handles
    47     CloseHandle(pi.hProcess);
    48     CloseHandle(pi.hThread);
    49 
    50     return 0;
    51 }
```

<div dir="rtl" markdown="1">
قبل الخوض في تفاصيل الكود، من الجيد أن نعرف الـ Signature / Prototype الخاص بالدالة حتى يسهل علينا تتبّع الكود 

# CreateProcess Prototype

بحسب موقع [مايكروسوفت](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa) فهذا الـ Syntax الخاص بالدالة 

</div> 

```c
    BOOL CreateProcessA(
      [in, optional]      LPCSTR                lpApplicationName,
      [in, out, optional] LPSTR                 lpCommandLine,
      [in, optional]      LPSECURITY_ATTRIBUTES lpProcessAttributes,
      [in, optional]      LPSECURITY_ATTRIBUTES lpThreadAttributes,
      [in]                BOOL                  bInheritHandles,
      [in]                DWORD                 dwCreationFlags,
      [in, optional]      LPVOID                lpEnvironment,
      [in, optional]      LPCSTR                lpCurrentDirectory,
      [in]                LPSTARTUPINFOA        lpStartupInfo,
      [out]               LPPROCESS_INFORMATION lpProcessInformation
    );
```

<div dir="rtl" markdown="1">
<br>
لنبدأ التفصيل في هذا الـSyntax : 
- نلاحظ أن الدالة تُرجع قيمة من نوع `BOOL` ، ستكون القيمة `TRUE` في حال تم تنفيذ الدالة بشكل سليم ، و `FALSE` في حال فشلت الدالة في بدء العملية
- بالنسبة للـ parameters الخاصة بالدالة نلاحظ وجود بعض العبارات مثل `in` و `out` و `optional`
    - تستخدم هذه العبارات بين المطورين كوسيلة لشرح الدالة وماتفعله بالـ parameters
    - `in` : اختصار لـ `input` والمقصود بها أن الـ parameter هذا سيتم استخدامه داخل الدالة
    - `out` : اختصار لـ `output` والمقصود منه أن الدالة ستستقبل هذا المتغيّر وستقوم بارجاعه او التعديل على قيمته في حالة الـ pointers
    - `optional` : كما ترمز الكلمة، تمرير هذا الـ parameter للدالة ليس ضروري 
- ننتقل الآن للـ parameters
    - `lpApplicationName` : يشير الى البرنامج الذي سيتم تنفيذه
    - `lpCommandLine` : بدل من تمرير اسم البرنامج في الـ parameter السابق، بامكاننا تمرير `commands` يتم تنفيذها في العملية، على سبيل المثال لو أردنا فتح ملف `test.txt` عبر الـ `Notepad.exe` نمرر القيمة `notepad.exe test.txt` لهذا الـ parameter
    - `lpProcessAttributes` : عبارة عن pointer يشير لـ structure يسمى `SECURITY_ATTRIBUTES` ، للمزيد اقرأ عن هذا الـ [structure](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/legacy/aa379560(v=vs.85)) 
    - `lpThreadAttributes` : يمثل pointer لنفس الـ structure السابق
    - `bInheritHandles` : يمثل قيمة `Boolean` وتعبر عن هل بالامكان وراثة الـ `handles` الخاصة بالـ `calling process`
    - `dwCreationFlags` : يستخدم هذا الـ parameter للتحكم في مرحلة انشاء العملية، بالامكان الاطلاع على جميع الـ flags [من هنا](https://learn.microsoft.com/en-us/windows/win32/procthread/process-creation-flags)
    - `lpEnvironment` : عبارة عن pointer للـ `environment variables` الخاصة بالعملية
    - `lpCurrentDirectory` : يمثل المسار الكامل الخاص بالعملية
    - `lpStartupInfo` : عبارة عن pointer يشير لـ structure يسمى `STARTUPINFO` ، يمثل هذا الـ structure الاعدادات الخاصة بالعملية عند بدءها مثل حجم وشكل الـ window , العنوان وخلافه ، بالامكان القراءة عن هذا الـ structure [هنا](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/ns-processthreadsapi-startupinfoa)
    - `lpProcessInformation` : عبارة عن pointer يشير الى structure يسمى `PROCESS_INFORMATION` هذا الـ structure يحمل العديد من المعلومات عن العملية التي تم انشاءها مثل : الـ handle الخاص بالعملية ، الـ thread الأساسي في العملية وغيره ، بالامكان القراءة عن هذا الـ structure [هنا](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/ns-processthreadsapi-process_information) 

نعود الآن للكود 
# انشاء عملية `notepad.exe` باستخدام دالة `CreateProcess`
</div> 

```c
    1 // CreateProcess.cpp : This file contains the 'main' function. Program execution begins and ends there.
    2 //
    3 
    4 #include <iostream>
    5 #include <windows.h>
    6 #include <stdio.h>
    7 int main()
    8 {
    9     // Declare variables
    10     STARTUPINFO si;
    11     PROCESS_INFORMATION pi;
    12     BOOL result;
    13     DWORD exitCode;
    14 
    15     // Initialize variables
    16     ZeroMemory(&si, sizeof(si));
    17     si.cb = sizeof(si);
    18     ZeroMemory(&pi, sizeof(pi));
    19 
    20     // Create a process using the command line argument
    21     result = CreateProcess(
    22         L"C:\\Windows\\System32\\notepad.exe", // Module name
    23         NULL, // Command line
    24         NULL, // Process handle not inheritable
    25         NULL, // Thread handle not inheritable
    26         FALSE, // Set handle inheritance to FALSE
    27         0, // No creation flags
    28         NULL, // Use parent's environment block
    29         NULL, // Use parent's starting directory
    30         &si, // Pointer to STARTUPINFO structure
    31         &pi); // Pointer to PROCESS_INFORMATION structure
    32 
    33     if (!result)
    34     {
    35         printf("CreateProcess failed (%d).\n", GetLastError());
    36         return -1;
    37     }
    38 
    39     // Wait until child process exits
    40     WaitForSingleObject(pi.hProcess, INFINITE);
    41 
    42     // Get the exit code of the child process
    43     GetExitCodeProcess(pi.hProcess, &exitCode);
    44     printf("Child process exited with code %d.\n", exitCode);
    45 
    46     // Close process and thread handles
    47     CloseHandle(pi.hProcess);
    48     CloseHandle(pi.hThread);
    49 
    50     return 0;
    51 }
```

<div dir="rtl" markdown="1">
<br>
لنبدأ بالتفصيل : 
- `10` : نلاحظ أننا قمنا بتعريف متغيّر باسم `si` من نوع `STARTUPINFO` وهو يمثّل الـ structure الذي قمنا بشرحه سابقًا، إن كنت تتساءل كيف استطعنا الوصول لهذا النوع ( `STARTUPINFO` ) فلأنه قمنا بادراج المكتبة `windows.h` في السطر رقم `5`
- `11` : قمنا بتعريف متغيّر باسم `pi` والذي يشير كذلك للـ structure الذي سيحمل المعلومات الخاصة بالعملية التي سيتم انشاءها
- `12` : قمنا بتعريف المتغيّر `result` الذي سيحمل نتيجة تنفيذ الدالة 
- `13` : الغرض من المتغيّر `exitCode` هو لمعرفة الـ exit code  الخاص بعملية الـ `notepad.exe` بعد انتهاءها 
- `16` : قمنا بعمل initialization للمتغيّر `si` عبر الدالة `ZeroMemory` ، الخطوة هذه لتهيئة الذاكرة وحذف أي قيم سابقة في الذاكرة قبل استخدامها 
- `17` : قمنا بتعريف قيمة المتغيّر `si.cb` والذي يمثّل الـ size الخاص بالـ `STARTUPINFO` structure
- `18` : قمنا بتهيئة الذاكرة للمتغيّر `pi`
- `21` : قمنا بنداء دالة `CreateProcess` ونلاحظ أن القيمة العائدة من الدالة سيتم تخزينها في المتغيّر `result`
- `22` : قمنا بتمرير القيمة `C:\\Windows\\System32\\notepad.exe` والتي تمثل اسم البرنامج الذي سيتم تنفيذه ( `lpApplicationName` )
- `23` : قمنا بتمرير القيمة `NULL` كوننا عرفنا اسم البرنامج في الـ parameter السابق 
- `24` : يمثل الـ `lpProcessAttributes` وقمنا بتمرير القيمة `NULL` مما يعني أنه سيتم استخدام الاعدادات الافراضية للـ `Security Descriptor` الخاص بعملية الـ `notepad.exe` 
- `25` : نفس السطر السابق لكن على مستوى الـ Thread
- `26` : قمنا بتمرير القيمة `FALSE` والتي تعني هنا أنه لن يتم توريث الـ handles الخاصة بالـ `calling process` الى الـ `notepad.exe` process
- `27` : لم نقم بتحديد أي  `Creation Flags` ، بالتالي سيتم تنفيذ الـ Thread الخاص بالعملية مباشرة
- `28` : لم نقم بتحديد قيمة للـ `environment block` ، بالتالي سيتم استخدام الـ `environment block` الخاص بالـ `calling process`
- `29` : لم نقم بتحديد قيمة للمسار، سيتم استخدام اعدادات الـ `calling process`
- `30` : قمنا بتمرير الـ pointer الخاص بالـ `STARTUPINFO` structure
- `31` : قمنا بتمرير الـ pointer الخاص بالـ `PROCESS_INFORMATION` structure
- `33` - `37` : في حال فشل تنفيذ الدالة `CreateProcess` سيتم طباعة الخطأ باستخدام دالة `GetLastError`
- `39` - `40` : في هذه السطرين نسمح لعملية الـ `notepad.exe` أن تعمل حتى يتم انهاءها بشكل يدوي من المستخدم ، يتم تنفيذ هذا عن طريق الدالة `WaitForSingleObject`
- `43` - `44` : قمنا بقراءة الـ `exit code` الخاص بالعملية باستخدام دالة `GetExitCodeProcess`
- `47` - `48` : قمنا باغلاق الـ handles الخاصة بالعملية وبالـ Thread الخاص بها 

بعد تنفيذ الكود، سنرى أنه استطعنا بدء عملية الـ `notepade.exe`
</div> 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/ProcessCreationFunctions/1.png)



<div dir="rtl" markdown="1">

> ℹ️ [ملاحظة]
> هذه المقالة تمّت كتابتها خلال دراسة هذه المواضيع، فكل ما تم ذكره هنا قد يحتمل الخطأ، لكن بالإمكان العودة إلى المراجع التي إستندت عليها هذه المقالة 
</div>


<div dir="rtl" markdown="1">

# المراجع
</div> 

-  [Windowns Internals 7th Edition](https://learn.microsoft.com/en-us/sysinternals/resources)
-  [Windows System Programming](https://www.amazon.com/Programming-Paperback-Addison-Wesley-Microsoft-Technology/dp/0134382250)
