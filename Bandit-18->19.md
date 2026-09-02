# Bandit Level 18 → 19

## Objective

The password for the next level is stored in a file `readme` in the home directory. Unfortunately, someone has modified `.bashrc` to log you out when you log in with SSH. 

## Concept Tested

Shell startup file behavior (`.bashrc` executing on every interactive login shell) and how to bypass it - either by having SSH run a specific command non-interactively, or by focusing a shell that skips the offending startup file. 

## Initial Access

Establish an SSH session to the target host as user `bandit18`:

```bash
$ ssh bandit18@bandit.labs.overthewire.org -p 2220
bandit18@bandit.labs.overthewire.org's password: [REDACTED]

Byebye! 
Connection to bandit.labs.overthewire.org closed.
```

## Recon

As displayed in the Initial Access section, performing the normal login process into `bandit18` failed, which confirms the objectives' claim. The `Byebye!` message also confirms that not only is something kicking me out when I try to login but also that I'm on the right track as the password from level 17 was accepted before the disconnect.
Additionally, that also tells me that the `Byebye!` message is a startup script action and not an auth failure.
I believe this is happening because SSH gives you an interactive _login_ shell, and I think bandit's login startup files (`.bash_profile`/`.profile`) are set up to explicitly source `~/.bashrc` before handing you a prompt. If `.bashrc` contains something like `echo "Byebye!"; exit`, you're disconnected before you ever see a shell.

## Solution

```bash
[Laptop-3344]:~$ ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
                         _                     _ _ _
                        | |__   __ _ _ __   __| (_) |_
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_
                        |_.__/ \__,_|_| |_|\__,_|_|\__|


                      This is an OverTheWire game server.
            More information on http://www.overthewire.org/wargames

backend: gibson-1
bandit18@bandit.labs.overthewire.org's password: [REDACTED]
[REDACTED]
```

I tried running a command in the ssh command since when you give ssh a command argument, it runs that single command remotely over a non-interactive shell and exits. So in the end it never sources `.bashrc` at all, because `.bashrc` is especially for interactive shells. This sidesteps the trap entirely rather than trying to defeat it once connected.
And as the objective claims the password is in the `readme` file, so I `cat` it.

Alternatively, one can force an interactive shell that skips reading the `.bashrc`

```bash
[Laptop-3344]:~$ ssh -t bandit18@bandit.labs.overthewire.org -p 2220 "bash --norc"
                         _                     _ _ _
                        | |__   __ _ _ __   __| (_) |_
                        | '_ \ / _` | '_ \ / _` | | __|
                        | |_) | (_| | | | | (_| | | |_
                        |_.__/ \__,_|_| |_|\__,_|_|\__|


                      This is an OverTheWire game server.
            More information on http://www.overthewire.org/wargames

backend: gibson-1
bandit18@bandit.labs.overthewire.org's password: [REDACTED]
bash-5.3$ ls -la
total 24
drwxr-xr-x   2 root     root     4096 Jun 24 14:58 .
drwxr-xr-x 150 root     root     4096 Jun 24 15:02 ..
-rw-r--r--   1 root     root      220 Feb 13  2026 .bash_logout
-rw-r-----   1 bandit19 bandit18 3874 Jun 24 14:58 .bashrc
-rw-r--r--   1 root     root      807 Feb 13  2026 .profile
-rw-r-----   1 bandit19 bandit18   33 Jun 24 14:58 readme
bash-5.3$ cat readme
[REDACTED]
```

- `-t` forces the SSH to allocate a pseudo-terminal, which is needed for interactive work and not just for running one command. 
- `bash --norc` starts the bash shell that explicitly skips reading `~/.bashrc` on startup.

This option would be better if you need to poke around interactively rather than run one fixed command.

## Verification

Authenticate as `bandit19` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit19@bandit.labs.overthewire.org -p 2220
bandit19@bandit.labs.overthewire.org's password: [REDACTED]
bandit19@bandit:~$ whoami
bandit19
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit19@bandit:~$`) confirms successful authentication as `bandit19`.

## Why This Works

`.bashrc` is sourced automatically by every _interactive non-login_ bash shell — that's the mechanism the level abuses, weaponizing a normally-benign customization hook (aliases, prompt tweaks, env vars) into a forced logout. Both bypasses work by never triggering that sourcing step in the first place: `ssh host "command"` never spawns an interactive shell at all — it runs the command directly via a minimal shell invocation, so `.bashrc` is never read. `bash --norc` does spawn an interactive shell, but explicitly tells bash to skip the "read startup files" step. Neither approach "defeats" the malicious `.bashrc` through cleverness — they just never invoke it, because the trap only fires under one specific condition (interactive shell, default startup behavior) that both methods avoid.

## Key Takeaway

Don't assume `.bashrc`/`.bash_profile` is inert boilerplate — anyone with write access to it can turn it into a logout trap, a backdoor, or a credential harvester, since it runs arbitrary shell code on every interactive login. If a system's startup files are untrusted or you need to guarantee a clean environment (e.g. automated scripts, CI), explicitly bypass them (`ssh host cmd`, `bash --norc`, `env -i`) rather than relying on them behaving as expected.

## Password

`[REDACTED - see level for retrieval method]`


---
