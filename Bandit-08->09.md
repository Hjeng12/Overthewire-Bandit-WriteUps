# Bandit Level 8 → 9

## Objective
 
The password for the next level is stored in the file `data.txt` and is the only line of text that occurs only once. 

## Concept Tested

Text processing and stream filtering in Linux, specifically using the `sort` and `uniq` commands via command-line piping to isolate unique data lines from large, duplicate-heavy datasets. 
## Initial Access

Establish an SSH session to the target host as user `bandit8`:

```bash
$ ssh bandit8@bandit.labs.overthewire.org -p 2220
bandit8@bandit.labs.overthewire.org's password: [REDACTED]
bandit8@bandit:~$
```

## Recon

The first command ran was the list command to identify the files located on the current directory and reveal the ownership and POSIX mode bits.

```bash
bandit8@bandit:~$ ls -la
total 56
drwxr-xr-x   2 root    root     4096 Jun 24 14:59 .
drwxr-xr-x 150 root    root     4096 Jun 24 15:02 ..
-rw-r--r--   1 root    root      220 Feb 13 12:16 .bash_logout
-rw-r--r--   1 root    root     3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root    root      807 Feb 13 12:16 .profile
-rw-r-----   1 bandit9 bandit8 33033 Jun 24 14:59 data.txt
```

Upon scanning the current directory the `data.txt` has been located. Now in accordance with the objective, one would need to locate the only line in the text file that is not duplicated. I decided to use the `sort` command for this challenge.

```bash
bandit8@bandit:~$ sort -r data.txt
ZpnAurXvto4z1V8ySf59BjY2Ieq2xa7y
ZpnAurXvto4z1V8ySf59BjY2Ieq2xa7y
ZpnAurXvto4z1V8ySf59BjY2Ieq2xa7y
...
```

`-r` reverses the text file while putting it in order from Z -> 0. I was able to find the code using this, but it took far too long.
I tried using the `sort -u` which sorts a text file and removes duplicate lines, printing only one copy of each unique line.

```bash
bandit8@bandit:~$ sort -u data.txt
0LTDNpAmqqfuE0FlE0ksGf6c0Kraspzs
1cKKjk7M0Pl2cPUbYgc9W4307bYC0ohF
1PesxCa7cihwvCvzBeKAcjKkjUwp7i2z
...
```

So as this also provided me with a multiple choices for passwords. This is not the correct choice.

## Solution

So to find the password for the challenge, one would have to `sort` the `data.txt` and then  use the `uniq` command to find the unique lines in the text file.

```bash
bandit8@bandit:~$ sort data.txt | uniq -u
[REDACTED]
```

- `sort` reorders lines alphabetically or numerically so matching lines sit next to each other.
- `uniq` filters out or isolates repeated lines, requiring sorted input to function accurately.

`uniq -u` alone won't catch non-adjacent duplicates so if you don't sort the text file, the duplications in the file won't properly  be caught and it will leave just the single one that exist.

## Verification

Authenticate as `bandit9` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit9@bandit.labs.overthewire.org -p 2220
bandit9@bandit.labs.overthewire.org's password: [REDACTED]
bandit9@bandit:~$ whoami
bandit9
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit9@bandit:~$`) confirms successful authentication as `bandit9`.

## Why This Works

`uniq` is intentionally a lightweight, streaming tool — it compares each line only to the line immediately before it, which is O(1) per line and requires no memory of lines seen earlier in the file. This makes it fast but means it can only detect adjacent repeats, not repeats scattered throughout an unsorted file. `sort` solves this limitation as a preprocessing step: by reordering the file so that all identical lines end up next to each other, it transforms a "duplicates anywhere in the file" problem into a "duplicates next to each other" problem, which is exactly what `uniq` is built to solve. This sort-then-uniq pattern is one of the most common idioms in Unix text processing for exactly this reason.
## Key Takeaway

Remember that `uniq` only catches adjacent duplicates — if you need whole-file deduplication or uniqueness detection on unsorted data, always pipe through `sort` first (`sort file | uniq` for dedup, `sort file | uniq -u` for lines with no duplicates, `sort file | uniq -d` for lines that do have duplicates).

## Password

`[REDACTED - see level for retrieval method]`

---
