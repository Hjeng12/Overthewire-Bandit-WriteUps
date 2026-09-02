# Bandit Level 16 → 17

## Objective

The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the `range 31000 to 32000`. First find out which of these ports have a server listening on them. Then find out which of those speak SSL/TLS and which don't. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.

## Concept Tested

Port scanning and service fingerprinting with `nmap` to narrow down an  unknown attack surface, followed by distinguished "real" services from decoys (echo servers) that mimic a valid response.

## Initial Access

Establish an SSH session to the target host as user `bandit16`:

```bash
$ ssh bandit16@bandit.labs.overthewire.org -p 2220
bandit16@bandit.labs.overthewire.org's password: [REDACTED]
bandit16@bandit:~$
```

## Recon

Seeing as the objective provided the range `(31000-32000 ports)` where the to send the password of the current directory, blind guessing is not needed. 

```bash
bandit16@bandit:~$ nmap -sV -p 31000-32000 localhost
Starting Nmap 7.98 ( https://nmap.org ) at 2026-08-26 03:10 +0000
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00051s latency).
Other addresses for localhost (not scanned): ::1
Not shown: 996 closed tcp ports (conn-refused)
PORT      STATE SERVICE     VERSION
31046/tcp open  echo
31518/tcp open  ssl/echo
31691/tcp open  echo
31790/tcp open  ssl/unknown
31960/tcp open  echo
```

- `nmap` was used as it is a command that scans networks for active computers, servers, etc. and listening, open or closed ports. 
- `-p 31000-32000` restricts the scan to just that range.
- `-sV` asks `nmap` to actively probe each open port and try to fingerprint what's actually running there.

After the scan was complete, there were only 5 ports listening but upon a closer look only 2 are running SSL. The other three are `echo` which is exactly the behavior the objective was warning me about. 

*P.S. `echo` is a service that just reflects back whatever you send it.*

So of the two ssl options, I decided to start with the first one tagged `ssl/echo` to confirm whether or not it's a dead end and not just assume.

```bash
bandit16@bandit:~$ cat /etc/bandit_pass/bandit16 | openssl s_client -connect localhost:31518 -quiet
Connecting to 127.0.0.1
Can't use SSL_get_servername
depth=0 CN=SnakeOil
verify error:num=18:self-signed certificate
verify return:1
depth=0 CN=SnakeOil
verify return:1
[REDACTED]
```

It seems the current directory password got echoed straight back, confirming this port was just an echo service wrapped by SSL. Now that it has been confirmed that there is only one port left to check.

## Solution

So now that the 5 ports have been narrowed down to one I shall input the command to retrieve the password for the next level.

```bash
$ cat /etc/bandit_pass/bandit16 | openssl s_client -connect localhost:31790 -quiet
Correct!
-----BEGIN RSA PRIVATE KEY-----
[REDACTED]
-----END RSA PRIVATE KEY-----
```

It seems like, unlike the plaintext-password levels, this one returned an entire RSA private key instead. Seeing as that is the case I'll attempt to log in to the next level by saving the RSA private key.

```bash
[Laptop-3344]:~$ nano private_key
[pasted the key contents, including BEGIN/END lines, saved]
[Laptop-3344]:~$ chmod 700 private_key
```

So like in level `13 -> 14` `chmod` command was used to change the permissions of the private key as `ssh` refuses to use a private key file if its permissions are too open, as a private key readable by others defeats the point of asymmetric auth. `chmod 700` was used to restrict read/write/execute to only the owner of the file, satisfying the SSH's sanity check.

## Verification

Authenticate as `bandit17` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh -i private_key bandit17@bandit.labs.overthewire.org -p 2220
bandit17@bandit:~$ whoami
bandit17
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit17@bandit:~$`) confirms successful authentication as `bandit17`.

## Why This Works

The scenario models a real internal-network recon problem: a service exists somewhere in a large port range, but its exact location isn't published, and several _decoy_ services (the echo servers) exist that superficially "respond" without doing anything meaningful. `nmap -sV` works by sending protocol-specific probes to each open port and pattern-matching the responses against a database of known service banners/behaviors, which is how it can distinguish "generic echo" from "unidentified SSL service" without you manually connecting to all five. The actual credential-granting service issues a real key pair per session/level and hands the private half to anyone who authenticates with the correct current password — this is functionally a tiny custom CA-less key-issuance service, gated only by "did you send the right string." SSH's refusal to use a world-or-group-readable key file isn't cosmetic: a private key's entire security model assumes only its owner can ever read it, so SSH enforces that assumption itself rather than trusting the filesystem.

## Key Takeaway

When scanning for services in an unknown range, don't just check "is the port open" — use active service fingerprinting (`nmap -sV`) to filter out decoys before manually testing each candidate, or you'll waste time hand-verifying services a scanner could have ruled out in one pass. And always lock down private key files (`chmod 700 or 600`) immediately after creating them — a key that anyone else on the box can read is equivalent to no authentication at all.

## Password

`[REDACTED - see level for retrieval method]`

---
