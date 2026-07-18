# Chapter 10: WSL & Remote Basics — Linux, SSH & Permissions

## Overview

Everything so far ran on your own machine. Real development eventually reaches machines you *can't* sit at: Linux servers in the cloud, deployment targets, CI runners. This final chapter closes the loop: **WSL** gives you a genuine Linux environment inside Windows (your practice server, zero risk), **SSH** is how you reach remote machines, and **Linux file permissions** (`chmod`, `chown`, `sudo`) are the access-control model every server enforces. Finish this chapter and the Bash column of every previous chapter becomes your working reality, not just a comparison.

## Definitions & Explanations

**WSL (Windows Subsystem for Linux)** — A Windows feature running a real Linux distribution (default: Ubuntu) alongside Windows — actual `apt`, real permissions, genuine server-identical tooling, without dual-booting or a VM app. Version 2 (WSL2, the default) runs a true Linux kernel.

**Filesystem bridges** — Two worlds, two views:
- From Linux, your Windows drives appear under `/mnt/c/`, `/mnt/d/` (e.g., `D:\atoop` → `/mnt/d/atoop`).
- From Windows, the Linux filesystem appears at the `\\wsl$\Ubuntu\` network path.
Performance rule: keep Linux work (builds, git repos used from Linux) *in the Linux filesystem* (`~/...`), not on `/mnt/c` — cross-boundary file access is much slower.

**SSH (Secure Shell)** — Encrypted remote login: `ssh user@host` gives you a shell *on the remote machine*. Every command you then type runs there, not locally. Windows 10/11 ship a native OpenSSH client, so `ssh` works in PowerShell out of the box.

**SSH keys** — The professional alternative to passwords: a **key pair** — private key (secret, stays on your machine, never shared, never emailed) and public key (freely copied to servers, into `~/.ssh/authorized_keys`). Possession of the private key proves identity: no password prompts, immune to password guessing. Generate with `ssh-keygen -t ed25519`.

**`~/.ssh/config`** — A local file of per-host connection settings, so `ssh myserver` replaces `ssh -i C:\Users\atoop\.ssh\special_key admin@203.0.113.10 -p 2222`.

**scp / sftp** — File transfer over SSH: `scp` copies files between machines with `cp`-like syntax (`scp local.txt user@host:/remote/path`).

**Linux permissions** — Every file has an **owner**, a **group**, and three permission sets — owner/group/others — each with **r**ead, **w**rite, e**x**ecute bits. `ls -l` shows them as `-rwxr-xr--`: type (`-` file, `d` dir), then rwx for owner, group, others. On directories, `x` means "may enter," `r` means "may list."

**Octal notation** — Each rwx triplet as a digit: r=4, w=2, x=1 summed. `rwx`=7, `rx`=5, `r`=4. So `chmod 755` = owner everything, everyone else read+enter/execute — the standard for scripts and directories. `644` = owner read/write, others read — the standard for ordinary files. `600` = owner-only — required for private keys.

**chmod / chown** — `chmod` changes permission bits (`chmod +x script.sh`, `chmod 600 key`); `chown` changes owner/group (`sudo chown atoop:atoop file`).

**root and sudo** — `root` is the Linux administrator. `sudo <cmd>` runs one command as root (you'll be asked for *your* password). System changes (installing packages, editing `/etc`, touching other users' files) need it; ordinary work in your home directory must not use it.

## Command Examples

Installing and entering WSL:

```powershell
# PowerShell (admin recommended for first install)
wsl --install                  # installs WSL2 + Ubuntu; reboot may be required
# ... after reboot, Ubuntu opens and asks you to create a Linux username/password
# (this password is what sudo will ask for)

wsl --list --verbose           # what's installed and running?
#   NAME      STATE     VERSION
# * Ubuntu    Running   2

