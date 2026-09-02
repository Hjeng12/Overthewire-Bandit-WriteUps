# Bandit Level 28 → 29

## Objective

There is a git repository at `ssh://bandit28-git@bandit.labs.overthewire.org/home/bandit28-git/repo` via the port `2220`. The password for the user `bandit28-git` is the same as for the user `bandit28`.

From your local machine (not the OverTheWire machine!), clone the repository and find the password for the next level. This needs git installed locally on your machine.

## Concept Tested

Git history immutability, a "fix" commit that removes sensitive data from the current file state does not erase it from the repository; every prior commit is still fully retrievable, meaning `git log` / `git log -p` / `git show` can resurrect anything ever committed.

## Initial Access

Establish an SSH session to the target host as user `bandit28`:

```bash
$ ssh bandit28@bandit.labs.overthewire.org -p 2220
bandit28@bandit.labs.overthewire.org's password: [REDACTED]
bandit28@bandit:~$
```

## Recon

I'll start this by using the same pattern as level 27→28 — writable scratch directory, clone the named repo using the `bandit28` password (per the level description), `bandit28-git` shares it.

```bash
[Laptop-3344]:~$ git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo
[Laptop-3344]:/tmp/clonedgit$ cd repo
[Laptop-3344]:/tmp/clonedgit/repo$ ls -la
total 4
drwxr-xr-x 3 Laptop-3344 Laptop-3344  80 Sep  1 00:27 .
drwxrwxrwx 3 Laptop-3344 Laptop-3344  60 Sep  1 00:27 ..
drwxr-xr-x 7 Laptop-3344 Laptop-3344 240 Sep  1 00:27 .git
-rw-r--r-- 1 Laptop-3344 Laptop-3344 111 Sep  1 00:27 README.md
```

I cloned the `bandit28-git` and then moved to the `repo` directory immediately after that I ran the list command to locate any files that may be stored within.

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ cat README.md
# Bandit Notes
Some notes for level29 of bandit.

## credentials

- username: bandit29
- password: xxxxxxxxxx
```

I proceeded to `cat` the `README` file and noticed the working-tree file first, the same move as level 27 — but this time the password field is a placeholder, not a real value. That's the signal the level goal is pointing at: the current content is deliberately scrubbed, so the real value must exist somewhere else in the repo. Most likely its history, since this is a git-specific level.

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ git log
commit e2e1de5396037bafb23e9bb37c12ebea9b911cfd (HEAD -> master, origin/master, origin/HEAD)
Author: Morla Porla <morla@overthewire.org>
Date:   Wed Jun 24 14:59:20 2026 +0000

    fix info leak

commit 2678cfadd8f2a347bc23e1ea491f702e5b184709
Author: Morla Porla <morla@overthewire.org>
Date:   Wed Jun 24 14:59:20 2026 +0000

    add missing data

commit 9530d526c22b9e6e6ae11070ef8ff8ee21eb2e02
Author: Ben Dover <noone@overthewire.org>
Date:   Wed Jun 24 14:59:20 2026 +0000

    initial commit of README.md
```

- `git log` lists every commit in reverse chronological order along with its message.

As I read the commit messages, I looked for anything that stood out — commit messages alone are often enough to spot where something changed. `fix info leak` immediately stood out as the most likely place a real password got replaced with a placeholder. The "leak" part of the commit message strongly implies something sensitive was exposed and then patched.

## Solution

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ git log -p -1
commit e2e1de5396037bafb23e9bb37c12ebea9b911cfd (HEAD -> master, origin/master, origin/HEAD)
Author: Morla Porla <morla@overthewire.org>
Date:   Wed Jun 24 14:59:20 2026 +0000

    fix info leak

diff --git a/README.md b/README.md
index 42331d9..5c6457b 100644
--- a/README.md
+++ b/README.md
@@ -4,5 +4,5 @@ Some notes for level29 of bandit.
 ## credentials

 - username: bandit29
-- password: [REDACTED]
+- password: xxxxxxxxxx
```

- `-p` shows the actual diff introduced by each commit (not just the message).
- `-1` limits output to the single most recent commit, exactly the "fix info leak" one I'd flagged.

The diff makes the exploit trivial to read: the `--` line is what the file contained before this commit (the real password), and the `+-` line is what replaced it (the placeholder). Git stores full snapshots/diffs of every commit forever by default. Deleting a line in a new commit doesn't touch the old commit object, it just adds a newer one on top.

## Verification

Authenticate as `bandit29` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit29@bandit.labs.overthewire.org -p 2220
bandit29@bandit.labs.overthewire.org's password: [REDACTED]
bandit29@bandit:~$ whoami
bandit29
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit29@bandit:~$`) confirms successful authentication as `bandit29`.

## Why This Works

Git is fundamentally an append-only content-addressable store. Every commit creates a new, immutable snapshot object referenced by its hash, and existing commits are never modified or deleted by ordinary operations like a new commit or even `git rm`. When someone "removes" a secret by committing a change that deletes it from the current file, all that actually happens is a _new_ commit object gets created; the previous commit, the one where the secret still exists in plaintext, remains fully intact and reachable via `git log`, `git show <hash>`, or `git checkout <hash>`. Anyone with clone access to the repository has access to its entire commit graph, not just the latest snapshot, so "fixing" a leak after the fact only prevents future clones from seeing it in the default view, it does nothing to the object database itself.

## Key Takeaway

Never assume deleting a secret in a new commit "removes" it. If a credential, key, or token was ever committed to a git repository, treat it as permanently compromised and rotate it immediately; the only real fix is invalidating the leaked secret, not just editing it out of the latest commit (history rewriting via `git filter-repo`/BFG can scrub it from the repo, but you must also assume anyone who already cloned it still has the old value).

## Password

`[REDACTED - see level for retrieval method]`


---
