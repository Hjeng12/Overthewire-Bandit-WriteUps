# Bandit Level 14 → 15

## Objective

The password for the next level can be retrieved by submitting the password of the current level to `port 30000` on localhost.

## Concept Tested

Raw TCP interaction with a service using `netcat` for example: manually speaking a plaintext protocol to a listening port instead of going through a purpose-built client.

## Initial Access

Establish an SSH session to the target host as user `bandit14`:

```bash
$ ssh bandit14@bandit.labs.overthewire.org -p 2220
bandit14@bandit.labs.overthewire.org's password: [REDACTED]
bandit14@bandit:~$
```

## Recon

So in accordance with the objective I need just retrieve the password from the current directory and submit it to `port 30000` on localhost to get the actual password so the first command I ran was the list command to identify the files currently available on the current directory and to inspect its permissions.

```bash
bandit14@bandit:~$ ls -la
total 24
drwxr-xr-x   3 root root 4096 Jun 24 14:58 .
drwxr-xr-x 150 root root 4096 Jun 24 15:02 ..
-rw-r--r--   1 root root  220 Feb 13  2026 .bash_logout
-rw-r--r--   1 root root 3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root root  807 Feb 13  2026 .profile
drwxr-xr-x   2 root root 4096 Jun 24 14:58 .ssh
```

Noticing by the d in `drwxr-xr-x` of `.ssh`, it can be confirmed that it is a directory. So I decided to navigate to that directory as it was not a directory available in the other bandit levels. Additionally, after navigating to this directory I'll run the list command again to reveal its contents.

```bash
bandit14@bandit:~$ cd .ssh/
bandit14@bandit:~/.ssh$ ls -la
total 12
drwxr-xr-x 2 root     root     4096 Jun 24 14:58 .
drwxr-xr-x 3 root     root     4096 Jun 24 14:58 ..
-rw-r----- 1 bandit14 bandit14  568 Jun 24 14:58 authorized_keys
```

Now it is revealed that there is a readable file called `authorized_keys`. I then `cat` and ran the `file` command on the file.

```bash
bandit14@bandit:~/.ssh$ file authorized_keys
authorized_keys: OpenSSH RSA public key
bandit14@bandit:~/.ssh$ cat authorized_keys
ssh-rsa ...
```

After seeing that the actual password is not located in this directory meaning this turned out to be a red herring for this particular level. I tried to `cat` the password from the `bandit_pass` directory to see if that works. 

```bash
bandit14@bandit:~$ cat /etc/bandit_pass/bandit14
REDACTED
```

Now that I have retrieved the password of the current user `bandit14` it's time to send it to localhost `port 30000`.

## Solution

At first I tried initiating the state of the terminal listening for the password.

```bash
bandit14@bandit:~/.ssh$ nc localhost 30000
cat /etc/bandit_pass/bandit14
Wrong! Please enter the correct current password.
```

It didn't work, and I assume it didn't work because the `cat /etc/bandit_pass/bandit14` command most likely got sent through the `nc` command as a literal string as it doesn't have a shell behind it. And so to avoid that I'll run the command in the terminal connecting both command with a pipe `|`. 

```bash
cat /etc/bandit_pass/bandit14 | nc localhost 30000
Correct!
[REDACTED]
```

It worked.

- `nc` is short for `Netcat`, a utility used to read and write data across network connections.
- `localhost` is the standard hostname meaning "this same computer" (resolving to IP address 127.0.0.1).
- `30000` is the specific communication endpoint (port number) where a local service or program is currently listening.

## Verification

Authenticate as `bandit15` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit15@bandit.labs.overthewire.org -p 2220
bandit15@bandit.labs.overthewire.org's password: [REDACTED]
bandit15@bandit:~$ whoami
bandit15
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit15@bandit:~$`) confirms successful authentication as `bandit15`.

## Why This Works

A process on the box (started as root, running as bandit14) is bound to `127.0.0.1:30000` and implements a trivial custom protocol: it reads a line of text from any connected client and compares it byte-for-byte against the contents of `/etc/bandit_pass/bandit14`. It doesn't care _how_ that connection was made — SSH, telnet, a browser, or `nc` — because TCP is transport-agnostic and this "protocol" is nothing more than "read a line, string-compare it." `netcat` works here because it does the minimum: opens a raw TCP socket and pipes stdin/stdout to it, with no protocol logic layered on top. That's the whole point of the level — proving you can talk to _any_ TCP service without a dedicated client, as long as you know (or can guess) the protocol.

## Key Takeaway

Don't roll your own authentication protocol on a raw socket — it has no encryption, no rate-limiting, no replay protection, and (as this level shows) no client-side trust boundary at all: anyone who can reach the port can "authenticate" with any tool that speaks TCP. If a service needs to check a secret, put it behind TLS and a real protocol (or just SSH), not a bare listener that accepts whatever `nc` sends it.

## Password

`[REDACTED - see level for retrieval method]`


---
