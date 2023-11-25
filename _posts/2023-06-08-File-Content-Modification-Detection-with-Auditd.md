---
layout: post
published: true
title: File Content Modification Detection with Auditd
date: '2023-06-08'
tags:
- auditd
- Blue Team
- Adversary Simulation
- English-Article
- Linux
---


# Scope & Required Knowledge
Before we dive in, let me tell you what this article is about and what it is not.

This article is not meant to cover:
- How `auditd` works
- How to configure `auditd` rules
- Explaining the record types of `auditd`

But it is about:
- How to detect if a file content was changed using `auditd` logs
- A deep dive into `open`/`openat` system call - the `SYSCALL` record of `auditd`

It's **highly** recommended to read these articles before reading my article:
- [Linux auditd for Threat Detection [Part 1]](https://izyknows.medium.com/linux-auditd-for-threat-detection-d06c8b941505)
- [Linux auditd for Threat Detection [Part 2]](https://izyknows.medium.com/linux-auditd-for-threat-hunting-part-2-c75500f591e8)
- [Linux auditd for Threat Detection [Final]](https://izyknows.medium.com/linux-auditd-for-threat-detection-final-9d5173706b3f)

All the examples and commands in this article were tested on : 
- `CentOS 7` and `Kali 2021.4`.
- The results and outputs may vary depending on the distribution and version of Linux you are using.

# Overview
Last month, I was reading and learning about [`auditd`](https://linux.die.net/man/8/auditd), a Linux audit system that gives us the capability to see all actions from the kernel's perspective.

 ![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/file-change-detection-with-auditd/1.png#center)
 - Image from [Auditd: Rule Writing for better Threat Detection on *nix Devices](https://www.researchgate.net/publication/355181208_Auditd_Rule_Writing_for_better_Threat_Detection_on_nix_Devices/download)

One of the cases that I was investigating with `auditd` was **file content change detection**. I wanted to use `auditd` to detect if any of the files had been modified or changed in any way, such as by an attacker altering the configuration of a web server (the `.htaccess` file for instance), or updating the SSH server keys (or the `authorized_keys` file). I ran the tests on two Linux distributions, `Centos 7` and `Kali 2021.4`, using the standard `auditd` rule for tracking file changes. I should mention that I made a minimal change on that rule (which I will describe later). The first test ran on `Centos`, and the last test ran on `Kali`. In `Centos`, I found that the rule was generating logs **every time** the file was accessed with `write` permission, which does not necessarily mean that the file content had been updated. In `Kali`, however, the situation was different. I applied the same `auditd` rule to detect file content changes, but it was a straightforward process and the `auditd` rule generated logs **only when the file content was changed**.

In this article, I will share with you the results, analysis and observations of these two tests, and how I found a way to detect file content changes in `Centos`.

Let the hunt begin!

# The Standard Auditd Rule for Tracking File Changes
One of the resources that I found online that suggested a way to monitor file changes using `auditd` was [this article](https://access.redhat.com/solutions/10107) from RedHat. This article provides an example of an `auditd` rule that is supposed to log any attempt to access a file with `write` (`w`), `read` (`r`), and `attribute change` (`a`) permissions.

```
auditctl -w /etc/hosts -p war -k monitor-hosts
```

in my case, I was only interested in `write` activities. So I updated the rule to be like :
```
auditctl -w /etc/hosts -p w -k hosts_file_content_changed
```

I tested the rule in two scenarios. In the first scenario, I **opened the file without making any changes**. In the second scenario, I **opened the file and updated it**. I did both scenarios on `Centos` and `Kali` machines. Let’s first see the results on `Centos` machine.


## Testing The Rule on `Centos`
### Scenario `#1` : File opened, no update
In this scenario, I opened the file with the `nano` editor, but I did not make any change. As shown in the screenshot below, the rule was triggered (**one** log entry)

 ![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/file-change-detection-with-auditd/2.png#center)

We can see that **one** log entry was generated with the type `SYSCALL`. This event indicates that the `open` syscall (see the ID in the `syscall` field) was successfully invoked (see the `success` field). A quick look at the man page of the [`open`](https://man7.org/linux/man-pages/man2/open.2.html) system call reveals the following definition:

> The open() system call opens the file specified by pathname.

However, the `open` system call is not only used for opening files. It can also be invoked when a file is created or some content is written into a file. This is stated on the same man page of the open system call.

> If the specified file does not exist, it may optionally (if O_CREAT is specified in flags) be created by open().

This sentence means that the `O_CREAT` flag can be used to create a file when it is passed to the `open` system call. 

Now, let’s examine the flags that were passed to the `open` syscall when we only opened the file, without modifying its content. We can spot the `open` syscall flags in the `a1` field [[1]](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security_guide/sec-understanding_audit_log_files) . In this scenario the value was 

```
 a1=441
```

Let's examine the `flags` parameter of the `open` system call in more detail. The man page of `open` says this about flags:

>   The argument flags must include one of the following access
>       modes: O_RDONLY, O_WRONLY, or O_RDWR.  These request opening the
>       file read-only, write-only, or read/write, respectively.

>  In addition, zero or more file creation flags and file status
>       flags can be bitwise-or'd in flags.  The file creation flags are
>       O_CLOEXEC, O_CREAT, O_DIRECTORY, O_EXCL, O_NOCTTY, O_NOFOLLOW,
>       O_TMPFILE, and O_TRUNC.  The file status flags are all of the
>       remaining flags listed below. 

This means that `flags` consists of **three parts** that can be combined with the `bitwise OR operator` (`|`):
1. `An access mode`
2. Zero or more `file creation flags`
3. Zero or more `file status flags`

Let’s revisit the `flags` value that is passed through the `a1`. The `a1=441` is a hexadecimal representation. This corresponds to `O_WRONLY | O_CREAT | O_APPEND` (each flag has a constant value defined in the [fcntl.h](https://github.com/torvalds/linux/blob/master/include/uapi/asm-generic/fcntl.h) file), which means opening the file for **writing only**, **creating it** if it does not exist, and **appending to it** if it already exists.

Also, to interpret the hex values, we can use the [`ausearch`](https://man7.org/linux/man-pages/man8/ausearch.8.html) tool with this command:

```
 ausearch -i -a <event-id>
```

 ![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/file-change-detection-with-auditd/7.png#center)


Let’s revisit our rule too. As you may remember, the rule was only logging the `write` permission, and by looking at the log generated by the rule we can see that the `O_WRONLY` flag is present in the `flags` value of the `open` system call. This makes sense, but it is not enough to detect content change, because we cannot be sure that the file was actually modified based on this log record alone. 

In the next section, we will see a more reliable record that we can use to identify the file content change

### Scenario `#2` : File opened, content updated
In this scenario, I opened the file with `nano` again, but this time I **changed the content**. As shown in the screenshot below, the rule was triggered and generated **two** log entries.

 ![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/file-change-detection-with-auditd/3.png#center)

The first record is identical to the one we saw in Scenario `#1`. To recap, we can see that the `a1` value is `441`, which means `O_WRONLY | O_CREAT | O_APPEND`. The second record has a different `a1` value, which is `241`. It is important to note that the first record was generated when the **file was opened**, while the second record was generated **after I modified the file content and closed the file**.

Let’s examine the second record more closely. We can see that the `a1` value is `241`. This represents `O_WRONLY | O_CREAT | O_TRUNC` as we can see with `ausearch` result

 ![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/file-change-detection-with-auditd/8.png#center)

This means that the file is opened for **writing only**, **creating it** if it does not exist, and **truncating it** if it already exists

Comparing the two scenarios, we can conclude that:
- for Scenario `#1` : when the file is opened with `write` permission but the content is not updated, we would expect to see **one** `SYSCALL` record indicating that the `open` system call was invoked with `a1=441` (`O_WRONLY | O_CREAT | O_APPEND`).
- for Scenario `#2`: when the file is opened with `write` permission and the content is updated, we would expect to see **two** `SYSCALL` records, both indicating that the `open` system call was invoked, but with different values of `a1`: the first event has `a1=441` (`O_WRONLY | O_CREAT | O_APPEND`) and the second event has `a1=241` (`O_WRONLY | O_CREAT | O_TRUNC`).

Let’s turn now to the `Kali` machine scenarios.

## Testing The Rule on `Kali`
### Scenario `#1` : File opened, no update
In this scenario, **no log entry** was generated, as we can see in the screenshot below. `Kali` seems to treat the `write` permission defined in the rule differently (I have not investigated this aspect).

 ![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/file-change-detection-with-auditd/4.png#center)

### Scenario `#2` : File opened, content updated
In this scenario, we only saw one record, as shown in the image below

 ![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/file-change-detection-with-auditd/5.png#center)
 
The generated record is similar to the one we saw in `Centos` when the file content was updated, except that the [`openat`](https://linux.die.net/man/2/openat) system call (`257`) was invoked instead of `open` and the `flags` were passed through the `a2` not `a1`. However, the `openat` record has the same value for the `flags` as the `open` record, which is `a2=241` (`O_WRONLY | O_CREAT | O_TRUNC`).

 ![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/file-change-detection-with-auditd/9.png#center)

# Summary 
In this article, I have shown how to use `auditd` to monitor file changes in Linux. I have also observed some interesting findings: `auditd` behavior is not the same in different Linux distributions, such as `Kali` and `Centos`. This is may be because each distribution has its own kernel, and `auditd` simply reports what the kernel sees for a given activity. Therefore, when building `auditd` rules, it is important to test them in the target distribution and adjust them accordingly. In short, one `auditd` rule might not generate the same logs in all Linux distributions. This is a crucial point to keep in mind when building your detection rules. 

The diagram below summarizes the two test scenarios in Kali and Centos.
 ![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/file-change-detection-with-auditd/6.png#center)

