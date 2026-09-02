# Bandit Level 15 → 16

## Objective

The password for the next level can be retrieved by submitting the password of the current level to `port 30001` on localhost using SSL/TLS encryption.

## Concept Tested

Manually driving an SSL/TLS handshake from the command line - using `openssl s_client` as a raw encrypted-socket client, the TLS equivalent of what `nc` was for plaintext in the previous level.

## Initial Access

Establish an SSH session to the target host as user `bandit15`:

```bash
$ ssh bandit15@bandit.labs.overthewire.org -p 2220
bandit15@bandit.labs.overthewire.org's password: [REDACTED]
bandit15@bandit:~$
```

## Recon

To start off this command I tried the previous level's approach.

```bash
bandit15@bandit:~$ cat /etc/bandit_pass/bandit15
[REDACTED]
```

Then I tried to pipe it to the `nc localhost 30001`. 

```bash
bandit15@bandit:~$ cat /etc/bandit_pass/bandit15 | nc localhost 30001
```

I received no response so I'll assume it didn't work. The response either got dropped or got hung up with garbage/no readable response. That must mean the service is speaking a TLS handshake at the byte level, and plain `nc` has no idea how to negotiate that, so it just sees noise.
Since that didn't work I'll try `openssl s_client` as the objective specifically said that this level uses SSL encryption and `openssl s_client` is a tool built to perform a full TLS handshake as a client then exposes stdin/stdout as the plaintext application-layer channel once the encryption tunnel is up.

```bash
bandit15@bandit:~$ openssl s_client -connect localhost:30001
Connecting to 127.0.0.1
CONNECTED(00000003)
...
Certificate chain
 0 s:CN=SnakeOil
   i:CN=SnakeOil
...
Server certificate
-----BEGIN CERTIFICATE-----
...
```

After running the command a visible TLS handshake (CONNECTED, certificate chain info, a self-signed cert warning), then the connection sits open - same "waiting for input" behavior as in `bandit14`'s listener, this one is just wrapped in TLS. Confirming that the only thing different in this level is the transport.

## Solution

```bash
$ cat /etc/bandit_pass/bandit15 | openssl s_client -connect localhost:30001 -quiet
Correct!
[REDACTED]
```

Piping `cat`'s output into `openssl s_client` sends the password as the first line of data over the now-established encrypted channel. I added `-quiet` to suppress the certificate/handshake debug output so the response (`Correct!` + next password) is the only thing printed cleanly.

## Verification

Authenticate as `bandit16` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit16@bandit.labs.overthewire.org -p 2220
bandit16@bandit.labs.overthewire.org's password: [REDACTED]
bandit16@bandit:~$ whoami
bandit16
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit16@bandit:~$`) confirms successful authentication as `bandit16`.

## Why This Works

The service on port 30001 runs the same trivial "compare stdin to a known secret" logic as the one on 30000 in the previous level — the only difference is it's wrapped in a TLS listener instead of a bare TCP one. `openssl s_client` implements a full TLS client: it performs the handshake (key exchange, cipher negotiation, certificate exchange) and then, once the encrypted tunnel is established, hands you a plaintext-looking stdin/stdout interface — everything you type is transparently encrypted before hitting the wire, and everything the server sends back is transparently decrypted. That's why the _application-layer_ interaction (send password, receive password) is identical to level 14; TLS only wraps the transport, it doesn't change what the service does with the bytes once decrypted. Plain `nc` fails here most likely because it has no concept of a TLS record layer — it would try to send your password as raw bytes into what the server expects to be an encrypted ClientHello, which the server can't parse as anything meaningful.

## Key Takeaway

Encryption in transit (TLS) protects data from being read or tampered with _on the wire_, but it says nothing about what the server does with that data once decrypted — a service can be perfectly "encrypted" and still have zero real authentication logic behind it (as here, a bare string compare). Don't conflate "it's over TLS" with "it's secure": TLS defends against network-level eavesdropping/MITM, not against weak or absent server-side auth design.

## Password

`[REDACTED - see level for retrieval method]`


---
