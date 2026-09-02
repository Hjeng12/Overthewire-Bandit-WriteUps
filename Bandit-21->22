# Bandit Level 21 → 22

## Objective

A program is running automatically at regular intervals from `cron`, the time-based job scheduler. Look in `/etc/cron.d/` for the configuration and see what command is being executed.

## Concept Tested

Reading cron job definitions (`/etc/cron.d`) to discover scheduled/automated processes running with privileges you don't have, and following the script it invokes to find a security flaw — here, a script that writes a secret to a world-readable location.

## Initial Access

Establish an SSH session to the target host as user `bandit21`:

```bash
$ ssh bandit21@bandit.labs.overthewire.org -p 2220
bandit21@bandit.labs.overthewire.org's password: [REDACTED]
bandit21@bandit:~$
```

## Recon

Since the objective stated the location of the path where the automated script ran (essentially where the challenge will take place) I will list the scripts running in that directory and identify their ownership and permissions.

```bash
bandit21@bandit:~$ ls -la /etc/cron.d
total 56
drwxr-xr-x   2 root root  4096 Jul  3 16:19 .
drwxr-xr-x 124 root root 12288 Aug 17 21:05 ..
-rw-r--r--   1 root root   102 Nov  5  2025 .placeholder
-r--r-----   1 root root    47 Jun 24 14:59 behemoth4_cleanup
-rw-r--r--   1 root root   127 Jul  3 16:19 clean_tmp
-rw-r--r--   1 root root   120 Jun 24 14:58 cronjob_bandit22
-rw-r--r--   1 root root   122 Jun 24 14:58 cronjob_bandit23
-rw-r--r--   1 root root   120 Jun 24 14:59 cronjob_bandit24
-rw-r--r--   1 root root   188 Feb 13  2026 e2scrub_all
-r--r-----   1 root root    48 Jun 24 15:00 leviathan5_cleanup
-rw-------   1 root root   138 Jun 24 15:01 manpage3_resetpw_job
-rwx------   1 root root    52 Jun 24 15:02 otw-tmp-dir
```

Due to the fact that I want the password for `bandit22`, I'll interact with `cronjob_bandit22` first. Seeing as `cronjob_bandit22` is a file I can read, I'll `cat` it and see what is outputted. Cron job entries tend to be plain text and human-readable so I suspect the `cat`command to be enough.

```bash
bandit21@bandit:~$ cat /etc/cron.d/cronjob_bandit22
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
```

This output tells me four things:

- `* * * * *` tells me that every minute.
- `bandit22` it runs as this user.
- `/usr/bin/cronjob_bandit22.sh` is what executes.
- `&> /dev/null` discards all output.

```bash 
bandit21@bandit:~$ ls -la /usr/bin/cronjob_bandit22.sh
-rwxr-x--- 1 bandit22 bandit21 130 Jun 24 14:58 /usr/bin/cronjob_bandit22.sh
```

This script runs as `bandit22`, which is an important detail as I'm currently `bandit21` but when I ran the list command for the script it revealed that my current user, `bandit21` can read and execute the script. 

```bash
bandit21@bandit:~$ cat /usr/bin/cronjob_bandit22.sh
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

Since the cron entry pointed to this script, and scripts (unlike compiled binaries) are just readable text, I `cat` it directly since executing it as `bandit21` would fail as I don't have the permission, so reading the source is the reliable way to understand its actual behavior. This revealed the bug immediately: the script `chmod`s a fixed-name file in `/tmp` to `644` (world-readable) and then writes bandit22's password into it — as `bandit22`, since that's who cron runs it as.

## Solution

```bash
bandit21@bandit:~$ cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
[REDACTED]
```

Seeing as the script runs every minute as bandit22, the password file at that fixed `/tmp` path is continuously being (re)written and left world-readable. I don't need any privilege escalation trick — I just read the file directly as bandit21, since `644` permissions grant read access to everyone, not just the file's owner.

## Verification

Authenticate as `bandit22` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit22@bandit.labs.overthewire.org -p 2220
bandit22@bandit.labs.overthewire.org's password: [REDACTED]
bandit22@bandit:~$ whoami
bandit22
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit22@bandit:~$`) confirms successful authentication as `bandit22`.

## Why This Works

Cron jobs execute with whatever privileges are configured for them, here as `bandit22`, completely independent of who's watching. That alone isn't a flaw; the flaw is what the _script_ does with that privilege: it writes a secret to a predictable, fixed path in `/tmp` (a world-writable, world-readable by default directory) and then explicitly loosens permissions on that file to `644`. Since `/etc/cron.d/*` files and `/usr/bin/*.sh` scripts are themselves world-readable by design (so any user can inspect what's scheduled), any user on the box can trace the exact path cron will write the secret to before cron even runs it. There's no race condition or timing needed, just reading two plaintext files.

## Key Takeaway

Never have a privileged scheduled task write secrets to a predictable, shared location like `/tmp` with permissive access. If a script needs to hand a value to a specific lower privileged user, it should write to a location owned and restricted to that user (or use a proper IPC/secrets mechanism), not a fixed filename anyone can `cat`. Also remember: cron configs and the scripts they invoke are usually readable by any local user, so "security through cron" only works if what the job does is actually secure, not just because the schedule itself is somewhat hidden.

## Password

`[REDACTED - see level for retrieval method]`

---
