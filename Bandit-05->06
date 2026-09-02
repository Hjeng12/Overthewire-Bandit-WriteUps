# Bandit Level 5 → 6

## Objective

The password for the next level is stored in a file somewhere under the `inhere` directory and has all of the following properties: 

- human-readable
- 1033 bytes in size 
- not executable 

## Concept Tested

Combining multiple `find` filters (type, size, permissions) to narrow a large search space of decoy files down to a single match — practical use of `find`'s predicate options beyond simple name/type matching.

## Initial Access

Establish an SSH session to the target host as user `bandit5`:

```bash
$ ssh bandit5@bandit.labs.overthewire.org -p 2220
bandit5@bandit.labs.overthewire.org's password: [REDACTED]
bandit5@bandit:~$
```

## Recon

The first command ran was the list command to identify the location of the directory `inhere` and reveal the ownership and POSIX mode bits.

```bash
bandit5@bandit:~$ ls -la
total 24
drwxr-xr-x   3 root root    4096 Jun 24 14:59 .
drwxr-xr-x 150 root root    4096 Jun 24 15:02 ..
-rw-r--r--   1 root root     220 Feb 13 12:16 .bash_logout
-rw-r--r--   1 root root    3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root root     807 Feb 13 12:16 .profile
drwxr-x---  22 root bandit5 4096 Jun 24 14:59 inhere
```

Now that the `inhere` directory has been identified to be located in the current directory. One would now have to navigate to the `inhere` directory. 

```bash
bandit5@bandit:~$ cd inhere/
bandit5@bandit:~/inhere
```

After navigating to the `inhere` directory run this list command again to identify the files stored there.

```bash
bandit5@bandit:~/inhere$ ls -la 
total 88
drwxr-x--- 22 root bandit5 4096 Jun 24 14:59 .
drwxr-xr-x  3 root root    4096 Jun 24 14:59 ..
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere00
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere01
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere02
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere03
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere04
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere05
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere06
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere07
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere08
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere09
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere10
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere11
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere12
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere13
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere14
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere15
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere16
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere17
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere18
drwxr-x---  2 root bandit5 4096 Jun 24 14:59 maybehere19
```

## Solution

To reveal the password of this challenge one would need to find the exact file that the password is stored in and the command to do that based on the objective would be.

```bash
bandit5@bandit:~/inhere$ find . -type f -size 1033c ! -executable
./maybehere07/.file2
```

- `-type f` indicate regular files only, skip directories.
- `-size 1033c` indicates the exact byte size match.
- `! -executable` indicates the negation of the executable-permission test, so any file that has executable bits will be excluded.

And so now that 20 `maybehere` files identified in the Recon Section have been narrowed down to one file, only thing to do is `cat` the file.

```bash
bandit5@bandit:~/inhere$ cat ./maybehere07/.file2
[REDACTED]
```

*P.S. Bytes are represented as `c` so `1033 bytes` is `1033c`.*

## Verification

Authenticate as `bandit6` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit6@bandit.labs.overthewire.org -p 2220
bandit6@bandit.labs.overthewire.org's password: [REDACTED]
bandit6@bandit:~$ whoami
bandit6
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit6@bandit:~$`) confirms successful authentication as `bandit6`.

## Why This Works

`find` supports composable predicates that are evaluated as a logical AND by default when listed side by side, and each predicate inspects filesystem metadata directly (inode size, type, permission bits) rather than relying on filenames or manual inspection. This lets you describe a target file by its _properties_ instead of searching by content or guessing names, which scales to arbitrarily deep/wide directory trees — a hundred decoy files and folders is no harder to search than ten, since the constraints are checked programmatically rather than by eye.

## Key Takeaway

When you need to find "a specific file" in a large or adversarial filesystem, describe it by verifiable attributes (size, type, permissions, modification time, owner) via `find`'s predicates rather than trying to eyeball directory listings — this is both faster and immune to intentionally misleading names.

## Password

`[REDACTED - see level for retrieval method]`

---
