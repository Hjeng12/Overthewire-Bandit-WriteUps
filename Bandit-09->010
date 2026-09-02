# Bandit Level 9 → 10

## Objective

The password for the next level is stored in the file `data.txt` in one of the few human-readable strings, preceded by several '=' characters.

## Concept Tested

Binary data analysis and data filtering. It shows you know to extract readable `ASCII strings` from a file filled with binary or non-printable garbage data using tools like `strings` and `grep`.
## Initial Access

Establish an SSH session to the target host as user `bandit9`:

```bash
$ ssh bandit9@bandit.labs.overthewire.org -p 2220
bandit9@bandit.labs.overthewire.org's password: [REDACTED]
bandit9@bandit:~$
```

## Recon

The first command ran was the list command to identify the files located on the current directory and reveal the ownership and POSIX mode bits.

```bash
bandit9@bandit:~$ ls -la
total 40
drwxr-xr-x   2 root     root     4096 Jun 24 14:58 .
drwxr-xr-x 150 root     root     4096 Jun 24 15:02 ..
-rw-r--r--   1 root     root      220 Feb 13 12:16 .bash_logout
-rw-r--r--   1 root     root     3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root     root      807 Feb 13 12:16 .profile
-rw-r-----   1 bandit10 bandit9 19382 Jun 24 14:58 data.txt
```

Now that the scan was conducted the `data.txt` file has been discovered. So now I will use the `file` command to confirm the file type and then `cat` the file to identify what other information I can get from the file.

```bash
bandit9@bandit:~$ file data.txt
data.txt: data
bandit9@bandit:~$ cat data.txt
rBj��W��-��[�B�*�       }v 9m��\�Y��f�A��y�~.[o���#��o��N�T�ҝ��D�;��`t�&>�P�␦���h��Văe��N�      ɛYL;E���]��y�$* �5�}�e-l�!����ΐ�N�����Rz­�zJP�0�k��퍁���0��ݵ�ơ��NEH
       xߙ�e��\�/␦r�E�_��Hʈ��˵�␦Z��O�    cg�
       ...
```

Now that the files have been revealed, one can discern that it is not human-readable. As stated by the objective, to obtain the password the text output has to be formatted in a way where the human readable bits can be read when the proceeding after the equal sign `=`. So the `strings` command will be used.

```bash
bandit9@bandit:~$ strings data.txt
v 9m
~.[o
...
========== the
%$;1
ho>n
...
========== password
fry{(
xJ3[Hg
na):7
...
Y========== is
-\[sE-u
s?
j0Fy
...
========== [REDACTED]
```

## Solution

The password was already visible in the raw `strings` output above, but filtering with `grep` isolates it cleanly so it can be more easily readable, with using the `grep` command.

```bash
bandit9@bandit:~$ strings data.txt | grep '==='
========== the
========== password
Y========== is
========== [REDACTED]
```

- `strings data.txt` — scans the binary file and extracts only sequences of printable characters (by default, runs of 4 or more printable ASCII characters), discarding everything that isn't human-readable text. This is the standard first step when trying to pull readable content out of an otherwise binary blob.
- `grep '==='` — filters that extracted text down to lines containing an equals sign, since the level description tells us the password is preceded by several `=` characters, which makes for a distinctive and rare pattern in this file compared to other extracted strings.

## Verification

Authenticate as `bandit10` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit10@bandit.labs.overthewire.org -p 2220
bandit10@bandit.labs.overthewire.org's password: [REDACTED]
bandit10@bandit:~$ whoami
bandit10
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit10@bandit:~$`) confirms successful authentication as `bandit10`.

## Why This Works

Binary files are just streams of bytes that don't map to meaningful text under most encodings — but they can still _contain_ embedded printable-ASCII substrings (strings that were originally stored as text within a larger binary structure). `strings` works by scanning byte-by-byte and reporting any contiguous run of bytes that fall within the printable ASCII range and meet a minimum length threshold, ignoring everything else. This lets you recover readable fragments without needing to understand or parse the binary format itself. Piping to `grep` on top of that narrows a potentially large set of extracted strings down to the one matching a known distinguishing feature, exactly like using `grep` on any large text output.

## Key Takeaway

When you're facing a file that isn't plain text, don't try to `cat` it blindly (it can be unreadable or even affect your terminal) — use `file` to confirm the type first, then reach for `strings` to safely extract any embedded readable text, and chain it with `grep` when you know something distinctive to search for.

## Password

`[REDACTED - see level for retrieval method]`

---
