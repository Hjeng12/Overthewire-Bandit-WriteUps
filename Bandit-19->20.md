# Bandit Level 19 → 20

## Objective

To gain access to the next level, you should use the setuid binary in the home directory. Execute it without arguments to find out how to use it. The password for the level can be found in the usual place (/etc/bandit_pass/), after you have used the setuid binary.

## Concept Tested

SUID (Set owner User ID) binaries indicates how the setuid bit changes a process's effective UID to the file's owner, regardless of who actually runs it, and how that's used (legitimately or otherwise) to grant controlled privilege escalation. 

## Initial Access

Establish an SSH session to the target host as user `bandit19`:

```bash
$ ssh bandit19@bandit.labs.overthewire.org -p 2220
bandit19@bandit.labs.overthewire.org's password: [REDACTED]
bandit19@bandit:~$
```

## Recon

So the first command I ran for this level like in earlier levels is the list command to identify the files and directories located on the current directory and verify their ownership and permissions.

```bash
bandit19@bandit:~$ ls -la
total 36
drwxr-xr-x   2 root     root      4096 Jun 24 14:58 .
drwxr-xr-x 150 root     root      4096 Jun 24 15:02 ..
-rw-r--r--   1 root     root       220 Feb 13  2026 .bash_logout
-rw-r--r--   1 root     root      3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root     root       807 Feb 13  2026 .profile
-rwsr-x---   1 bandit20 bandit19 14880 Jun 24 14:58 bandit20-do
```

Upon scanning the current directory a `bandit20-do` file was discovered and its permissions have an `s` where the owner's execute bit would normally be located (`rws` instead of `rwx`), and the owner is `bandit20` and not `bandit19`. This means that the current user (`bandit19`) that I have access to does not own this file, but I can still execute it (group `bandit19` has `r-x`). Additionally, it's worth investigating the `s` in the owner permissions triplet as it is a setuid bit and it's present on a binary owned by a different, higher privileged user. I believe it to be a designed mechanism for "run this specific program with someone else's privileges," and the level objective confirms that's intentional here.

```bash
bandit19@bandit:~$ cat /etc/bandit_pass/bandit20
cat: /etc/bandit_pass/bandit20: Permission denied
```

I did this to confirm that I really don't have access to read the bandit password with bandit19 privileges. 

```bash
bandit19@bandit:~$ ./bandit20-do
Run a command as another user.
Example: ./bandit20-do whoami
```

I will now run the `whoami` command through it first (as the binary's own example suggested) to observe the response I was going to receive, rather than jumping straight to the password.
```bash 
bandit19@bandit:~$ ./bandit20-do whoami
bandit20
```

 The output stated that I was the user `bandit20`, so now I believe that if this binary lets me execute commands as the file's owner `bandit20` then I should be able to read files `bandit20` owns and will also be able to `cat` the `bandit20` password.

I also ran `id` through it specifically to _observe_ the privilege escalation happening. 

```bash
bandit19@bandit:~$ ./bandit20-do id
uid=11019(bandit19) gid=11019(bandit19) euid=11020(bandit20) groups=11019(bandit19)
```

The output is the whole lesson in one line: my real UID (`uid`) is still bandit19, but my **effective** UID (`euid`) during this command is bandit20 — proof the setuid mechanism is live and working exactly as designed.
## Solution

```bash
bandit19@bandit:~$ ./bandit20-do cat /etc/bandit_pass/bandit20
[REDACTED]
```

Since `bandit20-do` runs its argument as a command with `euid=bandit20`, passing `cat /etc/bandit_pass/bandit20` as that argument means the _kernel_ checks file permissions against the effective UID (bandit20) when `cat` opens the file — not against my real identity (bandit19). That's why this succeeds where the direct `cat` attempt in Recon failed.

## Verification

Authenticate as `bandit20` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit20@bandit.labs.overthewire.org -p 2220
bandit20@bandit.labs.overthewire.org's password: [REDACTED]
bandit20@bandit:~$ whoami
bandit20
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit20@bandit:~$`) confirms successful authentication as `bandit20`.

## Why This Works

Every Linux process carries both a _real_ UID (who invoked it) and an _effective_ UID (whose permissions the kernel checks for file access, and most other privilege decisions). Normally these match. The setuid bit tells the kernel: when this file is executed, set the new process's effective UID to the file _owner's_ UID, not the caller's — regardless of who ran it. Here, `bandit20-do` is owned by bandit20 and has setuid set, so no matter which permitted user executes it, the resulting process runs with `euid=bandit20`. The binary then does something entirely ordinary internally — it forks and execs whatever command you gave it as an argument — but that child process inherits the elevated effective UID, so every permission check it triggers (like opening `/etc/bandit_pass/bandit20`) is evaluated as if bandit20 were doing it. This is the identical mechanism `passwd` and `sudo` rely on; the only "vulnerability" here is that this particular setuid binary was intentionally designed to run an arbitrary caller-supplied command, which is a massive red flag in any real system.

## Key Takeaway

A setuid binary is only as safe as the code path it lets you reach — if it ever passes user-controlled input into something that executes a shell command (directly or via `system()`, `exec()`, etc.), it's a privilege-escalation vector by construction, not by accident. Real setuid binaries should do one narrow, fixed thing with elevated privileges and drop back to the real UID immediately after; they should never hand the caller a general-purpose "run anything as me" primitive like this level's toy example does.

## Password

`[REDACTED - see level for retrieval method]`


---
