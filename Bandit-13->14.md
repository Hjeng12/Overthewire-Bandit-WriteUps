# Bandit Level 13 → 14

### Objective

The password for the next level is stored in `/etc/bandit_pass/bandit14` and can only be read by user `bandit14`. For this level, you don't get the next password, but you get a private SSH key that can be used to log into the next level. Look at the commands that logged you into previous bandit levels, and find out how to use the key for this level.

### Concept Tested

SSH key-based authentication (specifically using an unencrypted SSH private key identity file) and client-side key permission enforcement (`chmod`).

### Initial Access

Establish an SSH session to the target host as user `bandit13`:

```bash
$ ssh bandit13@bandit.labs.overthewire.org -p 2220
bandit13@bandit.labs.overthewire.org's password: [REDACTED]
bandit13@bandit:~$
```

### Recon

I ran `ls -la` to locate the SSH private key mentioned in the level objective and inspect its permissions.

```bash
bandit13@bandit:~$ ls -la
total 28
drwxr-xr-x   2 root     root     4096 Jun 24 14:59 .
drwxr-xr-x 150 root     root     4096 Jun 24 15:02 ..
-rw-r--r--   1 root     root      220 Feb 13 12:16 .bash_logout
-rw-r--r--   1 root     root     3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root     root      807 Feb 13 12:16 .profile
-rw-r-----   1 bandit14 bandit13  467 Jun 24 14:59 HINT
-rw-r-----   1 bandit14 bandit13 2602 Jun 24 14:59 sshkey.private
```

Scanning the directory confirmed that `sshkey.private` exists and is readable by group `bandit13`. This confirmed the level is about leveraging key-based authentication rather than password discovery.

Next, I ran the `file` command to verify that the file type matches what `ssh -i` expects:

```bash
bandit13@bandit:~$ file sshkey.private
sshkey.private: OpenSSH private key
```

To use the key from my local machine, I outputted the contents of `sshkey.private` and saved them to a local file named `key`:

```bash
bandit13@bandit:~$ cat sshkey.private
-----BEGIN OPENSSH PRIVATE KEY-----
...
...
...
-----END OPENSSH PRIVATE KEY-----

bandit13@bandit:~$ exit
logout
Connection to bandit.labs.overthewire.org closed.

[Laptop-3344]:~$ nano key
```

### Solution

After pasting the key into the local `key` file, I attempted to log into `bandit14` and encountered an error:

```bash
[Laptop-3344]:~$ ssh -i key bandit14@bandit.labs.overthewire.org -p 2220

            More information on http://www.overthewire.org/wargames

backend: gibson-0
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0640 for 'key' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "key": bad permissions
bandit14@bandit.labs.overthewire.org's password:
```

Attempting to connect directly with the saved key triggers OpenSSH's client-side security checks, which reject identity files accessible by other users.

```bash
[Laptop-3344]:~$ chmod 700 key
[Laptop-3344]:~$ ssh -i key bandit14@bandit.labs.overthewire.org -p 2220
...
  Enjoy your stay!

bandit14@bandit:~$ cat /etc/bandit_pass/bandit14
[REDACTED]
```

- Run `chmod 700 key` (or `chmod 600 key`) to restrict the file's read, write, and execute permissions exclusively to the owner, stripping group and world access.
- Connect to the server as user `bandit14` on port `2220` using `ssh -i key bandit14@bandit.labs.overthewire.org -p 2220`.
- Once authenticated as `bandit14`, run `cat /etc/bandit_pass/bandit14` to retrieve the password string for the next level. 

### Verification

Authenticate as `bandit14` using the retrieved password to confirm access:

```bash
[Laptop-3344]:~$ ssh bandit14@bandit.labs.overthewire.org -p 2220
bandit14@bandit.labs.overthewire.org's password: [REDACTED]
bandit14@bandit:~$ whoami
bandit14
```

Initiating a new SSH session and supplying the retrieved password grants access. Running `whoami` and observing the updated shell prompt (`bandit14@bandit:~$`) confirms successful authentication.

### Why This Works

OpenSSH client binaries intentionally fail closed when reading key files accessible by other local security principals (group- or world-readable). This client-side guardrail prevents accidental exposure of high-entropy private keys on multi-tenant systems because without this protection a co-tenant with a shell on the same machine could otherwise copy an over-permissive private key and impersonate its owner on any remote system that trusts that key, entirely independent of the remote server's own authentication. Once permissions are restricted using `chmod 700` or `600`, OpenSSH processes the private key, completes the SSH handshake using asymmetric cryptography (without prompting for a password), and spawns a shell as `bandit14`. From there, standard POSIX DAC permissions allow `bandit14` to read `/etc/bandit_pass/bandit14`.

### Key Takeaway

Always enforce strict file permissions (`600` or `700`) on SSH private keys and TLS certificates. Never leave key material group-readable or world-readable on non-isolated systems.

### Password

`[REDACTED - see level for retrieval method]`

---
