# Bandit Level 7 → 8

## Objective

The password for the next level is stored in the file `data.txt` next to the word `millionth` 

## Concept Tested

Command-line text searching and data filtering using the `grep` utility to parse large files.

## Initial Access

Establish an SSH session to the target host as user `bandit7`:

```bash
$ ssh bandit7@bandit.labs.overthewire.org -p 2220
bandit7@bandit.labs.overthewire.org's password: [REDACTED]
bandit7@bandit:~$
```

## Recon

The first command ran was the list command to identify the files located on the current directory and reveal the ownership and POSIX mode bits.

```bash
bandit7@bandit:~$ ls -la
total 4108
drwxr-xr-x   2 root    root       4096 Jun 24 14:59 .
drwxr-xr-x 150 root    root       4096 Jun 24 15:02 ..
-rw-r--r--   1 root    root        220 Feb 13 12:16 .bash_logout
-rw-r--r--   1 root    root       3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root    root        807 Feb 13 12:16 .profile
-rw-r-----   1 bandit8 bandit7 4184396 Jun 24 14:59 data.txt
```

Upon scanning the current directory the `data.txt` file was revealed. So now one would then `cat` the file or use `head` command to reveal a few lines of the data on the file since it has been identified that the file is `4184396` bytes and may not want that much data dumped to the terminal at once with `cat`.

```bash
bandit7@bandit:~$ cat data.txt
bracken's w4m77B7X7GYsOmJz0t0F2GwggoBY9gnH
packing n5xbRJzpSDgdC6c9q967mzFhM2NrtsRc
weighty Yy2wte9aVxCwRaD7OE25OoSaXgCh6DEp
...
```

Or a more cleaner option:

```bash
bandit7@bandit:~$ head -n 3 data.txt
bracken's w4m77B7X7GYsOmJz0t0F2GwggoBY9gnH
packing n5xbRJzpSDgdC6c9q967mzFhM2NrtsRc
weighty Yy2wte9aVxCwRaD7OE25OoSaXgCh6DEp
```

- `head` to view the beginning of the file.
- `-n 3` is the modifier that specifies the number of lines in the file to display.
- `data.txt` is the name of the file.

But when `data.txt` was displayed with `cat`, it revealed that the `data.txt` file contains many words with what it seems to be password to go along with them. So now one would only need to locate the password by finding the '`millionth`' word as tasked by the objective.

## Solution

So to reveal the password in the `data.txt` file, one would need to `grep` the file and state the word in the file you are looking for.

```bash
bandit7@bandit:~$ grep 'millionth' data.txt
millionth       [REDACTED]
```

- `grep` is a tool in Linux that searches for specific text patterns or words within files.

## Verification

Authenticate as `bandit8` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit8@bandit.labs.overthewire.org -p 2220
bandit8@bandit.labs.overthewire.org's password: [REDACTED]
bandit8@bandit:~$ whoami
bandit8
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit8@bandit:~$`) confirms successful authentication as `bandit8`.

## Why This Works

`grep` implements efficient line-by-line pattern matching (using regex or literal string matching depending on flags) without needing to load or display the entire file — it just streams through the input and reports matches. This makes it possible to pull a needle out of an arbitrarily large haystack in roughly linear time, which is exactly why the level hands you a huge file and a keyword instead of a small one: it's demonstrating that grep, not manual reading, is the right tool once data exceeds human-scannable size.

## Key Takeaway

When you know _something_ distinctive about the line you need (a keyword, a pattern, a prefix), reach for `grep` (or `grep -E`/`egrep` for more complex patterns) instead of paging through large files manually — it scales to files of any size and is one of the most fundamental tools for working with text data on Unix systems.
## Password

`[REDACTED - see level for retrieval method]`

---
