---
layout: post
published: true
title: Exploring system() in SUID Programs
date: '2023-01-16'
tags:
  - Red-Teaming
  - Linux
  - English-Article
---


While preparing the lab example for my [SUID Part 1](https://0xb1tbyte.github.io/2022-12-31-suid-programs-1/) article, I wrote a sample program to demonstrate how SUID Programs can be exploited using Environment Variables, and as someone who is still learning the concepts, I missed writing a few important lines in the program (so that it can be exploited), but guess what? this mistake turned into an interesting learning journey !

let’s stop talking words and speak codes

## `system()` in SUID Program without calling `setuid()` 

the first code I wrote was like this 

```c
#include <stdio.h>
#include <stdlib.h>
  int main () 
  {
	system("whoami");
  }
```

and I thought that the `system()` will have the `root` privilege as it is already part of the SUID program .. and this is actually the concept behind the SUID programs (or at least this is how I understand it), the process of the SUID programs is privileged

The first time I run the program, it printed my current user (`kali`), not the `root`

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/system-in-suid/1.png)

at that time, I thought I made a mistake in setting the SUID bit, reviewed the commands I’ve run, and the program is a real SUID program (the `root` is the owner), but the `system()` still running within my current user privilege ! 

Doing a quick search to see how people use `system()` in SUID programs, and found that all the codes examples are calling the `setuid()`  & `setgid()` and passing the required privilege to it (the user id which we seek his privilege) first before calling the `system()`  

so if we want the `system()` to run within the `root` context, the code would be like this

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
  int main () 
  {
	setuid (0); // passing root priv
	system("whoami");
  }
```

![2](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/system-in-suid/2.png) 

and this finding confused me more, we are supposed to be in the context of the SUID program, and as per the SUID concept, the `euid` value of the process will be obtained from the owner of the file, **so why we would need to set the** `euid` **explicitly**? is not the program already have the `euid` set?

to get the answer to this question I started to understand three things:

- The first one: **How the program calls the** `system()` .. what’s happening under the hood ? 

- The second one: **What do** `setuid()` **and** `setgid()` **actually do?**

- The Third one (a Review): **The** `uid` **and** `euid` **values in Normal VS SUID programs**

let’s start from the bottom 

## **The** `uid` **and** `euid` **values in Normal VS SUID programs**

to check the values of the `uid` and `euid` we can use the below C code 

```c
#include <stdio.h>
#include <unistd.h>
  int main () 
  {
  int real = getuid();
  int euid = geteuid();
  printf("The REAL UID =: %d\n", real);
  printf("The EFFECTIVE UID =: %d\n", euid);
  }
```

Compiling the program

```bash
gcc printIDs.c -o testIDs
```

the permissions of the program before setting the SUID bit

![3](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/system-in-suid/3.png) 

after running the program we can see that the `uid` and `euid` both have the same value `1000` (our `kali`user)

![4](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/system-in-suid/4.png)

let’s set the SUID bit now 

```bash
sudo chown root testIDs
```

```bash
sudo chmod 4755 testIDs
```

after running the program we can see now that its `euid` value is `0` (has `root` privilege), but the `uid` is still `1000` (`kali` user)

![5](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/system-in-suid/5.png)

So : 

- in **Normal Programs** : the `uid` and `euid` both have the same value
- in **SUID programs**: the `uid` value will be the value of the user who starts the program, and the `euid` value will be the value of the owner of the program

let’s now bring the `system()` function to the context and see what would be the output?

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
  int main () 
  {
  int real = getuid();
  int euid = geteuid();
  printf("The REAL UID =: %d\n", real);
  printf("The EFFECTIVE UID =: %d\n", euid);
	system("whoami");
  }
```

first let’s run the program again as **normal program** without SUID bit 

![6](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/system-in-suid/6.png)

so far all good, let’s now set the SUID for the same program and see the result

![7](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/system-in-suid/7.png)

the `euid` is `0`, but the `whoami` command we passed to `system()` function said that we are `kali` ?!

and even if we pass “`/bin/bash`” to `system()` function , we will get a `kali` shell not `root` shell

what if we call the `setuid()` and `setgid()` before calling `system()` as suggested in the internet examples?

