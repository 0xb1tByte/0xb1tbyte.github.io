---
layout: post
published: true
title: Process Creation Functions Part 1 
date: '2023-08-04'
tags:
- Red-Teaming
- Blue Team
- Malware Development
- Arabic-Article
- Windows
- Windows Internals
---

<div dir="rtl" markdown="1">

السلام عليكم ورحمة الله وبركاته 

المقالة هذه ( وسلسلة المقالات اللاحقة إن يسرها الله ) تلخيص لقرائتي لبعض الفصول من كتاب [Windowns Internals 7th Edition](https://learn.microsoft.com/en-us/sysinternals/resources/windows-internals) 

قبل أن نبدأ أود الاشارة بأن سلسلة المقالات هذه تتوقع من القارئ الإلمام بمفاهيم أنظمة التشغيل ولغات البرمجة ، فلن يتم الاسهاب في هذا الجانب

- الطلاب الأعزاء الذين يقرأون السلسلة، مُرحب بأسئلتكم [هنا](https://0xb1tbyte.github.io/calendar/)

بسم الله نبدأ 

# ماهي العمليات ( Processes ) ؟ 
عرّف الكتاب العمليات كالآتي :

</div>

> A program is a static sequence of instructions, whereas a process is a container for a set of resources used when executing the instance of the program.

<div dir="rtl" markdown="1">
في نظري هذا من أذكى التعاريف التي مرّت عليّ في شرح معنى العمليات، فقد أظهر المعنى الحقيقي للعمليات بطريقة واضحة ومُختصرة

يخبرنا الكتاب أن العمليات ماهي إلّا `Container` يحوي العديد من المصادر حتى يُتم البرنامج عمله .. بلغة أخرى ـوكما سنعرف لاحقًا- ليست العمليات **بحد ذاتها** التي تقوم بتنفيذ ( `Execute` ) البرنامج ، انما هي **مخزن -إن صح التعبير- يحوي المصادر التي يحتاجها البرنامج**

# مكوّنات العمليات 
في أنظمة `Windows` تتكوّن العمليات من الآتي ( القائمة تطول لكن هذه أهم النقاط بشكل عام ) : 
- `A Private Virtual Address Space` : مساحة من العناوين الخاصة بالعملية، كل عملية لديها مجال ذاكرة افتراضي خاص بها ولا يمكن الوصول إليه من قبل عمليات أخرى إلا إذا كان مشتركًا ، لا تستطيع العمليات كذلك معرفة الـ `physical address` ولا تحتاج لذلك
- `An Executable Program` :  يشير الى البرنامج الذي سيتم تنفيذه
- `A List of Open Handles` : في نظام ويندوز يوجد ما يسمى بالـ `Handles` الفكرة من هذا المفهوم بشكل مختصر جدًا هو أن المصادر في نظام التشغيل ( مثل العمليات / الملفات وغيرها ) يمكن الاشارة لها بمعرف، بمعنى آخر للوصول للعملية للتعامل معها يجب الوصول أولًا للـ handle المتاح لها ، سيتم إيضاح هذه النقطة لاحقًا في الكود
- `A Security Context` : يمثّل مايرتبط بالعملية من ناحية أمنية ، على وجه التحديد نقصد الـ `access token` والتي تحتوي على : المستخدم الذي قام بإنشاء العملية، المجموعة التي ينتمي لها المستخدم، الصلاحيات وغيرها
- `A Process ID` : معرّف يشير الى رقم العملية ، هذه القيمة من الممكن أن تتكرر ، لكن ليس لعمليتين تعملان في نفس الوقت
- `At Least One Thread of Execution` : الـ `Thread` هو الجزء المسؤول عن تنفيذ الكود ، ويجب أن تحتوي العملية على `Thread` واحد على الأقل

  
# دوال العمليات في Windows 
الجدول التالي يلخص أهم دوال العمليات في نظام ويندوز

| الدالة | الوصف | DLL |
| --- | --- | --- |
| `CreateProcess` | تنشئ عملية جديدة والـ `Thread` الخاص بها. تعمل العملية الجديدة ضمن صلاحيات العملية التي قامت بإنشاء هذه العملية ( `Caller` ). | `Kernel32.dll` |
| `CreateProcessAsUser` | تنشئ عملية جديدة والـ `Thread` الخاص بها. تعمل العملية الجديدة ضمن صلاحيات المستخدم المحدد. | `Advapi32.dll` |
| `CreateProcessWithLogonW` | مشابهة لدالة `CreateProcessAsUser` لكن تختلف عنها في انه لا حاجة لنداء دالة `LogonUser`. دالة `LogonUser` تتيح تعريف الـ `logon session` والـ `security token` للمستخدم. | `Advapi32.dll` |
| `CreateProcessWithTokenW` | تنشئ عملية جديدة والـ `Thread` الخاص بها. تعمل العملية الجديدة ضمن صلاحيات المستخدم المحدد باستخدام الـ `Token`. | `Advapi32.dll` |
| `OpenProcess` | تمكننا هذه الدالة من الوصول إلى عملية تم إنشاءها مسبقًا. | `Kernel32.dll` |

لمزيد من التفاصيل حول هذه الدوال أرشّح الاطلاع على [هذا المرجع](https://www.tenouk.com/crstufunction1.html)

# دالة `CreateProceess`
الكود التالي مثال على استخدام دالة `CreateProcess` و إنشاء عملية تبدأ برنامج الـ `Notepad`


</div>


```c
  // CreateProcess.cpp : This file contains the 'main' function. Program execution begins and ends there.
  //
  
  #include <iostream>
  #include <windows.h>
  #include <stdio.h>
  int main()
  {
      // Declare variables
      STARTUPINFO si;
      PROCESS_INFORMATION pi;
      BOOL result;
      DWORD exitCode;
  
      // Initialize variables
      ZeroMemory(&si, sizeof(si));
      si.cb = sizeof(si);
      ZeroMemory(&pi, sizeof(pi));
  
      // Create a process using the command line argument
      result = CreateProcess(
          L"C:\\Windows\\System32\\notepad.exe", // Module name
          NULL, // Command line
          NULL, // Process handle not inheritable
          NULL, // Thread handle not inheritable
          FALSE, // Set handle inheritance to FALSE
          0, // No creation flags
          NULL, // Use parent's environment block
          NULL, // Use parent's starting directory
          &si, // Pointer to STARTUPINFO structure
          &pi); // Pointer to PROCESS_INFORMATION structure
  
      if (!result)
      {
          printf("CreateProcess failed (%d).\n", GetLastError());
          return -1;
      }
  
      // Wait until child process exits
      WaitForSingleObject(pi.hProcess, INFINITE);
  
      // Get the exit code of the child process
      GetExitCodeProcess(pi.hProcess, &exitCode);
      printf("Child process exited with code %d.\n", exitCode);
  
      // Close process and thread handles
      CloseHandle(pi.hProcess);
      CloseHandle(pi.hThread);
  
      return 0;
  }
```


<div dir="rtl" markdown="1">

نستكمل النظر في الكود في مقالة لاحقة بإذن الله 

ختامًا، أود الإشارة بأن هذه المقالة تمّت كتابتها خلال دراسة هذه المواضيع، فكل ما تم ذكره هنا قد يحتمل الخطأ، لكن بالإمكان العودة إلى المراجع التي إستندت عليها هذه المقالة

وإن أصبت فمن توفيق الله وحده، وإن أخطأت فمن نفسي والشيطان

# المراجع 
-  [Windowns Internals 7th Edition](https://learn.microsoft.com/en-us/sysinternals/resources/windows-internals) 


</div>
