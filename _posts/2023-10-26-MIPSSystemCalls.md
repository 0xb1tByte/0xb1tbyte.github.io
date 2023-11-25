---
layout: post
published: true
title: MIPS System Calls
date: '2023-10-26'
tags:
- Secret
---

# `4` & `10` syscalls Example
```assembly
  .data
      booth: .asciiz "TCC"
      date: .asciiz "14-16 Nov 2023"
      msg: .asciiz "b1tByte is attending BlackHatMEA\n"
  
  .text
      main:
          li $v0, 4       ; syscall for print string
          la $a0, msg     ; load address of msg
          syscall         ; print msg
  
          li $v0, 4       ; syscall for print string
          la $a0, booth   ; load address of booth
          syscall         ; print booth
  
          li $v0, 4       ; syscall for print string
          la $a0, date    ; load address of date
          syscall         ; print date
  
          li $v0, 10      ; syscall for exit
          syscall         ; exit program
```

```
; DATA IN MEMORY
; booth
00c00000: 54434300 ; TCC
; date
00c00004: 31342d31 ; 14-1
00c00008: 36204e6f ; 6 No
00c0000c: 76203230 ; v 20
00c00010: 32330000 ; 23
; msg
00c00014: 62317442 ; b1tB
00c00018: 79746520 ; yte
00c0001c: 69732061 ; is a
00c00020: 7474656e ; tten
00c00024: 64696e67 ; ding
00c00028: 20426c61 ; Bla
00c0002c: 636b4861 ; ckHa
00c00030: 744d4541 ; tMEA
00c00034: 0a000000 ; \n
```

# Reference 
[1](https://www.youtube.com/watch?v=_3YWDq05zko)
