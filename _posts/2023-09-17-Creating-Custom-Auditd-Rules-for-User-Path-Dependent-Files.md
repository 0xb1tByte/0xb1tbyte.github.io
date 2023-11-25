---
layout: post
published: true
title: Creating Custom Auditd Rules for Username-Path Dependent Files 
date: '2023-09-17'
tags:
- auditd
- Blue Team
- English-Article
- Linux
---


## Scope

All the examples, info, and commands in this article were tested on :

- `CentOS 7`

## Recap & The Problem

In a [previous article](https://0xb1tbyte.github.io/2023-06-08-File-Content-Modification-Detection-with-Auditd/), we learned how to create a watch rule for detecting file content modification with auditd. This was our rule, and we also learned how to investigate the logs of this rule:

```bash
auditctl -w /etc/hosts -p w -k hosts_file_content_changed
```

However, the rule we described in that article was for a specific file with a specific path (the `/etc/hosts` file). What if we want to monitor a file that is **username-path dependent**, such as the `authorized_keys` file? How can we achieve this at scale (for different users)? Can a single rule cover this? Or do we need to create a specific rule for each user’s `authorized_keys` file?

The short answer is that we cannot create a single file system rule for different files that are **username-path dependent**.

Unfortunately, auditd has some limitations that can make it difficult to use in some scenarios. One of these limitations is that auditd does not support wildcards for file system rules [[1]](https://www.digitalocean.com/community/tutorials/how-to-write-custom-system-audit-rules-on-centos-7). This means that auditd cannot watch for files whose paths contain dynamic names (such as usernames directories). For example, this rule will not work:

```bash
auditctl -w /home/*/.ssh/authorized_keys -p w -k auth_keys_file_content_changed
```

## The Goal

The aim of this article is to show you how to create file system rules for files that reside in different users’ folders dynamically ( a script-based approach )

## A Case : Monitoring `authorized_keys` Files For Each User on CentOS
### WHY we need to audit the `authorized_keys` file?

The **`authorized_keys`** file contains the public keys of the users who are allowed to log in to the system via SSH. If an attacker can compromise this file, they can add their own public key and gain persistent access to the system without needing a password. This is a common technique used by malware and hackers to establish a backdoor and execute malicious commands on the system. This technique is classified as `Account Manipulation: SSH Authorized Keys (T1098.004)` by MITRE ATT&CK®.

An example of how an attacker might abuse this file is by copying the content of their `id_rsa.pub` file, which contains their public key, to the `authorized_keys` file of the target user. They can do this by using a command like:

```bash
cat ~/.ssh/id_rsa.pub | ssh targetUser@<target-ip> "cat >> /home/targetUser/.ssh/authorized_keys"
```

This will append the attacker’s public key to the end of the `authorized_keys` file of the target user. This will allow the attacker to log in as the target user via SSH using their private key.

Therefore, it is important to audit the **`authorized_keys`** file for any changes and alert the system administrator if any unauthorized keys are added or removed. MITRE ATT&CK® also recommends auditing this file as `File Modification (DS0022)`.

### Auditing the `authorized_keys` file

As we have seen, the `authorized_keys` file is a critical file that controls the SSH access to the system. Therefore, we need to monitor this file for any changes and generate audit records when it is modified. However, this is not a straightforward task with auditd , because of the following challenges:

- The `authorized_keys` file is located in the user’s home directory, which can vary depending on the system configuration and the user name.
- Auditd does not support wildcards for file system rules.

For example, we cannot use a simple rule like the following to watch the `authorized_keys` with auditd.

```bash
auditctl -w /home/*/.ssh/authorized_keys -p w -k auth_keys_file_content_changed
```

We can see in the following screenshot that the auditd rule engine did not accept the rule and it’s not applied 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/Creating-Custom-Auditd-Rules-for-User-Path-Dependent-Files/1.png)

Using the environment variables ( for example the `$USER` ) in the rule is not an option also

```bash
auditctl -w /home/$USER/.ssh/authorized_keys -p w -k auth_keys_file_content_changed
```

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/Creating-Custom-Auditd-Rules-for-User-Path-Dependent-Files/2.png)

These challenges make it impossible to create a single auditd rule that can watch the `authorized_keys` file for any user on the system. We need to find a way to overcome these limitations and create dynamic auditd rules that can cover all the possible locations of the `authorized_keys` file on the system.

### Dynamic auditd Rules for `authorized_keys`
Our approach will be to generate the auditd rules dynamically and write them to the `audit.rules` file, which is the main configuration file for auditd. We will use a script that scans the `/home` directory for any subdirectories that contain a `.ssh/authorized_keys` file and creates a corresponding rule for each one.

This is the final script 

```bash
#!/bin/bash
# This script generates auditd rules for authorized_keys files in /home directory
# It appends the rules to the /etc/audit/rules.d/audit.rules file and restarts the auditd service

# Find all authorized_keys files in /home directory
files=$(find /home -type f -name "authorized_keys")

# Loop through the files and create a rule for each one
for file in $files
do
  # Append the rule to the audit.rules file
  echo "-w $file -p w -k auth_keys_file_content_changed" >> /etc/audit/rules.d/audit.rules
done

# Restart the auditd service to apply the new rules
service auditd restart
```

After running the script we can see that the rule was loaded successfully, 

![1](https://raw.githubusercontent.com/0xb1tByte/0xb1tbyte.github.io/master/assets/media/Creating-Custom-Auditd-Rules-for-User-Path-Dependent-Files/3.png)

We can also use a cron job to run this script **periodically** and update the `audit.rules` file as new users are added or removed from the system. This way, we can ensure that our auditd rules are always up-to-date and cover all the `authorized_keys` files on the system

<br>

I would like to acknowledge [@VolatileKhalid](https://twitter.com/VolatileKhalid) for his valuable feedback and guidance on this article.
