# Bandit Level 12 → 13

## Objective

The password for the next level is stored in the file `data.txt`, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work. Use `mkdir` with a hard to guess directory name. Or better, use the command “`mktemp -d`”. Then copy the datafile using `cp`, and rename it using `mv` (read the manpages!)

## Concept Tested

Nested data decompression and file type identification (often called "peeling the onion"). Rather than a software exploit, it trains you to inspect raw hex data (`xxd`), recognize magic numbers/file signatures, and recursively unpack mixed archive formats like Gzip, Bzip2 and Tar.

## Initial Access

Establish an SSH session to the target host as user `bandit12`:

```bash
$ ssh bandit12@bandit.labs.overthewire.org -p 2220
bandit12@bandit.labs.overthewire.org's password: [REDACTED]
bandit12@bandit:~$
```

## Recon

To start this challenge the list command will be ran to ensure the `data.txt` file is located in the current directory and to reveal the ownership and POSIX mode bits.

```bash
bandit12@bandit:~$ ls -la
total 24
drwxr-xr-x   2 root     root     4096 Jun 24 14:58 .
drwxr-xr-x 150 root     root     4096 Jun 24 15:02 ..
-rw-r--r--   1 root     root      220 Feb 13 12:16 .bash_logout
-rw-r--r--   1 root     root     3851 Jun 24 14:50 .bashrc
-rw-r--r--   1 root     root      807 Feb 13 12:16 .profile
-rw-r-----   1 bandit13 bandit12 2641 Jun 24 14:58 data.txt
```

After the scan has been completed, it can be confirmed that the `data.txt` file is stored in the current directory. I have also noticed that the `data.txt` file is of decent size, so I will `cat` 3 lines of the file in an attempt to reveal what is stored in the file. I will also check the file type.

```bash
bandit12@bandit:~$ file data.txt
data.txt: ASCII text
bandit12@bandit:~$ cat data.txt | head -n 3
00000000: 1f8b 0808 a6f0 3b6a 0203 6461 7461 322e  ......;j..data2.
00000010: 6269 6e00 0144 02bb fd42 5a68 3931 4159  bin..D...BZh91AY
00000020: 2653 5904 ab91 e100 001c 7fff fffb bebf  &SY.............
```

Through the `cat` command it has been revealed that `data.txt` is a hexdump (offset, then hex byte pairs) rather than the file itself, so before anything else I needed to reconstruct the actual binary from it. 

```bash
bandit12@bandit:~$ TMPDIR=$(mktemp -d /tmp/banditXXXX)
bandit12@bandit:~$ cd $TMPDIR
bandit12@bandit:/tmp/banditNygA$ cp ~/data.txt .
bandit12@bandit:/tmp/banditNygA$ ls -la
total 4
drwx------  2 bandit12 bandit12   60 Aug 12 00:42 .
drwxrwx-wt 77 root     root     1860 Aug 12 00:42 ..
-rw-r-----  1 bandit12 bandit12 2641 Aug 12 00:42 data.txt
```

I worked in a scratch temp directory rather than my home dir, since I knew from the objective this would generate a chain of intermediate files and I didn't want to clutter (or lose track of) anything. Then i checked to ensure the `data.txt` file was there.

```bash
bandit12@bandit:/tmp/banditNygA$ xxd -r data.txt data
bandit12@bandit:/tmp/banditNygA$ ls -la
total 8
drwx------  2 bandit12 bandit12   80 Aug 12 00:42 .
drwxrwx-wt 77 root     root     1860 Aug 12 00:42 ..
-rw-rw-r--  1 bandit12 bandit12  613 Aug 12 00:42 data
-rw-r-----  1 bandit12 bandit12 2641 Aug 12 00:42 data.txt
```

The objective states `data.txt` is a hexdump, not the file itself, so before any decompression could happen I had to reverse the hexdump back into real binary bytes with `xxd -r`.

*From here, my recon process for every subsequent layer was the same three commands, repeated:*

```bash
file <current file>
mv <current file> data.<correct extension>
<decompress with matching tool>
```

