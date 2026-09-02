# Bandit Level 32 → 33

## Objective

After all this `git` stuff, it’s time for another escape. Good luck!

## Concept Tested

Escaping a restricted shell via shell expansion rather than a literal command name. Using the special parameter `$0` (which expands to the running shell's own path/name) to slip past a filter that only transforms literal typed text, not the result of variable expansion.

## Initial Access

Establish an SSH session to the target host as user `bandit32`:

```bash
$ ssh bandit32@bandit.labs.overthewire.org -p 2220
bandit32@bandit.labs.overthewire.org's password: [REDACTED]
WELCOME TO THE UPPERCASE SHELL 
>>
```

## Recon

Upon logging in I was dropped directly into a non-standard prompt (`>>`, not the usual `$`), which was the first sign this isn't a normal bash session. I figured it was worth noting immediately rather than assuming it's just a themed banner.

```bash
>> ls
sh: 1: LS: Permission denied
>> whoami
sh: 1: WHOAMI: Permission denied
>> pwd
sh: 1: PWD: Permission denied
>> ls -la
sh: 1: LS: Permission denied
```

The first thing I tried was to use the most basic commands first to characterize the restriction precisely, rather than assume from the banner text alone. The error messages were the key clue. `LS`, `WHOAMI`, `PWD` confirmed that the shell is taking my literal input, uppercasing it, then trying to execute that string as a command. Since every real Linux binary name is lowercase, every command I type this way is guaranteed to fail, no matter what I try.

If the filter operates on the literal characters I type, the way around it should be to make the shell execute something that isn't those literal characters at the moment the uppercasing happens. Something that only turns into a real command after the shell processes it internally, like a variable expansion. I knew `$0` was a shell special parameter (not something you set yourself with `export`) that reflects whatever the calling process passed as the "zeroth" argument when it started the shell, though after reading into it, that value turned out to be more of a convention than a guarantee, so I couldn't be fully sure in advance what it would actually contain here. My guess was that since expansion happens as part of normal shell parsing, it's a later step than whatever pass does the uppercasing on my raw keystrokes which would explain why `$0` isn't touched at all (neither `$` nor `0` has a case to mangle in the first place). What I couldn't be certain of without digging further is exactly what `$0` would resolve to in this specific shell. I initially assumed it'd be a full path like `/bin/sh`, but that's not actually guaranteed. Based on how similar restricted shells tend to be implemented (piping the modified input into something like a `system()` call), it's more likely to resolve to just the bare name `sh`. Either way, a bare name isn't directly runnable on its own meaning the shell would still need to search through the directories listed in `$PATH` to find and execute the matching file, the same lookup that happens any time I type a plain command like `ls`.

## Solution

```bash
>> $0
$
```

Typing the literal two characters `$0` still uppercases to `$0` (digits and `$` have no uppercase form, so the visible transformation does nothing to this input) but the crucial part is what happens when the resulting string is executed. The shell evaluates `$0` as a parameter expansion, substituting in the actual shell binary's path, and then executes that, spawning a brand-new, completely normal shell process. The prompt changing from `>>` to a plain `$` confirmed I'd broken out into an unrestricted shell.

```bash
$ ls -la 
-rwsr-x--- 1 bandit33 bandit32 ... uppershell
```

With a real shell, checked the home directory the same way as every prior level. Found `uppershell` which is a setuid binary owned by `bandit33` which also confirms that mechanism behind the whole restricted environment wasn't a login-shell trick like `showtext` (levels 25/26), but a compiled setuid program that itself implements the uppercase-and-execute loop, running with bandit33's effective privileges the entire time I was inside it (and after escaping it, since my new `$0`-spawned shell inherited that same process's privilege level).

```bash
$ whoami
bandit33
$ cat /etc/bandit_pass/bandit33
[REDACTED]
```

Since the shell I escaped into is a child process of the setuid `uppershell` binary, it inherited `euid=bandit33` which is the same setuid mechanism from levels 19/26, just reached via a shell-filter escape instead of a "run any command" helper. I also confirmed the user I was active as and noticed it was `bandit33` and so a direct `cat` on bandit33's password file succeeds here, with no further privilege trick needed.

## Verification

Authenticate as `bandit33` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit33@bandit.labs.overthewire.org -p 2220
bandit33@bandit.labs.overthewire.org's password: [REDACTED]
bandit33@bandit:~$ whoami
bandit33
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit33@bandit:~$`) confirms successful authentication as `bandit33`.

## Why This Works

`uppershell` almost certainly reads a line of input, transforms it (e.g., via something equivalent to `tr '[:lower:]' '[:upper:]'`), and passes the result to a real shell (`sh -c "$INPUT"`) for execution. The flaw is a mismatch between what the filter operates on and what actually gets executed: it uppercases my literal keystrokes, but the string handed to the underlying `sh -c` still gets fully parsed and expanded by that shell before running. Shell parameter expansion (`$0`, `$PATH`, command substitution, etc.) happens as a normal part of that parsing, independent of letter case. `$0` survives the filter completely unscathed (it contains no lowercase letters to mangle), and once `sh` evaluates it, the result is a lowercase path to a real shell binary — which then executes just fine. Because `uppershell` itself is `setuid-bandit33`, the shell it spawns via `$0` inherits that same effective UID, handing me a fully privileged shell in one step.

## Key Takeaway

A restricted shell that filters or transforms literal input text is not equivalent to one that restricts what actually gets executed. Anything that gets evaluated by the underlying interpreter after the filter runs (variable expansion, command substitution, globbing) is a potential bypass. If you're building a genuinely restricted shell, don't hand user input to a real `sh -c` at all after light preprocessing; use a strict allowlist of fully-resolved commands with no further shell interpretation, or a dedicated restricted-shell framework rather than a custom wrapper.

## Password

`[REDACTED - see level for retrieval method]`


---
