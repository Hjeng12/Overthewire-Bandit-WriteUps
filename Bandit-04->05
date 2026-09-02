# Bandit Level 4 → 5

## Objective

The password for the next level is stored in the only human-readable file in the `inhere` directory.

## Concept Tested

File type identification and data inspection, specifically using the `file` command and distinguishing `ASCII text` from `binary data`.

## Initial Access

Establish an SSH session to the target host as user `bandit4`:

```bash
$ ssh bandit4@bandit.labs.overthewire.org -p 2220
bandit4@bandit.labs.overthewire.org's password: [REDACTED]
bandit4@bandit:~$
```

## Recon

This challenge was initiated with the list command to locate and verify that the `inhere` directory was in the current directory and to identify ownership and POSIX mode bits.

```bash
bandit4@bandit:~$ ls -la
total 24
drwxr-xr-x   3 root root 4096 Jun 24 14:59 .
drwxr-xr-x 150 root root 4096 Jun 24 15:02 ..
-rw-r--r--   1 root root  220 Feb 13 12:16 .bash_logout
-rw-r--r--   1 root root 3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root root  807 Feb 13 12:16 .profile
drwxr-xr-x   2 root root 4096 Jun 24 14:59 inhere
```

Now that the `inhere` directory has been identified, one only need to navigate to it using the change directory command. Additionally, the list command will be ran again to identify what files are stored in  the directory.

```bash
bandit4@bandit:~$ cd inhere
bandit4@bandit:~/inhere$ ls -la
total 48
-rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file00
-rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file01
-rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file02
-rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file03
-rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file04
-rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file05
-rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file06
-rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file07
-rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file08
-rw-r----- 1 bandit5 bandit4   33 Jun 24 14:59 -file09
drwxr-xr-x 2 root    root    4096 Jun 24 14:59 .
drwxr-xr-x 3 root    root    4096 Jun 24 14:59 ..
```

Through the list command, it has been identified that 10 files exist within the `inhere` directory (`-file00 to -file09`). Additionally, the all the files have a dash (`-`) in the beginning of their names.

## Solution

To reveal which file has the password one has to identify which file is human-readable. 

```bash
bandit4@bandit:~/inhere$ find . -type f -exec file {} \; | grep 'ASCII text'
./-file06: Non-ISO extended-ASCII text, with NEL line terminators
./-file07: ASCII text
```

- `find . -type f` *recursively lists every regular file in the current directory.*
- `exec file {} \;` *runs the `file` command on each result - `file` inspects the actual byte content (magic numbers) of a file to determine its type, regardless of its name.*
- *Piping (`|`) to* `grep 'ASCII text` *filters the output down to just the file(s) that are in plain human-readable text, discarding binary/data/empty results.*

Two files were flagged `-file06` and `-file07`, the file with the password is `-file07` and the reason that it is not `-file06` is because `-file07` is an unqualified `ASCII text` whereas `-file06` is `Non-ISO extended-ASCII text` meaning it contains non-standard/extended-ASCII bytes, for example not pure 7-bit ASCII.
Now that the `ASCII text` file has been located. One can now `cat` the file to reveal the password.

```bash
bandit4@bandit:~/inhere$ cat ./-file07
[REDACTED]
```

## Verification

Authenticate as `bandit5` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit5@bandit.labs.overthewire.org -p 2220
bandit5@bandit.labs.overthewire.org's password: [REDACTED]
bandit5@bandit:~$ whoami
bandit5
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit5@bandit:~$`) confirms successful authentication as `bandit5`.

## Why This Works

The `file` command doesn't guess based on extension or name — it reads the first chunk of bytes in a file and compares them against a database of known "magic numbers" / signatures (defined in `/usr/share/file/magic` or similar) to identify the format: ELF binary, ASCII text, JPEG, gzip, etc. This means file type identification is content-driven and can't be fooled by renaming a file, which is exactly why this level scatters decoy files with meaningless names — you're forced to inspect substance over label. Additionally, a simpler command like`file *` or `file -file00 -file01 -file02 ..` was not used because each expanded filename starts with a dash (`-`), `file` (a standard getopt-style parser like `cat`) would try to interpret `-file00`, `-file01`, etc. as unrecognized option flags rather than filenames, causing it to error out or behave unexpectedly instead of inspecting the files — the exact same leading-dash parsing trap as `cat -` and `--spaces in this filename--` in earlier levels.

## Key Takeaway

Never trust a filename or extension as a guarantee of content or safety (this is a classic malware trick — `invoice.pdf.exe`) — when it matters, verify actual file type/content programmatically with tools like `file`, checksums, or proper content validation.

## Password

`[REDACTED - see level for retrieval method]

---
