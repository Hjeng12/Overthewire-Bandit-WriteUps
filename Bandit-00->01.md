# Bandit Level 0 → 1

## Objective

The password for the next level is stored in a file called `readme` located in the home directory. Use this password to log into `bandit1` using SSH.

## Concept Tested

Linux Discretionary Access Control (DAC) group permissions — specifically, how group membership grants file access independent of file ownership.

## Initial Access

Establish an SSH session to the target host as user `bandit0`:

```bash
$ ssh bandit0@bandit.labs.overthewire.org -p 2220
bandit0@bandit.labs.overthewire.org's password: bandit0
bandit0@bandit:~$
```

## Recon

Inspect the home directory to locate target files and examine their ownership and POSIX mode bits:

```bash
bandit0@bandit:~$ ls -la
total 24
drwxr-xr-x  2 root    root    4096 Oct  5 06:10 .
drwxr-xr-x 41 root    root    4096 Oct  5 06:10 ..
-rw-r-----  1 bandit1 bandit0   33 Oct  5 06:10 readme
```

`ls -la` lists all directory contents, including hidden files, along with file ownership and permission bits. The output shows `readme` is owned by user `bandit1` and group `bandit0`, with mode `0640` (`-rw-r-----`). World access is fully revoked (`---`), but members of group `bandit0` hold read privilege (`r--`).

## Solution

Read the target file to retrieve the password for the `bandit1` account:

```bash
bandit0@bandit:~$ cat readme
```

`cat` streams the file's contents to stdout, exposing the plaintext password.

## Verification

Authenticate as `bandit1` using the retrieved password:

```bash
bandit0@bandit:~$ ssh bandit1@bandit.labs.overthewire.org -p 2220
bandit1@bandit.labs.overthewire.org's password: [REDACTED]
bandit1@bandit:~$ whoami
bandit1
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit1@bandit:~$`) confirms successful authentication as `bandit1`.

## Why This Works

Linux DAC permission checks are evaluated in a fixed order: owner match, then group match, then "other." When `cat` opens `/home/bandit0/readme`, the kernel first checks whether the process's effective UID matches the file's owner UID — it doesn't, since the file is owned by `bandit1`. The kernel then checks whether the process's effective GID matches the file's group — it does, since `bandit0`'s primary group is `bandit0`. Because the group check succeeds, the kernel applies the group permission bits (`r--`) and grants read access, without ever needing owner-level permission.

Separately, SSH authentication here uses direct password auth rather than key-based auth — acceptable for a training environment like this one, but on a real system password auth over SSH is generally considered weaker than key-based auth (susceptible to brute-force/credential-stuffing in a way key auth is not).

## Key Takeaway

Group-based access control means a user's access to a file isn't just about who owns it — it's about every group that user belongs to. This makes group membership itself a security-relevant configuration: adding a user to a group silently expands their read/write access to every file that group owns, so group membership needs the same scrutiny as direct file ownership. Separately, plaintext credentials should never sit in flat files — even under strict permissions, files like this should be short-lived (e.g. deleted post-boot) or replaced with a proper secrets mechanism.

## Password

`[REDACTED - see level for retrieval method]`

---