wsl                            # enter Linux from any PowerShell prompt
# atoop@machine:/mnt/d/atoop/coding-projects$   <- note: dropped in at your Windows cwd
exit                           # back to PowerShell
```

First minutes inside Linux — orienting with skills you already have:

```bash
pwd                            # where am I?
cd ~                           # go to the LINUX home (not /mnt/c/Users/...)
sudo apt update && sudo apt upgrade -y     # refresh & update packages (Ch. 7)
sudo apt install -y tree htop  # install some tools
tree -L 2 ~                    # look around
ls /mnt/d/atoop/coding-projects   # your Windows D: drive, from Linux
```

Reading and changing permissions:

```bash
touch demo.sh && mkdir data
ls -l
# -rw-r--r-- 1 atoop atoop    0 Jul 18 10:00 demo.sh
# drwxr-xr-x 2 atoop atoop 4096 Jul 18 10:00 data
#  ^^^ ^^ ^^
#  |   |  \_ others: r-- (read only)
#  |   \____ group:  r-- / r-x
#  \________ owner:  rw- / rwx

./demo.sh
# bash: ./demo.sh: Permission denied      <- no execute bit yet!

chmod +x demo.sh               # add execute for everyone
ls -l demo.sh
# -rwxr-xr-x 1 atoop atoop 0 Jul 18 10:00 demo.sh
./demo.sh                      # now runs (does nothing — it's empty)

chmod 600 secrets.txt          # owner read/write only — for private material
chmod 644 notes.txt            # owner rw, world readable — normal file
chmod 755 bin/                 # standard directory/script permissions
chmod -R u+w data/             # recursive: give owner write throughout

sudo chown atoop:atoop stray-file    # take ownership (needs sudo)
```

SSH keys and connecting (works from PowerShell *and* WSL — Windows has native OpenSSH):

```powershell
# 1. Generate a key pair (one-time per machine):
ssh-keygen -t ed25519 -C "atoop laptop 2026"
# Enter file in which to save the key (C:\Users\atoop\.ssh\id_ed25519): <Enter>
# Enter passphrase: <optional but recommended>
# Your public key has been saved in C:\Users\atoop\.ssh\id_ed25519.pub

Get-Content ~\.ssh\id_ed25519.pub    # the PUBLIC half — safe to share
# ssh-ed25519 AAAAC3NzaC1... atoop laptop 2026

# 2. Connect to a server (first contact asks you to trust the host):
ssh atoop@203.0.113.10
# The authenticity of host '203.0.113.10' can't be established.
# ED25519 key fingerprint is SHA256:...
# Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
# atoop@server:~$        <- you are now ON the server; Bash chapters apply!

# 3. Put your public key on the server so future logins skip the password:
#    (from Linux/WSL: ssh-copy-id atoop@203.0.113.10)
#    From PowerShell, append the .pub content to the server's file manually:
type ~\.ssh\id_ed25519.pub | ssh atoop@203.0.113.10 "cat >> ~/.ssh/authorized_keys"

# 4. Run one remote command without an interactive session:
ssh atoop@203.0.113.10 "df -h /"
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/vda1        50G   12G   38G  24% /

# 5. Copy files:
scp .\report.pdf atoop@203.0.113.10:/home/atoop/     # local -> remote
scp atoop@203.0.113.10:/var/log/app.log .\logs\      # remote -> local
```

`~/.ssh/config` — stop retyping connection details:

```text
# File: C:\Users\atoop\.ssh\config   (and/or ~/.ssh/config in WSL)
Host myserver
    HostName 203.0.113.10
    User atoop
    IdentityFile ~/.ssh/id_ed25519

Host work-box
    HostName build.internal.example.com
    User deploy
    Port 2222
