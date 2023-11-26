---
title: Projects
permalink: /projects/
layout: page
excerpt: 
comments: false
---

## **Red Teaming Related Projects/Scripts :**
- **[`ADEnum.ps1`](https://github.com/0xb1tByte/CRTE-Journey/tree/main/Scripts)**
  - **Description** : A simple script to automate some of manual AD enumeration commands.
  - **Programming / Scripting language** : `Powershell` 
  - **Status** : Done
- **[`ProcessInjection.py`](https://github.com/0xb1tByte/OSEP-Journey/tree/main/Scripts/ProcessInjection)**
  - **Description** : A PoC script for **Process Injection** through `Python`. 
  - **Programming / Scripting language** : `Python`
  - **Status** : Done
- **[`ProcEnum.py`](https://github.com/0xb1tByte/MalDev-Journey/tree/main/Scripts)**
  - **Description** : Processes Enumeration using Win APIs : [`CreateToolhelp32Snapshot`](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-createtoolhelp32snapshot) & [`Process32First`](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-process32first) & [`Process32Next`](https://learn.microsoft.com/en-us/windows/win32/api/tlhelp32/nf-tlhelp32-process32next)
  - **Programming / Scripting language** : `Python`
  - **Status** : Done
    
## **Web Pentesting Related Projects :**
- **[`SMTPFuzzer.py`](https://github.com/0xb1tByte/OSWE-Journey/tree/main/Scripts/SMTPFuzzer)**
  - **Description** : This script can be used to automate the **injection-based** vulnerabilities (xss,sqli .. etc) against **SMTP headers**
  - **Programming / Scripting language** : `Python`
  - **Status** : Done
- **[`tinyXSSFuzzer.py`](https://github.com/0xb1tByte/OSWE-Journey/tree/main/Scripts/XSSFuzzer)**
  - **Description** : a tiny XSS fuzzer (template script actually) that can read XSS payloads from txt file or XML file, and iterate through each payload and inject it into a POST parameter, or JSON parameter , no such magic here
  - **Programming / Scripting language** : `Python`
  - **Status** : Done
- **[`ParsingWSDLinks.py`](https://github.com/0xb1tByte/Pentest-Scripts/tree/main/Web)**
  - **Description** : A tiny script that fetches the content of `spdisco.aspx` file (a directory of all web services available in SharePoint) and extracts all **WSDL links**, then passes these links to Burp where the **WSDLER** tool can be used to parse each WSDL file
  - **Programming / Scripting language** : `Python`
  - **Status** : Done
- **`AutoBlindSQLite.py`**
  - **Programming / Scripting language** : `Python`
  - **Status** : Unpublished yet (working on the white-paper of the project)

## **MSc Courses mini Projects:**
- **[PRESENT block cipher for 16-bits Micro-controller](https://github.com/0xb1tByte/PRESENT)**
  - **Description** :
    - Developed a reference implementation of the PRESENT encryption algorithm for the [MSP430 Micro-controller](https://www.ti.com/microcontrollers/msp430-ultra-low-power-mcus/overview.html).
    - Developed an optimized implementation of the PRESENT algorithm for the Micro-controller using the Bits-slicing technique.
  - **Course** : Hardware Security
  - **Programming / Scripting language** : `C` 
  - **Status** : Done

- **[Proxy Server](https://github.com/0xb1tByte/PWN/tree/master/PwnAdventure)** 
  - **Description** :
    - Developed a proxy server for the [Pwn Adventure 3](https://www.pwnadventure.com/) game.
    - The proxy server analyses a few packets between the game server and the client.
  - **Course** : Penetration Testing
  - **Programming / Scripting language** : `Java` 
  - **Status** : Done
    
- **[BOFunctions](https://github.com/0xb1tByte/RE/tree/master/BOFunctions)**
  - **Description** :
    - Developed an IDC script for IDA to find vulnerable functions to Buffer Overflow vulnerability.
  - **Course** : Penetration Testing
  - **Programming / Scripting language** : `IDC` 
  - **Status** : Done
    
- **[SHA1Collision](https://github.com/0xb1tByte/Postgraduate/tree/master/Semester_1/Cryptography/Assignments/SHA1Collision)**
  - **Description** : A simple `Java` program that finds a collision for the first **60 bits** (15 nibbles) of `SHA-1`.
  - **Course** : Crypto
  - **Programming / Scripting language** : `Java` 
  - **Status** : Done
    
- **[Modular Exponentiation](https://github.com/0xb1tByte/Postgraduate/tree/master/Semester_1/Cryptography/Assignments/Modular%20Exponentiation)**
  - **Description** : A simple `Java` program that automates some operations in Modular 
  - **Course** : Crypto
  - **Programming / Scripting language** : `Java` 
  - **Status** : Done
    
## **Just For Curiosity & Exploring APIs (not completed):**
- **[pythoNMAP](https://github.com/0xb1tByte/eCPPTv2-Journey/tree/master/Projects)**
  - **Programming / Scripting language** : `Python`
  - **Status** : not completed
    
- **[shellcodeHelper](https://github.com/0xb1tByte/eCPPTv2-Journey/tree/master/Projects)**
  - **Programming / Scripting language** : `Python`
  - **Status** : not completed
 
- **[ParsingHTML](https://github.com/0xb1tByte/eWPTXv1-Journey/tree/master/XSS%20-%20Filter%20Evasion%20and%20WAF%20Bypassing/Scripts/ParsingHTML)**
  - **Programming / Scripting language** : `Java`
  - **Status** : not completed
