# Bandit Level 17 → 18

## Objective

There are 2 files in the home directory: `passwords.old` and `passwords.new`. The password for the next level is in `passwords.new` and is the only line that has been changed between `passwords.old` and `passwords.new`

**NOTE: if you have solved this level and see ‘Byebye!’ when trying to log into bandit18, this is related to the next level, bandit19**
## Concept Tested

Line-by-line file comparison with `diff` using it as a signal-extraction tool to isolate a single changed value out of dozens of near-identical lines, instead of manually eyeballing two files. 

## Initial Access

Establish an SSH session to the target host as user `bandit17`:

```bash
$ ssh bandit17@bandit.labs.overthewire.org -p 2220
bandit17@bandit.labs.overthewire.org's password: [REDACTED]
bandit17@bandit:~$
```

## Recon

As in the earlier stages of these challenges. I will start with the list command to locate the files we're dealing with in accordance with the objective and identify their permissions. 

```bash
bandit17@bandit:~$ ls -la
total 36
drwxr-xr-x   3 root     root     4096 Jun 24 14:58 .
drwxr-xr-x 150 root     root     4096 Jun 24 15:02 ..
-rw-r-----   1 bandit17 bandit17   33 Jun 24 14:58 .bandit16.password
-rw-r--r--   1 root     root      220 Feb 13  2026 .bash_logout
-rw-r--r--   1 root     root     3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root     root      807 Feb 13  2026 .profile
drwxr-xr-x   2 root     root     4096 Jun 24 14:58 .ssh
-rw-r-----   1 bandit18 bandit17 3300 Jun 24 14:58 passwords.new
-rw-r-----   1 bandit18 bandit17 3300 Jun 24 14:58 passwords.old
```

Seeing that the files `passwords.old` and `passwords.new` are located in the current directory. I will now check the number of lines both files have to ensure that they are the same.

```bash
bandit17@bandit:~$ wc -l passwords.old passwords.new
 100 passwords.old
 100 passwords.new
 200 total
```

Now that it has been verified that both files have the same amount of lines because if they didn't, a simple line-diff could get confusing (insertions/deletions shift line numbers around, not just a value change), so this told me I could expect a clean single-line substitution, not an insert/delete.

## Solution

```bash
bandit17@bandit:~$ diff passwords.old passwords.new
42c42
< [REDACTED]
---
> [REDACTED]
```

- `diff` walks both files (`passwords.old` and `passwords.new`) line-by-line and reports only where they diverge. 
- `42c42` means the line 42 was changed in both files (not added or removed elsewhere) which is consistent with what the objective promised about it being only one line that was changed.
- `<` indicates that the line is the old content (from `passwords.old`).
- `>` indicates the new content (from `passwords.new`).

## Verification

Authenticate as `bandit18` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit18@bandit.labs.overthewire.org -p 2220
bandit18@bandit.labs.overthewire.org's password: [REDACTED]

ByeBye!
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Seeing as the `ByeBye!` appear I now have to complete `bandit19`, as stated in the objective.

## Why This Works

`diff` uses an algorithm based on finding the longest common subsequence between two files and reports the minimal edit — the smallest set of line insertions/deletions/substitutions needed to turn one file into the other. Because both files are otherwise identical, that minimal edit collapses down to exactly one line, which is precisely why the level is solvable in a single command instead of needing a manual scan through potentially hundreds of similar-looking password strings. This is the same underlying algorithm (and the same reason) `git diff` can show you "what changed" in a commit instead of dumping the entire file.

## Key Takeaway

When you need to find what changed between two versions of something — a config file, a password list, a deployed script — reach for `diff` (or `git diff`) instead of manual comparison. It's not just faster; it's far less error-prone once files get long enough that a one-character difference is invisible to the human eye.

## Password

`[REDACTED - see level for retrieval method]`


---
