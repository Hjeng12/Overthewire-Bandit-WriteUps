# Bandit Level 3 → 4

## Objective

The password for the next level is stored in a hidden file in the inhere directory.

## Concept Tested

Concept of hidden files (or dotfiles), which are files starting with a dot (.) character that are omitted by default file listing commands.

## Initial Access

Establish an SSH session to the target host as user `bandit3`:

```bash
$ ssh bandit3@bandit.labs.overthewire.org -p 2220
bandit3@bandit.labs.overthewire.org's password: [REDACTED]
bandit3@bandit:~$
```

## Recon

The list command was ran to reveal all the files  and directories (even hidden ones) located in the current directory and to identify ownership and POSIX mode bits.

```bash
bandit3@bandit:~$ ls -la
total 24
drwxr-xr-x   3 root root 4096 Jun 24 14:59 .
drwxr-xr-x 150 root root 4096 Jun 24 15:02 ..
-rw-r--r--   1 root root  220 Feb 13 12:16 .bash_logout
-rw-r--r--   1 root root 3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root root  807 Feb 13 12:16 .profile
drwxr-xr-x   2 root root 4096 Jun 24 14:59 inhere
```

Now that the `inhere` directory has been located, one will now navigate to the directory using the `cd` command in search for file that holds the password.

```bash 
bandit3@bandit:~$ cd inhere
bandit3@bandit:~/inhere$
```

*P.S. The `cd` command is short change directory.*

Now that we are in the `inhere` directory, the list command will be ran again to reveal all the files and directories in this directories.

```bash
bandit3@bandit:~/inhere$ ls -la
total 12
drwxr-xr-x 2 root    root    4096 Jun 24 14:59 .
drwxr-xr-x 3 root    root    4096 Jun 24 14:59 ..
-rw-r----- 1 bandit4 bandit3   33 Jun 24 14:59 ...Hiding-From-You
```

The output of `ls -la` command revealed the file `...Hiding-From-You`. This would've otherwise been hidden if the `-a` (`all`) flag wasn't used. It would've looked like this:

```bash 
bandit3@bandit:~/inhere$ ls -l
total 0
```


## Solution

To reveal the contents of the hidden file `...Hiding-From-You`, one would only need to `cat` the file like normal.

```bash
bandit3@bandit:~/inhere$ cat ...Hiding-From-You
[REDACTED]
```

## Verification

Authenticate as `bandit4` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit4@bandit.labs.overthewire.org -p 2220
bandit4@bandit.labs.overthewire.org's password: [REDACTED]
bandit4@bandit:~$ whoami
bandit4
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit4@bandit:~$`) confirms successful authentication as `bandit4`.

## Why This Works


On Unix-like systems, "hidden" files are not a distinct filesystem attribute (unlike Windows' hidden bit) — it's purely a display convention baked into tools like `ls`, `find` (with defaults), and file managers. The rule that makes this possible is simple: `ls` (without `-a`) hides any file whose name begins with a period (`.`). So a file like `my.file.txt` stays visible, since its first character is `m`, not `.` — but a file like `...Hiding-From-You` gets hidden, since its first character _is_ `.`.

This convention is an old Unix-like design choice meant to declutter directories and keep background configuration out of view (e.g., keeping files like `.bashrc` out of normal directory listings) — not to provide security. The file is fully readable, listable, and accessible to anyone with the right permissions; you just have to ask the tool to show it (e.g., `ls -a`).

## Key Takeaway

Never treat a leading dot as a security boundary — it's cosmetic, not access control. If you need to actually hide sensitive data, use real permissions (`chmod`) or encryption, not naming conventions.

## Password

`[REDACTED - see level for retrieval method]

---
