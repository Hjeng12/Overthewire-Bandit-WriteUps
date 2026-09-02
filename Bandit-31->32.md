# Bandit Level 31 → 32

## Objective

There is a git repository at `ssh://bandit31-git@bandit.labs.overthewire.org/home/bandit31-git/repo` via the port `2220`. The password for the user `bandit31-git` is the same as for the user `bandit31`.

From your local machine (not the OverTheWire machine!), clone the repository and find the password for the next level. This needs git installed locally on your machine.

## Concept Tested

`.gitignore` as a local machine mechanism, not a security control. It only stops git from staging matching files by default; it has no ability to prevent a user from force adding a specific file, and a server-side hook can still validate whatever actually gets pushed regardless of what the ignore rules say.

## Initial Access

Establish an SSH session to the target host as user `bandit31`:

```bash
$ ssh bandit31@bandit.labs.overthewire.org -p 2220
bandit31@bandit.labs.overthewire.org's password: [REDACTED]
bandit31@bandit:~$
```

## Recon

Once again using the same cloning pattern as the prior levels.

```bash
[Laptop-3344]:/tmp/clonedgit$ git clone ssh://bandit31-git@bandit.labs.overthewire.org:2220/home/bandit31-git/repo
[Laptop-3344]:/tmp/clonedgit$ cd repo
[Laptop-3344]:/tmp/clonedgit/repo$ cat README.md
This time your task is to push a file to the remote repository.

Details:
    File name: key.txt
    Content: 'May I come in?'
    Branch: master
```

This time it seems the `README` file gives an explicit, unambiguous task rather than a puzzle to dig for. It provided what I assume is the exact filename, exact content and exact branch to obtain this password, which tells me the challenge isn't about finding anything but actually about correctly executing a git operation that something will actively be checking on the other end. So now I'll check for anything else present using the list command before jumping straight to a conclusion.

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ ls -la
total 8
drwxr-xr-x 3 Laptop-3344 Laptop-3344 100 Sep  1 22:19 .
drwxr-xr-x 3 Laptop-3344 Laptop-3344  60 Sep  1 22:19 ..
drwxr-xr-x 7 Laptop-3344 Laptop-3344 240 Sep  1 22:19 .git
-rw-r--r-- 1 Laptop-3344 Laptop-3344   6 Sep  1 22:19 .gitignore
-rw-r--r-- 1 Laptop-3344 Laptop-3344 147 Sep  1 22:19 README.md
[Laptop-3344]:/tmp/clonedgit/repo$ cat .gitignore
*.txt
```

Upon using the list command, It revealed a `.gitignore` file that I hadn't been told about was worth reading immediately, since its whole purpose is to influence what `git add`/`git status` do. The single rule `*.txt` was the catch: it matches the exact filename I was told to create, meaning a plain `git add key.txt` would silently do nothing useful. 

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ echo -n "May I come in?" > key.txt
[Laptop-3344]:/tmp/clonedgit/repo$ ls
README.md  key.txt
[Laptop-3344]:/tmp/clonedgit/repo$ git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
```

So now that I created the file exactly as specified, then checked `git status` before trying to add it confirms my prediction. That git shows `key.txt` under untracked files but flags it as ignored and as a result of that the `key.txt` file is completely omitted from the output, rather than offering it as something ready to stage. Confirming the block exists (and why) before working around it avoids wasted commands.

## Solution

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ git add -f key.txt
[Laptop-3344]:/tmp/clonedgit/repo$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   key.txt
```

- `.gitignore` is a default behavior toggle for `git add`/`git status`, not an access-control mechanism.
- `-f` force) explicitly tells git "I know this matches an ignore rule, stage it anyway."

This is the core of the level: the ignore file only stops the unintentional addition of matching files; it does nothing to stop a user who deliberately wants to add one.

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ git commit -m "adding key.txt"
Author identity unknown

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: empty ident name (for <[Laptop-3344].localdomain>) not allowed

```

Tried committing the `key.txt` file but ran into an unknown identity issue since git doesn't like not knowing the user contributing an attribute to a repository. Luckily the error contains the way the fix the issue so just follow it and you don't have to use the `--global` as you would only need it for this level and the `user.email` and `user.name` can be fabricated.

```bash
[Laptop-3344]:/tmp/clonedgit/repo$ git config user.email "you@example.com"
[Laptop-3344]:/tmp/clonedgit/repo$ git config user.name "Your Name"
[Laptop-3344]:/tmp/clonedgit/repo$ git commit -m "adding key.txt"
[master c7c45f8] adding key.txt
 1 file changed, 1 insertion(+)
 create mode 100644 key.txt
```

Standard commit, capturing the force-added file into a new commit on my local `master`, matching the branch the README specified. Also the method used to get around the error previously discussed.

```bash
git push
bandit31-git@bandit.labs.overthewire.org's password: [REDACTED]
```

Pushes the local `master` branch (with the new commit) to the remote. This is where the actual check happens, it isn't done on the client-side, but the server-side. The remote almost certainly runs a `pre-receive` (or similar) hook that inspects the incoming push, verifies a file named `key.txt` with the exact expected content landed on `master`, and only then reveals the next password as part of the push's remote-side output.

```bash
remote: ### Attempting to validate files... ####
...
remote: Well done! Here is the password for the next level:
remote: [REDACTED]
...
To ssh://bandit.labs.overthewire.org:2220/home/bandit31-git/repo
```

The `remote:`-prefixed lines are messages the server-side hook printed back to me during the push which is proof the check ran on the remote, not that I somehow tricked my local git client. My `.gitignore` workaround only had to get the file into my push, validating its content was the server's job entirely.

## Verification

Authenticate as `bandit32` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit32@bandit.labs.overthewire.org -p 2220
bandit32@bandit.labs.overthewire.org's password: [REDACTED]
bandit32@bandit:~$ whoami
bandit32
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit32@bandit:~$`) confirms successful authentication as `bandit32`.

## Why This Works

`.gitignore` operates entirely on the client, at `git add`/`git status` time. It's a UX feature meant to keep build artifacts, logs, and local junk out of `git status` noise, not a permission boundary. Git enforces nothing based on it beyond "don't auto-stage files matching these patterns," and even that default is trivially overridden with `-f`. Meanwhile, the actual gatekeeping in this level happens entirely server-side via a push hook, which inspects the content of the push independent of how the client arrived at it. The hook has no idea (and doesn't care) that `.gitignore` almost stopped this file from being added locally. This mirrors a real anti-pattern: teams sometimes assume `.gitignore`-ing a secrets file (like `.env`) means it can never end up in the repo, when in fact anyone who explicitly `git add -f`s it will commit and push it just fine.

## Key Takeaway

Never treat `.gitignore` as a security or compliance control, it prevents accidental commits, not deliberate ones, since `-f` bypasses it in one flag. If you need to guarantee a file (like credentials) never enters a repository, enforce that server-side — a pre-receive hook that rejects matching filenames/patterns, or a secret-scanning pipeline like GitHub's push protection rather than relying on local ignore rules that any contributor can override.

## Password

`[REDACTED - see level for retrieval method]`


---