- `file <current file>` identified true file type via magic bytes.
- `mv <current file> data.<correct extension>` renames the file so the right tool accepts it.
- `<decompress with matching tool>` like gunzip / bzip2 -d / tar -xf.

I didn't trust any file named tar or gzip that was handed to me (`data4.bin`, `data5.bin`, etc.) — each time, `file` was rerun immediately on whatever new file appeared before doing anything else with it.

## Solution

Layer 1 — identified as gzip, decompressed to reveal bzip2:

```bash
$ mv data data.gz
$ gunzip data.gz
$ file data
data: bzip2 compressed data, block size = 900k
```

Layer 2 — bzip2 → gzip:

```bash
$ mv data data.bz2
$ bzip2 -d data.bz2
$ file data
data: gzip compressed data, was "data4.bin", ...
```

Layer 3 — gzip → tar:

```bash
$ mv data data.gz
$ gzip -d data.gz
$ file data
data: POSIX tar archive (GNU)
```

Layer 4 — tar extraction produced a _new_ filename (`data5.bin`), so I checked it directly rather than assuming `data`:

```bash
$ mv data data.tar
$ tar -xf data.tar
$ ls -la
data5.bin
$ file data5.bin
data5.bin: POSIX tar archive (GNU)
```

(I hit one snag here — I first tried `mv data.bin data.tar` out of habit, which failed since the file was actually named `data5.bin`. `ls -la` caught the mistake immediately.)

Layer 5 — nested tar again:

```bash
$ mv data5.bin data.tar
$ tar -xf data.tar
$ ls -la
data6.bin
$ file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
```

Layer 6 — bzip2 → tar:

```bash
$ mv data6.bin data.bz2
$ bzip2 -d data.bz2
$ file data
data: POSIX tar archive (GNU)
```

Layer 7 — tar extraction produced `data8.bin`:

```bash
$ mv data data.tar
$ tar -xf data.tar
$ ls -la
data8.bin
$ file data8.bin
data8.bin: gzip compressed data, was "data9.bin", ...
```

Layer 8 — final gzip layer decompressed to plain text:

```bash
$ mv data8.bin data.gz
$ gzip -d data.gz
$ file data
data: ASCII text
```

Eight layers deep (gzip → bzip2 → gzip → tar → tar → bzip2 → tar → gzip), `file` finally reported plain text — that was my signal to stop unwrapping and just read it:

```bash
$ cat data
The password is [REDACTED]
```

## Verification

Authenticate as `bandit13` using the retrieved password:

```bash
[Laptop-3344]:~$ ssh bandit13@bandit.labs.overthewire.org -p 2220
bandit13@bandit.labs.overthewire.org's password: [REDACTED]
bandit13@bandit:~$ whoami
bandit13
```

**Explanation:** Initiating a new SSH session to port `2220` and supplying the retrieved password at the prompt grants access. Running `whoami` and observing the updated shell prompt (`bandit13@bandit:~$`) confirms successful authentication as `bandit13`.

## Why This Works

Every compressed/archive format starts with a distinctive magic-byte signature (gzip: `1f 8b`, bzip2: `42 5a 68` i.e. "BZh", tar: a `ustar` marker at a fixed offset). `file` reads these leading bytes directly and matches them against a known signature table — it never trusts the filename, extension, or what a prior tool happened to call the output, which is exactly why blindly renaming files without re-checking `file` output (as I briefly did with the `data.bin` mixup) causes errors. Chaining 8 layers of compression doesn't add real cryptographic security — each layer is a well-documented, keyless, publicly specified algorithm — it just adds mechanical tedium for whoever has to peel it back one step at a time.

## Key Takeaway

Never trust file extensions for identification, especially with untrusted input — validate by content/magic bytes the way `file`/libmagic does. Also worth internalizing operationally: when a process generates files with unpredictable names (like `tar -xf` did here), always re-`ls`/re-`file` after each step rather than assuming the next filename in sequence — that's precisely where the one hiccup in this run happened.

## Password

`[REDACTED - see level for retrieval method]`

---
