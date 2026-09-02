# Bandit Level 6 → 7

## Objective

The password for the next level is stored somewhere on the server and has all of the following properties: 

- owned by user bandit7
- owned by group bandit6
- 33 bytes in size

## Concept Tested

File system searching by ownership and metadata combined with **Standard Error (stderr) redirection**. It shows how to locate misplaced or hidden files across an entire Linux directory tree using specific attributes like user, group, and size, while managing terminal noise.

## Initial Access

Establish an SSH session to the target host as user `bandit6`:

```bash
$ ssh bandit6@bandit.labs.overthewire.org -p 2220
bandit6@bandit.labs.overthewire.org's password: [REDACTED]
bandit6@bandit:~$
```

## Recon

The first command ran was the list command to identify the files located on the current directory and reveal the ownership and POSIX mode bits.

```bash
bandit6@bandit:~$ ls -la
total 20
drwxr-xr-x   2 root root 4096 Jun 24 14:58 .
drwxr-xr-x 150 root root 4096 Jun 24 15:02 ..
-rw-r--r--   1 root root  220 Feb 13 12:16 .bash_logout
-rw-r--r--   1 root root 3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root root  807 Feb 13 12:16 .profil
```

At first glance, there are no files worth addressing in the current directory. But as the objective stated that the file with the password is located somewhere on the system. I will then try to locate it on the system using the `find` command for files smaller or equal to 33 bytes as stated in the objective.

```bash
bandit6@bandit:~$ find / -type f -size 33c 2>/dev/null
/run/tmpfiles.d/static-nodes.conf
/var/lib/dpkg/info/libtasn1-6:amd64.shlibs
/var/lib/dpkg/info/install-info.triggers
...
```

## Solution

Now to find the location of the file with the password. One would only need to input a command to locate the file on the system that is owned by user bandit7 and group bandit6 and is 33 bytes.

```bash
bandit6@bandit:~$ find / -type f -group bandit6 -user bandit7 -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password
```

What the code is doing:
- `find` is the Linux tool used to search for files and directories.
	- `/` states that the search will be at the root directory, so when the `find` command is ran it will scan everything in the system.
	- `-type f` ensures the command looks for regular files, skipping folders
	- `-group bandit6` filters the files that belongs to group `bandit6`.
	- `-user bandit7` filters the owned files for the user `bandit7`.
	- `-size 33c` looks for a file that is exactly 33 bytes big. (`c` means bytes)
	- `2>/dev/null` ensures to hide "permission denied" errors by throwing in the system trash (`/dev/null`)

Now that the file with the password is located. One would only need to `cat` the file path.

```bash
bandit6@bandit:~$ cat /var/lib/dpkg/info/bandit7.password
[REDACTED]
```

## Verification

Authenticate as `bandit7` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit7@bandit.labs.overthewire.org -p 2220
bandit7@bandit.labs.overthewire.org's password: [REDACTED]
bandit7@bandit:~$ whoami
bandit7
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit7@bandit:~$`) confirms successful authentication as `bandit7`.

## Why This Works

Every file on a Unix filesystem carries ownership metadata (UID and GID) stored in its inode, independent of its name, location, or content — `find`'s `-user` and `-group` predicates read that metadata directly, the same way `-size` reads the byte-length field. This means you can locate a file purely by "who owns it," which is useful precisely because ownership is a stable, non-cosmetic attribute that can't be disguised by clever naming or directory placement. Separately, `find`'s behavior of continuing past unreadable directories (rather than aborting) is by design — a search across a whole filesystem will inevitably hit protected directories, and `find` reports each one as an error on stderr but keeps going, so redirecting stderr is a normal, expected part of running `find /` as a non-root user rather than a workaround for something broken.

## Key Takeaway

When searching a filesystem you don't fully control, expect permission errors and separate them from real output early (`2>/dev/null` or `2>errors.log`) — and remember that ownership (`-user`/`-group`) is just as reliable a search key as name or size, often more so, since it reflects an administrative fact about the file rather than something set by whoever created it.

## Password

`[REDACTED - see level for retrieval method]`

---
