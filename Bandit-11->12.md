# Bandit Level 11 → 12

## Objective

The password for the next level is stored in the file `data.txt`, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions.

## Concept Tested

The ROT13 substitution cipher (a basic cryptographic concept where letters are shifted by 13 positions) and how to process it in the shell using the Linux `tr` (translate) command.

## Initial Access

Establish an SSH session to the target host as user `bandit11`:

```bash
$ ssh bandit11@bandit.labs.overthewire.org -p 2220
bandit11@bandit.labs.overthewire.org's password: [REDACTED]
bandit11@bandit:~$
```

## Recon

The list command is prompted to ensure the `data.txt` file is located in the current directory.

```bash
bandit11@bandit:~$ ls -la
total 24
drwxr-xr-x   2 root     root     4096 Jun 24 14:58 .
drwxr-xr-x 150 root     root     4096 Jun 24 15:02 ..
-rw-r--r--   1 root     root      220 Feb 13 12:16 .bash_logout
-rw-r--r--   1 root     root     3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root     root      807 Feb 13 12:16 .profile
-rw-r-----   1 bandit12 bandit11   49 Jun 24 14:58 data.txt
```

Now that the `data.txt` file has been confirmed to reside in the current directory, the next thing to do is verify the type of the file `data.txt` is and `cat` it to reveal what contents it holds.

```bash
bandit11@bandit:~$ file data.txt
data.txt: ASCII text
bandit11@bandit:~$ cat data.txt
Gur cnffjbeq vf TEBbmJCB8DlA0zTewHxVQ0JPLxMvDkeA
```

Upon revealing the contents of the file, the output looks like jumbled letters to hide the password. But since the objective stated that the text is shifted 13 positions, it means that this is a Caesar cipher. Additionally it has been confirmed through the `file` command that it is plain text and not binary, ruling out the need to do binary encoding first.

## Solution

```bash
bandit11@bandit:~$ tr 'A-Za-z' 'N-ZA-Mn-za-m' < data.txt
The password is [REDACTED]
```

`tr` performs character-by-character transliteration: it maps each character in the first set (`A-Za-z`) to the corresponding character in the second set (`N-ZA-Mn-za-m`). That second set is the alphabet rotated by 13 letters, so every character in the input gets shifted 13 positions forward (wrapping around), which both encodes and decodes ROT13 — since 13 is exactly half of 26, applying it twice returns the original text.

## Verification

Authenticate as `bandit12` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit12@bandit.labs.overthewire.org -p 2220
bandit12@bandit.labs.overthewire.org's password: [REDACTED]
bandit12@bandit:~$ whoami
bandit12
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit12@bandit:~$`) confirms successful authentication as `bandit12`.

## Why This Works

ROT13 is a Caesar cipher with a fixed shift of 13. Because the English alphabet has 26 letters, shifting by 13 is its own inverse — encrypting and decrypting are the identical operation. This makes ROT13 trivial to break: there's no key to guess (the "key" is fixed and public), and even without knowing it's ROT13 specifically, a Caesar cipher only has 25 possible shifts, so brute-forcing takes seconds. It was never intended as real security — historically it's used to obscure spoilers or punchlines from casual glance, not to resist any deliberate attempt at reading it.

## Key Takeaway

Substitution ciphers with small, fixed, or guessable key spaces (Caesar/ROT13, XOR with a short repeating key, etc.) offer effectively zero real-world protection. If a junior dev reaches for something like this to "hide" sensitive data, treat it the same as storing it in plaintext.

## Password

`[REDACTED - see level for retrieval method]`


---
