# Bandit Level 30 → 31

## Objective

There is a git repository at `ssh://bandit30-git@bandit.labs.overthewire.org/home/bandit30-git/repo` via the port `2220`. The password for the user `bandit30-git` is the same as for the user `bandit30`.

From your local machine (not the OverTheWire machine!), clone the repository and find the password for the next level. This needs git installed locally on your machine.

## Concept Tested

Git tags as another category of ref (alongside branches) that can point to a commit and, unlike branches that are often used as fixed "bookmarks" (e.g. release markers) which are easy to overlook if you only think to check `git log` and `git branch`.

## Initial Access

Establish an SSH session to the target host as user `bandit30`:

```bash
$ ssh bandit30@bandit.labs.overthewire.org -p 2220
bandit30@bandit.labs.overthewire.org's password: [REDACTED]
bandit30@bandit:~$
```

## Recon

Using the same opening as I have been for the past 3 levels so far.


```bash
[Laptop-3344]:/tmp/clonedgit$ git clone ssh://bandit30-git@bandit.labs.overthewire.org:2220/home/bandit30-git/repo
[Laptop-3344]:/tmp/clonedgit$ cd repo
[Laptop-3344]:/tmp/clonedgit/repo$ cat README.md
just an epmty file... muahaha
```

This time the `README` file isn't even trying to look like real project notes, it's explicitly taunting ("muahaha"), which told me not to waste more time reading file contents and to go straight to git's metadata instead.

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ git log
commit 929c564cd34ca667773e2eb02f74b514bc4eeebf (HEAD -> master, origin/master, origin/HEAD)
Author: Ben Dover <noone@overthewire.org>
Date:   Wed Jun 24 14:59:25 2026 +0000

    initial commit of README.md
[Laptop-3344]:/tmp/clonedgit/repo$ git log -p -1
commit 929c564cd34ca667773e2eb02f74b514bc4eeebf (HEAD -> master, origin/master, origin/HEAD)
Author: Ben Dover <noone@overthewire.org>
Date:   Wed Jun 24 14:59:25 2026 +0000

    initial commit of README.md

diff --git a/README.md b/README.md
new file mode 100644
index 0000000..029ba42
--- /dev/null
+++ b/README.md
@@ -0,0 +1 @@
+just an epmty file... muahaha
[Laptop-3344]:/tmp/clonedgit/repo$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
```

Ran the three techniques that worked in levels 28 and 29 first, since they're the natural next step, but this repo has only a single trivial commit and no branches beyond `master`. That ruled out "hidden in history" and "hidden on another branch" as the mechanism this time, meaning the level was testing a different git feature, not a harder version of the same one.

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ git tag
secret
```

With commits and branches exhausted, I researched what other ref types git supports and came across, `tags`. `Tags` are another major one, commonly used to mark specific commits (release versions, milestones) without creating a whole branch. Running `git tag` bare lists all tag names, and one showed up immediately: `secret`, a name that's on-the-nose enough to be the obvious next thing to inspect.

## Solution

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ git show secret
[REDACTED]
```

- `git show <ref>` works on any ref types: commits, branches, or tags and displays the object it points to.

Since a `tag` is (in the lightweight case) just a named pointer to a specific commit, `git show secret` resolves straight to that commit's content, which in this repo is simply the password string itself rather than a real code change. No `checkout` or `diff` needed; the `tag` is the answer.

## Verification

Authenticate as `bandit31` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit31@bandit.labs.overthewire.org -p 2220
bandit31@bandit.labs.overthewire.org's password: [REDACTED]
bandit31@bandit:~$ whoami
bandit31
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit31@bandit:~$`) confirms successful authentication as `bandit31`.

## Why This Works

Git has multiple kinds of refs that can all point into the same commit graph: branches (`refs/heads/*`), remote-tracking branches (`refs/remotes/*`), and tags (`refs/tags/*`). All of them are just named pointers stored in `.git`, fetched in full whenever someone clones the repo. There's no special access control distinguishing "tags" from "branches" from "history." This level's setup deliberately avoids the two ref types used in the previous two levels (commit history, branches) specifically to demonstrate that a thorough git audit has to check every ref namespace, not just the ones you've already learned to look for. A tag is exactly as easy to read as a commit or branch once you know it exists. `git tag` to list and `git show` to inspect, the only real barrier is remembering it's a category worth checking at all.

## Key Takeaway

When auditing a git repository for leaked data, check all ref types, not just `git log` on the default branch. Run `git branch -a`, `git tag`, and even `git stash list` (on a live working copy) as standard practice, since any of them can retain committed content indefinitely. From the developer side: nothing you commit to a tag, branch, or stash is any "safer" than committing it to master. Treat every git object as equally permanent and equally exposed to anyone with clone access.

## Password

`[REDACTED - see level for retrieval method]`


---
