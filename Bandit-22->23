# Bandit Level 22 → 23

## Objective

A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in `/etc/cron.d/` for the configuration and see what command is being executed.

***NOTE:** Looking at shell scripts written by other people is a very useful skill. The script for this level is intentionally made easy to read. If you are having problems understanding what it does, try executing it to see the debug information it prints.*

## Concept Tested

Predictable/derivable file naming as a security flaw. The script builds its "secret" output path from deterministic inputs (a fixed string + a known username), which means anyone who can read the script can compute the path in advance rather than needing to discover it.

## Initial Access

Establish an SSH session to the target host as user `bandit22`:

```bash
$ ssh bandit22@bandit.labs.overthewire.org -p 2220
bandit22@bandit.labs.overthewire.org's password: [REDACTED]
bandit22@bandit:~$
```

## Recon

Using the same approach as in level21, I'm going to run the list command on the path `/etc/cron.d` to identify the file/script I'll be interacting with this challenge along with its ownership and permissions.

```bash
bandit22@bandit:~$ ls -la /etc/cron.d
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

Seeing as I need the password for level 23, I'll focus my attention on `cronjob_bandit23`. So now I'll `cat` the file to view how the script will operate.

```bash
bandit22@bandit:~$ cat /etc/cron.d/cronjob_bandit23
@reboot bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
```

This output confirms that the pattern of this is the same as before. The script runs every minute, as `bandit23`, discarding its output. I'll be exploiting that privilege gap again. 

```bash
bandit22@bandit:~$ cat /usr/bin/cronjob_bandit23.sh
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
```

Upon revealing the file's script using the `cat` command, I have noticed that it is doesn't have a fixed filename like the previous level and that it computes one at runtime. This is how it works, line by line:

- `myname` captures whoever's running it (`bandit23` via cron).
- `mytarget` is the MD5 hash of the literal string `"I am user "` concatenated with that username.
- `cut -d ' ' -f 1` trims `md5sum`'s output down to just the hex digest (md5sum normally prints `<hash> <filename-or-dash>`). 
- `/tmp/<that hash>` is the path where `bandit23`'s password is written to and with no `chmod` this time, it inherits whatever default `umask` produces, which on this system is world-readable by default (same net effect as level 22, arrived at differently).

```bash
bandit22@bandit:~$ ls -la /tmp/8ca319486bfbbc3663ea0fbe81326349
-rw-r--r-- 1 bandit23 bandit23 33 Aug 30 05:35 /tmp/[REDACTED]
```

Ran this command to confirm that the default `umask` was world-readable.

## Solution

```bash
bandit22@bandit:~$ echo "I am user bandit23" | md5sum | cut -d ' ' -f 1
[REDACTED]
```

Firstly I ran the script's exact hashing logic myself, substituting `$myname` for `bandit23` since I know that's the value cron will supply. Because MD5 is a deterministic function — the same input always produces the same output — I don't need to wait and "discover" the filename; I can compute the exact value the script will use before it even runs again.

```bash
bandit22@bandit:~$ cat /tmp/[REDACTED]
[REDACTED]
```

Since I know that when the script runs it passes password to `mytarget` so all I needed to do my find out is what `mytarget` is, then complete the path `/tmp/mytarget` so I can then `cat` that path to get the password and since it's left world-readable, I can access it directly as `bandit22` with no further trickery.

## Verification

Authenticate as `bandit23` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit23@bandit.labs.overthewire.org -p 2220
bandit23@bandit.labs.overthewire.org's password: [REDACTED]
bandit23@bandit:~$ whoami
bandit23
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit23@bandit:~$`) confirms successful authentication as `bandit23`.

## Why This Works

The script's author likely intended the MD5 hash to act as an unguessable filename — a crude attempt at "security through obscurity" for where the secret gets dropped. But MD5 is a pure, deterministic function: given the same input, it always produces the same output, with no secret key or randomness (salt) involved anywhere in the computation. Since the inputs (the literal string `"I am user "` and the target username) are both fully visible in the world-readable script itself, "obscurity" collapses to zero — anyone can run the exact same one-line command and land on the exact same filename. This is functionally identical to level 22's flaw (a privileged script leaking a secret to a world-readable temp file); only the mechanism for finding the filename changed, from "hardcoded and directly visible" to "computable in one command."

## Key Takeaway

Never use a hash of predictable, non-secret inputs as a stand-in for real access control or a genuinely unguessable identifier — a hash only provides secrecy if at least one of its inputs is itself secret (e.g., a random nonce or a key). If a privileged process needs to hand a value to a specific user, restrict the destination file's permissions and ownership explicitly (e.g., `chmod 600`, owned by the intended recipient) rather than relying on obscurity of the filename.

## Password

`[REDACTED - see level for retrieval method]`


---
