# Bandit Level 27 → 28

## Objective

There is a git repository at `ssh://bandit27-git@bandit.labs.overthewire.org/home/bandit27-git/repo` via the port `2220`. The password for the user `bandit27-git` is the same as for the user `bandit27`.

From your local machine (not the OverTheWire machine!), clone the repository and find the password for the next level. This needs git installed locally on your machine.

## Concept Tested

Git-shell as a restricted login shell (a real-world pattern used to give an account git-only access with no general command execution), and basic `git clone` over SSH as a legitimate, non-exploit-based way to read content served by a special-purpose account.

## Initial Access

Establish an SSH session to the target host as user `bandit27`:

```bash
$ ssh bandit27@bandit.labs.overthewire.org -p 2220
bandit27@bandit.labs.overthewire.org's password: [REDACTED]
bandit27@bandit:~$
```

## Recon

Based on what the objective stated, this challenge is based around cloning a `git` repository. So the first thing I'll do is create a writable directory for the cloned `git` tree.

```bash
[Laptop-3344]:~$ mkdir /tmp/clonedgit
[Laptop-3344]:~$ cd /tmp/clonedgit
[Laptop-3344]:/tmp/clonedgit$
```

The home directory in some Bandit levels, has permission quirks (and cloning directly under a restricted home path is a known source of `.ssh` directory creation failures), so to get around that, all you need to do is listen to the objective and do the cloning on your local machine.

```bash
[Laptop-3344]:/tmp/clonedgit$ git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/re
po
[Enter password]
```

This is the exact URL given in the level description, no guessing required. I ran it as a normal `git clone`, deliberately not trying to `ssh` into `bandit27-git` directly for an interactive shell, since the level goal frames this explicitly as a git operation, not a shell-access one.
I was then prompted for `bandit27-git@bandit.labs.overthewire.org`'s password (same as `bandit27`'s, per the level description). 

```bash 
[Laptop-3344]:/tmp/clonedgit$ ls -la
total 0
drwxrwxrwx 3 hjeng hjeng  60 Aug 31 23:59 .
drwxrwxrwt 8 root  root  160 Sep  1 00:02 ..
drwxr-xr-x 3 hjeng hjeng  80 Aug 31 23:59 repo
[Laptop-3344]:/tmp/clonedgit$ cd repo
[Laptop-3344]:/tmp/clonedgit/repo$ ls -la
total 4
drwxr-xr-x 3 Laptop-3344 Laptop-3344  80 Aug 31 23:59 .
drwxrwxrwx 3 Laptop-3344 Laptop-3344  60 Aug 31 23:59 ..
drwxr-xr-x 7 Laptop-3344 Laptop-3344 240 Aug 31 23:59 .git
-rw-r--r-- 1 Laptop-3344 Laptop-3344  68 Aug 31 23:59 README
```

After succeeding in cloning the small repository with a `.git` directory. I `cd` into the repo then ran the list command to identify the files stored in it. Upon scan it has been identified that a `README` file found. This confirms that this account really is scoped to git operations specifically, since a normal SSH login attempt to it would likely be rejected or dropped into a restricted `git-shell` that refuses interactive commands.

## Solution

To complete this challenge all I need to do is `cat` the `README` file. No exploitation needed beyond the clone itself. `git clone` fully materializes the repository's tracked files locally, including `README`, which the repo's maintainer (presumably an admin script) placed there directly containing the password in plaintext.

```bash 
[Laptop-3344]:/tmp/clonedgit/repo$ cat README
The password to the next level is: [REDACTED]
```


## Verification

Authenticate as `bandit28` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit28@bandit.labs.overthewire.org -p 2220
bandit28@bandit.labs.overthewire.org's password: [REDACTED]
bandit28@bandit:~$ whoami
bandit28
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit28@bandit:~$`) confirms successful authentication as `bandit28`.

## Why This Works

I think that the `bandit27-git`'s login shell is set to `git-shell` (or something equivalent), a purpose-built restricted shell shipped with git itself. Its entire function is to accept a small allowlist of `git` protocol operations (`git-upload-pack`, `git-receive-pack`, etc.) over SSH and refuse everything else, including generic command execution. That's the correct way to expose "let people fetch this repo over SSH" without handing out a real shell. This is a much safer version than the informal restricted-shell attempts seen in Levels 25/26. There's no vulnerability being exploited here at all: the level's "solution" is simply that the account was designed to let anyone with its credentials clone the repo, and the repo's content (a `README` with the password) was placed there intentionally as the next step's reward.

## Key Takeaway

`git-shell` is the right tool when you genuinely need to expose git-only SSH access to an account. It's a well-tested, purpose-built restriction, unlike rolling your own wrapper script (which is exactly what went wrong in levels 25/26). The broader lesson: not every level is an exploit, sometimes the "vulnerability" is just that a repository's access controls were correctly scoped to git-only from the start, and reading its tracked files was the intended, legitimate use case all along.

## Password

`[REDACTED - see level for retrieval method]`


---
