# Bandit Level 20 → 21

## Objective

There is a setuid binary in the home directory that does the following: it makes a connection to localhost on the port you specify as a command line argument. It then reads a line of text from the connection and compares it to the password in the previous level (`bandit20`). If the password is correct, it will transmit the password for the next level (`bandit21`).

***NOTE:** Try connecting to your own network daemon to see if it works as you think*

## Concept Tested

Acting as your own "network daemon", running a listener that serves a value to a setuid client, combined with a basic shell job control(`&`, `jobs`) to run two processes concurrently in one terminal.

## Initial Access

Establish an SSH session to the target host as user `bandit20`:

```bash
$ ssh bandit20@bandit.labs.overthewire.org -p 2220
bandit20@bandit.labs.overthewire.org's password: [REDACTED]
bandit20@bandit:~$
```

## Recon

To start this challenge, the first thing I did was run the list command to identify the files located in the current directory and to check ownership and permissions on any files I am able to discover.

```bash
bandit20@bandit:~$ ls -la
total 36
drwxr-xr-x   2 root     root      4096 Jun 24 14:58 .
drwxr-xr-x 150 root     root      4096 Jun 24 15:02 ..
-rw-r--r--   1 root     root       220 Feb 13  2026 .bash_logout
-rw-r--r--   1 root     root      3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root     root       807 Feb 13  2026 .profile
-rwsr-x---   1 bandit21 bandit20 15604 Jun 24 14:58 suconnect
```

Just as in the previous level, the setuid bit of the file `suconnect` has a `s` with the addition of it being owned by `bandit21` so it's safe to assume the pattern is the same as `bandit20-do`. So I should be able to run the file `suconnect`. Additionally, `bandit20` does have access to execute the file demonstrated with the `r-x` section of the privileges above.

```bash
bandit20@bandit:~$ ./suconnect
Usage: ./suconnect <portnumber>
This program will connect to the given port on localhost using TCP. If it receives the correct password from the other side, the next password is transmitted back.
```

So I ran the file bare to observe what happens. Executing the file outputted a usage string which confirms it takes exactly one argument, which is a port number as stated in the objective through the statement "it makes a connection to localhost on the port you specify as a command line argument". 

The reason I did this was because the level description says this binary is the client which means it initiates the outbound connection and reads one line back. That to me means I need to be the server it connects to, something listening on a chosen port, ready to hand back the `bandit20` password the instant a connection comes in. The level's explicit hint ("try connecting to your own network daemon") is really just saying: you have to build both halves of this exchange yourself, in the same terminal, using job control so one doesn't block the other.

## Solution

So first I'll set up the listeners side before running the client as a listener has to exit before something can connect to it.

```bash
bandit20@bandit:~$ echo "ThePasswordForBandit20" | nc -lp 11111 &
[1] 193
```

- `echo "<password>"` Piping the password for `bandit20` into it means the very first (and only) thing that listener sends to any connecting client is that one line of text, which is exactly what `suconnect` was designed to read and compare.
- `nc -lp 11111` opens up a raw TCP listener on port `11111`.
- `&` trailing after the port number `11111` backgrounds the job once the command is ran so my shell prompt returns immediately instead of blocking on the listener.

```bash
bandit20@bandit:~$ jobs
[1]+  Running                    echo "ThePasswordForBandit20" | nc -lp 11111 &
```

Ran the `jobs` command to ensure the process is still alive and running in the background before activating the client.

```bash
bandit20@bandit:~$ ./suconnect 11111
Read: ThePasswordForBandit20
Password matches, sending next password
[REDACTED]
[1]+  Done                       echo "ThePasswordForBandit20" | nc -lp 11111
```

I executed `suconnect` (the setuid client) against the port my listener was bound to. `suconnect` connects out to `localhost:11111` as an ordinary TCP client and reads one line from that connection — the line my `nc`/`echo` sent, which was the password I supplied. It then compares that received line against `bandit20`'s password, which `suconnect` reads directly from `/etc/bandit_pass/bandit20`, readable to it because its effective UID is `bandit21`. Since my supplied value matches, the check passes, and `suconnect` prints `/etc/bandit_pass/bandit21` using that same elevated privilege.

## Verification

Authenticate as `bandit21` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit21@bandit.labs.overthewire.org -p 2220
bandit21@bandit.labs.overthewire.org's password: [REDACTED]
bandit21@bandit:~$ whoami
bandit21
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit21@bandit:~$`) confirms successful authentication as `bandit21`.

## Why This Works

`suconnect` is setuid `bandit21`, so when `bandit20` executes it, the resulting process's effective UID becomes `bandit21` for the duration of the run, exactly like `bandit20-do` in the previous level. That's what lets it read `/etc/bandit_pass/bandit21` at the end, a file `bandit20` has no direct access to. But the actual mechanic being taught here is different: this binary doesn't take a command from me, it takes a network response. It trusts whatever the first line of data from `localhost:<port>` says, compares it to a value it already knows internally — `bandit20`'s password, read directly from `/etc/bandit_pass/bandit20` using that same elevated privilege — and rewards a correct answer with a secret. Because I control both ends of the exchange, I know the correct value in advance and I control the server it connects to, so I can simply hand it the right answer directly instead of this being any kind of real authentication.

## Key Takeaway

Never write an authentication check where the client blindly trusts the first response from a network peer with no server identity verification. This binary connects to `localhost:<port>`, but nothing stops it from connecting to an attacker-controlled listener that happens to know or guess the expected value, exactly as happened here intentionally. Real credential checks should happen against a trusted, fixed endpoint (or use mutual authentication), never "whatever answered when I dialed this port."

## Password

`[REDACTED - see level for retrieval method]`


---
