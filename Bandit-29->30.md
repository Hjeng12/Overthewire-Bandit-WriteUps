# Bandit Level 29 → 30

## Objective

There is a git repository at `ssh://bandit29-git@bandit.labs.overthewire.org/home/bandit29-git/repo` via the port `2220`. The password for the user `bandit29-git` is the same as for the user `bandit29`.

From your local machine (not the OverTheWire machine!), clone the repository and find the password for the next level. This needs git installed locally on your machine.

## Concept Tested

Git branches as separate, independently-inspectable lines of history, auditing every branch (`git branch -a`), not just the default one, since sensitive data intentionally scrubbed from "production" can still be sitting untouched on a dev/experimental branch.

## Initial Access

Establish an SSH session to the target host as user `bandit29`:

```bash
$ ssh bandit29@bandit.labs.overthewire.org -p 2220
bandit29@bandit.labs.overthewire.org's password: [REDACTED]
bandit29@bandit:~$
```

## Recon

Just like in the previous level I'll do the same things again.

```bash
[Laptop-3344]:/tmp/clonedgit$ git clone ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo
[Laptop-3344]:/tmp/clonedgit$ cd repo
[Laptop-3344]:/tmp/clonedgit/repo$ cat README.md
# Bandit Notes
Some notes for bandit30 of bandit.

## credentials

- username: bandit30
- password: <no passwords in production!>
```

Now this time the password field has been replaced with an explanatory message rather than a redacted placeholder like in level 28. The key difference here is that it's not just hiding the value, it's telling me why it's missing. After looking at the message for some time I have come to the conclusion that the phrasing ("in production") is a deliberate hint about git's branching model, not just a random string. So just like in the last level I'll run the `git log` command first and observe what happens.

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ git log
commit b607fba0c867d0bbdf4b4a5e62cd04b79c8fea83 (HEAD -> master, origin/master, origin/HEAD)
Author: Ben Dover <noone@overthewire.org>
Date:   Wed Jun 24 14:59:22 2026 +0000

    fix username

commit 84c16f8255d7a07c45b96a1780234ed20f7457b7
Author: Ben Dover <noone@overthewire.org>
Date:   Wed Jun 24 14:59:22 2026 +0000

    initial commit of README.md
```

Upon checking the `git` logs it doesn't seem to me that these commits were important. But just to ensure I don't miss anything I'll check what they were before the commits.

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ git log -p -2
commit b607fba0c867d0bbdf4b4a5e62cd04b79c8fea83 (HEAD -> master, origin/master, origin/HEAD)
Author: Ben Dover <noone@overthewire.org>
Date:   Wed Jun 24 14:59:22 2026 +0000

    fix username

diff --git a/README.md b/README.md
index 2da2f39..1af21d3 100644
--- a/README.md
+++ b/README.md
@@ -3,6 +3,6 @@ Some notes for bandit30 of bandit.

 ## credentials

-- username: bandit29
+- username: bandit30
 - password: <no passwords in production!>


commit 84c16f8255d7a07c45b96a1780234ed20f7457b7
Author: Ben Dover <noone@overthewire.org>
Date:   Wed Jun 24 14:59:22 2026 +0000

    initial commit of README.md

diff --git a/README.md b/README.md
new file mode 100644
index 0000000..2da2f39
--- /dev/null
+++ b/README.md
@@ -0,0 +1,8 @@
+# Bandit Notes
+Some notes for bandit30 of bandit.
+
+## credentials
+
+- username: bandit29
+- password: <no passwords in production!>
+
```

Upon closer look the commits here were minor (a username correction), with nothing resembling the "fix info leak" pattern from before. That told me this level's answer isn't hiding in this branch's history at all, reinforcing that I needed to look at the repo's structure more broadly rather than digging deeper into `master`.

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/dev
  remotes/origin/master
  remotes/origin/sploits-dev
```

- `-a` lists all the branches, including remote-tracking ones in the repo and not just the one branch checked out locally.

`git clone` only checks out the default branch (`master`) into my working directory, but it fetches the full history of every branch on the remote, so they're all sitting there, retrievable, even though I'm not "on" them. Seeing `dev` and `sploits-dev` alongside `master` directly validated the `README` file's hint: this repo has separate environments, and "production" (`master`) is exactly the one branch guaranteed not to have the password.

## Solution

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ git diff remotes/origin/dev
diff --git a/README.md b/README.md
index d395d04..1af21d3 100644
--- a/README.md
+++ b/README.md
@@ -4,5 +4,5 @@ Some notes for bandit30 of bandit.
 ## credentials

 - username: bandit30
-- password: [REDACTED]
+- password: <no passwords in production!>

diff --git a/code/gif2ascii.py b/code/gif2ascii.py
deleted file mode 100644
index 8b13789..0000000
--- a/code/gif2ascii.py
+++ /dev/null
@@ -1 +0,0 @@
-
```

Rather than fully checking out the `dev` branch (which changes my working directory state), I used `git diff` to directly compare my current `HEAD` (master) against the `dev` branch's version of the same file. Git can `diff` against any ref without checking it out. It just reads both tree objects and compares them. The `--` line here is `dev`'s actual content; the `+-` line is what I currently have on master. That immediately surfaced the real password sitting in the untouched dev-branch copy of the `README` file.

## Verification

Authenticate as `bandit30` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit30@bandit.labs.overthewire.org -p 2220
bandit30@bandit.labs.overthewire.org's password: [REDACTED]
bandit30@bandit:~$ whoami
bandit30
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit30@bandit:~$`) confirms successful authentication as `bandit30`.

## Why This Works

Git branches aren't separate repositories. They're just named pointers into the same shared commit graph, all stored in one `.git` object database. When you `git clone`, you don't just get `master`'s files; you get the entire object database, including every commit reachable from every branch on the remote, because git has no concept of "only fetch the branch someone's supposed to look at." The workflow this level models is a common real one: a team keeps "production" (`master`) clean of secrets, while a "dev" or "sploits-dev" branch, used for local testing, feature work, or throwaway experiments' still has a real credential committed, because whoever pushed it assumed dev branches are lower-stakes or private. But from a read-access perspective, once someone can clone the repo at all, every branch is equally exposed; there's no separate permission boundary between branches within the same repository.

## Key Takeaway

Access control in git operates at the repository level, not the branch level. If someone can clone your repo, they can read every branch's full history, not just the one you intended them to see. Never assume a secret is "safe" because it's only on a dev/experimental/feature branch rather than main; if it's committed anywhere in a repo someone else can clone, treat it as leaked and rotate it, and use `git branch -a` as a standard part of any repo audit.

## Password

`[REDACTED - see level for retrieval method]`


---
