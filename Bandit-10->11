# Bandit Level 10 → 11

## Objective

The password for the next level is stored in the file `data.txt`, which contains `base64` encoded data.

## Concept Tested

**Data decoding** and **text transformation**, specifically focusing on **Base64** **decoding**. This level teaches you how to translate a **Base64-encoded** text back into human-readable plain text.

## Initial Access

Establish an SSH session to the target host as user `bandit10`:

```bash
$ ssh bandit10@bandit.labs.overthewire.org -p 2220
bandit10@bandit.labs.overthewire.org's password: [REDACTED]
bandit10@bandit:~$
```

## Recon

The first command ran as always is the list command to identify the files stored on the current directory and to reveal the ownership and POSIX mode bits.

```bash
bandit10@bandit:~$ ls -la
total 24
drwxr-xr-x   2 root     root     4096 Jun 24 14:58 .
drwxr-xr-x 150 root     root     4096 Jun 24 15:02 ..
-rw-r--r--   1 root     root      220 Feb 13 12:16 .bash_logout
-rw-r--r--   1 root     root     3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root     root      807 Feb 13 12:16 .profile
-rw-r-----   1 bandit11 bandit10   69 Jun 24 14:58 data.txt
```

The `data.txt` file has been located so now I will check the file type and then `cat` the `data.txt` file to reveal the encoded password.

```bash
bandit10@bandit:~$ file data.txt
data.txt: ASCII text
bandit10@bandit:~$ cat data.txt
VGhlIHBhc3N3b3JkIGlzIHBZZk9ZNkh3VXNEajVyTDlVdnloVTdNQ212OHZONVJvCg==
``` 


## Solution

This challenge is fairly simple. All one has to do now is decode the `data.txt` file to reveal the password. Since the encoding format is known to be Base64, all that's left is to decode it.

```bash
bandit10@bandit:~$ base64 -d data.txt
The password is [REDACTED]
```

The `-d` (decode) tells base64 to reverse the encoding.
## Verification

Authenticate as `bandit11` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit11@bandit.labs.overthewire.org -p 2220
bandit11@bandit.labs.overthewire.org's password: [REDACTED]
bandit11@bandit:~$ whoami
bandit11
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit11@bandit:~$`) confirms successful authentication as `bandit11`.

## Why This Works

**Base64** takes arbitrary binary data and re-represents it using only 64 printable ASCII characters (A–Z, a–z, 0–9, +, /), so it can safely pass through systems that only handle text (email, URLs, JSON, etc.) without corruption. It does this by grouping the input into 3-byte (24-bit) chunks and re-slicing them into four 6-bit values, each mapped to one of the 64 printable characters. Critically, this mapping is a fixed, publicly known, one-to-one algorithm — there's no key involved — so decoding requires no secret, only knowledge of the standard base64 alphabet, which every base64 implementation ships with by definition.

## Key Takeaway

Never mistake encoding for encryption. If you see base64, hex, or URL-encoding "protecting" sensitive data, treat it as equivalent to plaintext — anyone with a terminal can reverse it instantly. Actual confidentiality requires real encryption with a secret key.

## Password

`[REDACTED - see level for retrieval method]`


---