```

```powershell
ssh myserver                   # all details supplied from the config
scp data.csv myserver:~/data/  # aliases work everywhere ssh does
```

No server yet? Practice against WSL — it *is* a Linux box:

```bash
# Inside WSL:
sudo apt install -y openssh-server
sudo service ssh start
ip addr | grep "inet "         # find the WSL IP (e.g. 172.x.x.x)
# Then from PowerShell:  ssh <linux-user>@<that-ip>
# You are now "remoting" into a Linux machine — the full workflow, zero cost.
```

## Common Pitfalls

**Pitfall: `Permission denied` on your own script.**
Fresh files aren't executable; `./script.sh` fails even though *you own it*.
*Correction*: `chmod +x script.sh` once, or run via `bash script.sh` (interpreter mode needs only read permission). This is Chapter 8's rule shown in its native habitat.

**Pitfall: sudo-ing everything.**
Blocked once, some learners prepend `sudo` to every command. Then files in your own home are owned by root, later un-writable *without* sudo, and mistakes hit with admin power (`sudo rm -rf` in the wrong place is unrecoverable, system-wide).
*Correction*: `sudo` only for genuinely system-level actions (apt, services, `/etc`). If a file in *your* space needs sudo, its ownership is wrong — fix with `sudo chown $USER:$USER file` once, not with sudo forever. See `whoami` print `root`? Leave that shell.

**Pitfall: private key leaked or mishandled.**
Pasting `id_ed25519` (no `.pub`) into a ticket/chat, committing it to git, or copying it to a server "to make things work."
*Correction*: the private key never leaves your machine — only `.pub` travels. If a private key may have leaked: treat it as compromised, generate a new pair, remove the old public key from every `authorized_keys`. On Linux, ssh itself will refuse keys with loose permissions — keep `chmod 600 ~/.ssh/id_ed25519` and `chmod 700 ~/.ssh`.

**Pitfall: editing Linux-side files with Windows tools (and vice versa) carelessly.**
Editing files inside `\\wsl$\...` with random Windows apps, or running heavy git/npm work on `/mnt/c`, causes CRLF damage (Ch. 8) and dramatic slowness.
*Correction*: keep Linux projects in the Linux filesystem and edit with Linux-aware tools (VS Code's Remote-WSL does this properly). Use `/mnt/...` for *transferring* files, not for hosting active Linux projects.

**Pitfall: "I ran the command, but nothing changed on the server" — you were local (or vice versa).**
With several terminal tabs open, it's easy to run a destructive command in the wrong world.
*Correction*: read the prompt before every consequential command — hostname is in it (`atoop@server` vs `atoop@laptop` vs `PS D:\...`). Set distinct Windows Terminal color schemes per profile so local/WSL/SSH *look* different at a glance.

**Pitfall: locked out by a typo in ssh config or authorized_keys.**
A malformed `authorized_keys` line or wrong permissions on `~/.ssh` silently disables key login.
*Correction*: always keep one working session open while changing SSH settings; test new configuration from a *second* terminal before closing the first. `ssh -v myserver` (verbose) shows exactly why an authentication attempt failed.

## Practice Exercises

1. Install WSL and complete first-boot setup. From inside Linux: update packages, install `tree` and `htop`, and list your Windows `coding-projects` folder via `/mnt/`. From PowerShell: open your Linux home via `\\wsl$` in `explorer.exe .` — map the two bridges in a short note.
2. Permissions lab (in WSL): create `perms-lab/` with a script, a "secret" file, and a subdirectory. Set: script `755`, secret `600`, directory `700`. Verify each with `ls -l`, then prove the settings bite: try to run the script before/after `+x`, and check the secret's visibility by attempting `sudo -u nobody cat secret.txt` (expect denial).
3. Generate an ed25519 key pair in PowerShell. Identify which file is private and which is public, view the public one, and write one sentence on where each is allowed to go. Create `~/.ssh/config` with at least one Host alias (target your WSL instance or any machine you have).
4. Turn WSL into your practice server: install and start `openssh-server`, then ssh into it *from PowerShell*, first by password, then — after installing your public key into `~/.ssh/authorized_keys` — by key with no password prompt. Finish by running one non-interactive remote command (`ssh <alias> "uname -a"`) and one `scp` transfer in each direction.
5. Capstone drill — a full remote session using only earlier-chapter skills, all inside one ssh session to WSL: create a project directory, write a two-line Bash script with a heredoc or `nano`, `chmod` it, run it, redirect its output to a log, grep the log, and schedule it in crontab to run every 5 minutes; verify it fired twice, then remove the crontab entry. You have now operated a Linux server end to end.
