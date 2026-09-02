# Bandit Level 1 → 2

## Objective

The password for the next level is stored in a file called '-' located in the home directory.

## Concept Tested

Shell escaping and quoting, specifically handling whitespace/ spaces in filenames and command-line argument parsing.

## Initial Access

Establish an SSH session to the target host as user `bandit1`:

```bash
$ ssh bandit1@bandit.labs.overthewire.org -p 2220
bandit1@bandit.labs.overthewire.org's password: [REDACTED]
bandit1@bandit:~$
```

## Recon

The list command was used to inspect the home directory to ensure the '-' file was located there and to identify the ownership and POSIX mode bits:

```bash
bandit1@bandit:~$ ls -la
total 24
-rw-r-----   1 bandit2 bandit1   33 Jun 24 14:58 -
drwxr-xr-x   2 root    root    4096 Jun 24 14:58 .
drwxr-xr-x 150 root    root    4096 Jun 24 15:02 ..
-rw-r--r--   1 root    root     220 Feb 13 12:16 .bash_logout
-rw-r--r--   1 root    root    3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root    root     807 Feb 13 12:16 .profile
```

`ls -la` lists all the directory contents, including hidden files, along with the file ownership and permissions bits. The output shows the `-` file is owned by `bandit2` and group `bandit1`, with mode `0640` (`-rw-r-----`). World access is fully revoked (`---`), but members of the group `bandit1` hold read privileges (`r--`).

So now one would try to retrieve the password by displaying the contents of file `-` 

```bash
bandit1@bandit:~$ cat -
```

this fails due to the fact that `-` is recognized as a standard input instead of a filename.

## Solution

To read the file `-` one would have to input a command to look at the path of the file with using `.` to signify the current directory of `~` to signify the home directory which can be checked with `pwd`:

```bash
bandit1@bandit:~$ pwd
/home/bandit1
```

So to retrieve the password either one of these commands must be ran.

```bash
bandit1@bandit:~$ cat ~/-
[REDACTED]
```

OR

```bash
bandit1@bandit:~$ cat ./-
[Readacted]
```

## Verification

Authenticate as `bandit1` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit2@bandit.labs.overthewire.org -p 2220
bandit2@bandit.labs.overthewire.org's password: [REDACTED]
bandit2@bandit:~$ whoami
bandit2
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit2@bandit:~$`) confirms successful authentication as `bandit2`.

## Why This Works

Many Unix command-line tools (cat, cp, tar, etc.) follow a convention where a single `-` argument means "use stdin/stdout instead of a file." This is a _convention_, not a filesystem rule — the filesystem is perfectly happy to have a file literally named `-`. Prefixing the name with `./` (or giving a full path) changes the string passed to the program so it no longer matches the special-cased `-` token; it's now unambiguously a relative/absolute path, so the program does a normal file open instead of falling back to stdin.


## Key Takeaway

Never rely on filenames matching special CLI conventions (`-`, `--`, etc.) for security-relevant assumptions — and as a general practice, always reference files via `./filename` or absolute paths in scripts/automation to avoid ambiguous parsing.

## Password

`[REDACTED - see level for retrieval method]`

---