## **What do** `setuid()` **and** `setgid()` **actually do?**

by modifying the previous code, and calling the `setuid()` **and** `setgid()`

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
  int main () 
  {
  int real = getuid();
  int euid = geteuid();
  printf("The REAL UID =: %d\n", real);
  printf("The EFFECTIVE UID =: %d\n", euid);
  printf("Before calling setuid() and setgid() \n");
	system("whoami");
	setgid(0);
	setuid (0);
  int real2 = getuid();
  int euid2 = geteuid();
  printf("After calling setuid() and setgid() \n");
	system("whoami");
  printf("The REAL UID =: %d\n", real2);
  printf("The EFFECTIVE UID =: %d\n", euid2);
  }
```

running as **normal program** (of course the `setuid()` and `setgid()` calls will fail)

![8](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/system-in-suid/8.png)

running as SUID 

![9](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/system-in-suid/9.png)

the above screenshot tells us a lot! 

- before calling the `setuid()` and `setgid()` : we can see that only the `euid` value was changed (because it’s a SUID program, it only affects the `euid` value)
- After calling the `setuid()` and `setgid()`: we can see that **both** the `uid` and `euid` were changed, and if we check the [setuid()](https://man7.org/linux/man-pages/man2/setuid.2.html) page, we can see the below definition for the function
    
    > `setuid()` sets the effective user ID of the calling process.
    > 

So the function would update only the `euid` value ? no, there is a special case 

if we continue reading the document we will find the below phrase 

> If the calling process is privileged (more precisely: if the process has the CAP_SETUID capability in its user namespace), the real UID and saved set-user-ID are also set.
> 

the above means that if we call `setuid()` from within a SUID program (`root` is the owner) it will set both the `uid` and the `euid` .. as we’ve seen in the screenshot above 

at that phase, I realized that it’s important that the `uid` and `euid` both should have the `0` value so we can have `system()` running as root .. 

but I was still confused, as the access control of the process is defined based on the `euid` value  only, and `uid` is just representing the user who initiated that process, so why it’s important to `system()` to have both `uid` and `euid` as `0`

from here, I started to read more about the `system()` as there seem to be some secrets hidden in it

## **How the program calls the** `system()` .. **what’s happening under the hood ?**

I wrote an [article](https://0xb1tbyte.github.io/2019-12-16-Injection-Attacks-on-System/) before about how `system()` works. In short, `system()` runs `/bin/sh` to initiate the commands as mentioned on its page

```bash
execl("/bin/sh", "sh", "-c", command, (char *) NULL);
```

reading more on `system()` page and found the below phrase

> system() will not, in fact, work properly from programs with set-user-ID or set-group-ID privileges on systems on which/bin/sh is bash version 2: as a security measure, bash 2 drops privileges on startup.
> 

Nice!!! this seems to be an answer to my first question! 

it says that `system()`  won’t work as it should in the context of the SUID program, and the `/bin/sh` will drop the privilege as a security measure

digging deeper into `/bin/sh` and reading the [sh](https://man7.org/linux/man-pages/man1/sh.1p.html) page reveals to us the below two phrases 



```
 -i               Specify that the shell is interactive; see below. An
                 implementation may treat specifying the -i option as an
                 error if the real user ID of the calling process does
                 not equal the effective user ID or if the real group ID
                 does not equal the effective group ID.
 ```


```
 ENV             shall be ignored if the real and effective user IDs or real and
                 effective group IDs of the process are different.
 ```


So, from the above, we can see that the [sh](https://man7.org/linux/man-pages/man1/sh.1p.html) will drop the privilege whenever the `uid` and the `euid` of the process are different .. and that was actually the scenario in my initial program 😄

```c
#include <stdio.h>
#include<stdlib.h>
  int main () 
  {
	system("whoami");
  }
```

even if we set the SUID bit, the [sh](https://man7.org/linux/man-pages/man1/sh.1p.html) will drop the privilege as the program still have different values for `uid` and `euid`. and that’s why we need to call `setuid()` and `setgid()` before `system()`, to update the values for all user IDs, which won’t happen unless the process is privileged.


	
