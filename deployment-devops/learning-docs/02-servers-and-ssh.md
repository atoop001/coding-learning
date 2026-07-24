# Chapter 2: Servers & SSH

## Overview

Somewhere under every URL is a computer running Linux. Even if you spend your whole career deploying through friendly platforms that hide it, you will constantly bump into the machine underneath: in log output, in error messages, in interviews, and the first time something breaks in a way the platform's dashboard can't explain. This chapter teaches you what a server actually is, how professionals connect to one securely with SSH, and enough Linux-side navigation to not be helpless on one. Then it makes an important, honest argument: **you should almost certainly not host your first projects on a raw rented server** — a Platform-as-a-Service will do it better — but the entire value of the PaaS is that it automates things you're about to learn by hand. Understanding what the PaaS does *for* you is the difference between using it and being mystified by it.

## Definitions & Explanations

**Server (the machine)** — a computer optimized for running unattended: no monitor, no GUI, sitting in a data center with redundant power and fast networking, administered entirely over the network via text commands. Almost always Linux (Ubuntu and Debian are the common distributions), because Linux is free, stable, scriptable, and what the entire hosting ecosystem assumes. This is why the `command-line` track was a prerequisite — a server is just a shell prompt you reach over the network.

**VPS (Virtual Private Server)** — the standard rentable unit: a virtual machine (a software-simulated computer) carved out of a bigger physical machine, rented for a few dollars a month from providers like DigitalOcean, Hetzner, or Linode. You get root access and total freedom — which includes the freedom to misconfigure security and have your box conscripted into a botnet.

**SSH (Secure Shell)** — the protocol for opening an encrypted remote terminal session on another machine. `ssh user@host` gives you a shell prompt that is *on the server*: every command you type runs there, not on your laptop. SSH is also the transport under `scp` (file copy), `git push` over SSH remotes, and most deployment tooling. Windows 11 ships an SSH client in PowerShell natively — no extra installs.

**SSH key pair** — the professional alternative to passwords. You generate two mathematically linked files: a **private key** (stays on your machine, is never shared, ever) and a **public key** (freely given out, placed on servers you want to access). The server challenges your client to prove it holds the private key matching an authorized public key. You already used this pattern for GitHub in the git track — GitHub's servers hold your public key; pushing proves you hold the private one. Same mechanism, now for logging into machines.

**`authorized_keys`** — a file on the server (`~/.ssh/authorized_keys`) listing public keys allowed to log in as that user. Adding your public key to it is "granting access"; deleting the line is "revoking access."

**Root** — the Linux superuser, able to do anything including destroy the system. Good practice is to log in as a normal user and elevate specific commands with **`sudo`** ("superuser do"), which both limits blast radius and leaves an audit trail.

**Daemon / service** — a program that runs in the background, detached from any terminal, started at boot. Your deployed app must run as one — otherwise it dies the moment your SSH session closes. On modern Linux, **systemd** is the manager that starts services, restarts them when they crash, and captures their logs.

**Process manager** — anything that keeps your app process alive and restarts it on failure: systemd, or Node-specific tools like PM2. "Who restarts my app when it crashes at 3 a.m.?" must always have an answer.

**Reverse proxy** — a front-door program (usually **nginx** or **Caddy**) that accepts public traffic on ports 80/443, handles HTTPS, and forwards requests to your app listening privately on a port like 3000. Nearly every serious server deployment has one; PaaS platforms run one for you invisibly.

**Firewall** — rules controlling which network ports accept connections. A sane server allows 22 (SSH), 80 (HTTP), 443 (HTTPS) and blocks everything else, so your database port isn't exposed to the whole internet.

**`known_hosts`** — the client-side mirror of `authorized_keys`: a file on *your* machine (`~/.ssh/known_hosts`) recording the fingerprint of every server you've connected to. It's what powers the first-connection trust prompt and the loud warning if a known server's identity ever changes. `authorized_keys` answers "who may log into this server"; `known_hosts` answers "is this server who it was last time."

**ssh-agent** — a small background helper that holds your decrypted private key in memory for the session, so a passphrase-protected key doesn't demand the passphrase on every single connection. Windows 11 ships it as a service (`ssh-agent`); you load a key into it once with `ssh-add`. This is the mechanism that makes "passphrase on the key" livable rather than annoying — use both.

**Port 22** — SSH's standard port, and consequently the most attacked port on the internet. Some admins move SSH to a nonstandard port to cut log noise; it's obscurity, not security — keys-only auth is the actual defense.

**PaaS (Platform as a Service)** — Render, Railway, Fly.io and kin. You connect a GitHub repo; the platform provisions machines, builds your code, runs it under a process manager, terminates HTTPS behind its reverse proxy, injects your configuration, streams your logs to a dashboard, and redeploys on every push. Every clause in that sentence is a chore you'd otherwise do by hand on a VPS. That's the deal: less control and less learning-by-necessity, in exchange for skipping a hundred sysadmin tasks that are not your job as an application developer.

**Why start with a PaaS anyway?** Because on a VPS, *you* are the security team, the patch manager, the backup operator, and the 3 a.m. pager. None of that teaches web development, and unpatched beginner VPSes get compromised routinely (bots scan the entire internet for weak SSH configs continuously — connect a fresh server and watch the auth logs fill with strangers' login attempts within hours). The professional-grade skill is knowing what the platform automates, so you can debug it when it leaks and graduate to raw infrastructure (Chapter 9) when a job requires it.

## Code Examples

Windows 11's built-in OpenSSH, from PowerShell:

```powershell
# Confirm the SSH client exists (ships with Windows 10+):
ssh -V

# Generate a key pair. Ed25519 is the modern, recommended algorithm.
# Accept the default location; DO set a passphrase (it encrypts the private key on disk).
ssh-keygen -t ed25519 -C "your-email@example.com"

# Your keys now live here:
Get-ChildItem ~\.ssh
#   id_ed25519       <- PRIVATE. Never leaves this machine. Never in a repo. Never in a chat.
#   id_ed25519.pub   <- public. Safe to paste anywhere.

# Print the public key (this is what you paste into a server or a hosting dashboard):
Get-Content ~\.ssh\id_ed25519.pub
```

Connecting and moving files (usable against any Linux box — a free-tier cloud VM, a friend's server, or a WSL2 sshd if you set one up to practice):

```powershell
# Open a remote shell. First connection asks you to trust the host's fingerprint.
ssh yourname@203.0.113.10        # or: ssh yourname@server.example.com

# Run a single command remotely without an interactive session:
ssh yourname@203.0.113.10 "uptime"

# Copy a file up / down (scp = secure copy over SSH):
scp .\site.zip yourname@203.0.113.10:/home/yourname/
scp yourname@203.0.113.10:/var/log/myapp.log .\myapp.log
```

Once you're *on* a server, you're in bash, not PowerShell — the command-line track's Unix material applies directly:

```bash
# Orientation ritual for any unfamiliar server:
whoami                # which user am I?
pwd                   # where am I? (usually /home/<user>)
df -h                 # disk space — full disks cause bizarre failures
free -h               # memory
sudo systemctl status nginx     # is the web server service running?
sudo journalctl -u myapp --since "1 hour ago"   # logs for a systemd service
ss -tlnp              # what's listening on which ports?
```

A convenience worth adopting early — the SSH config file, so you never retype connection details:

```text
# File: C:\Users\<you>\.ssh\config   (no extension; plain text)
Host myserver
    HostName 203.0.113.10
    User yourname
    IdentityFile ~/.ssh/id_ed25519
```

```powershell
# Now this works:
ssh myserver
```

And the contrast that motivates the PaaS: here's the *abbreviated* checklist to host one Node app on a raw VPS —

```text
create server -> create non-root user -> install your public key -> disable password login
-> configure firewall -> install Node -> clone repo -> npm install -> write a systemd unit
-> enable & start the service -> install nginx -> write a reverse-proxy config
-> point DNS -> obtain a TLS certificate -> set up log rotation -> schedule OS updates
```

versus on Render/Railway: *connect repo, set start command, click deploy.* Both paths end with the same thing running. Chapter 9 returns to when the long path is worth it.

Setting up the Windows ssh-agent so your passphrase-protected key is pleasant to use:

```powershell
# One-time setup: the agent service is disabled by default on Windows.
# (Run these two in an elevated/Administrator PowerShell.)
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent

# Then, in any normal shell — load your key into the agent (asks the passphrase ONCE):
ssh-add ~\.ssh\id_ed25519

# Verify what the agent is holding:
ssh-add -l

# From now on, every ssh/scp/git-over-ssh in this login session uses the loaded
# key silently. Passphrase protection with none of the friction.
```

What a session's environment tells you — a tiny drill in "where am I?" awareness, because you'll have PowerShell, WSL2, and remote shells all open at once and they look similar:

```powershell
# PowerShell (local Windows):
$env:COMPUTERNAME          # your PC's name
```

```bash
# WSL2 or a remote server (bash):
hostname                   # which machine is this shell actually on?
uname -a                   # Linux kernel info — proves you're not in Kansas/Windows anymore
echo $SHELL                # which shell program
```

Make `hostname` a reflex before running anything destructive. "Wrong terminal window" is a real category of production incident, and the cure costs one word.

A first look at systemd, since its vocabulary shows up in every server log you'll ever read — this is what a service definition looks like (read it; you don't need to write one until you do the VPS side quest):

```text
# /etc/systemd/system/myapp.service  — "keep my app running, forever"
[Unit]
Description=My Node app
After=network.target                 # start after networking is up

[Service]
User=appuser                         # NOT root — Chapter 2's whole sermon
WorkingDirectory=/home/appuser/myapp
ExecStart=/usr/bin/node server.js    # the actual process
Restart=always                       # crashed? start it again. The 3 a.m. answer.
Environment=NODE_ENV=production      # env vars — Chapter 3's contract, server edition

[Install]
WantedBy=multi-user.target           # start at boot
```

```bash
# ...and how you'd drive it:
sudo systemctl enable --now myapp    # register at boot + start right now
systemctl status myapp               # running? since when? recent log lines
sudo journalctl -u myapp -f          # follow its logs live
```

Every line of that file is a thing Render/Railway/Fly does for you invisibly. When their dashboard shows "service restarted," this is the machinery underneath.

## Common Pitfalls

1. **Sharing or committing the private key.** The `.pub` file is public; the other one is the crown jewels. Pasting `id_ed25519` (no extension) anywhere — a repo, a gist, a Discord message — means anyone who saw it can be you. Correction: only ever paste the `.pub` file; if the private key leaks, delete the pair, generate a new one, and update `authorized_keys`/dashboards everywhere the old public key lived.

2. **Running everything as root because permissions are annoying.** One mistyped `rm -rf` as root ends the server; a compromised root-run app owns the whole box. Correction: normal user + `sudo` per command. If a tutorial says "just log in as root," treat the tutorial with suspicion generally.

3. **Starting an app over SSH and wondering why it died when you logged out.** Processes launched in a session are children of that session; closing it kills them. Correction: real deployments run under a process manager (systemd, PM2, or the PaaS's supervisor). `node server.js` in an SSH window is a demo, not a deployment.

4. **Editing files live on the server instead of deploying.** SSH in, hand-edit `server.js`, restart — now the server disagrees with Git and the next deploy silently erases the fix. Correction: the server only ever receives code through the deploy process. SSH is for *inspecting*, not authoring.

5. **Leaving password authentication enabled on a public server.** Internet-wide bots brute-force SSH passwords around the clock. Correction: keys only (`PasswordAuthentication no` in the server's sshd config) — and let this be one more nudge toward PaaS, where none of this is your problem.

6. **Choosing a VPS "to learn faster" and stalling for a month.** The failure mode is real: three weekends into nginx configs, no project shipped, motivation gone. Correction: PaaS first, ship things, then do this chapter's VPS material as a *bounded side quest* (exercise 6) rather than the main road.

7. **Ignoring the host fingerprint prompt's meaning.** That first-connection "authenticity of host can't be established" prompt is SSH protecting you from connecting to an impostor machine. Correction: on first connect it's normally fine to accept; but if it reappears *for a server you've used before* with a "HOST IDENTIFICATION HAS CHANGED" warning, stop and find out why (server rebuilt? or man-in-the-middle?).

## Practice Exercises

1. **Key ceremony.** Generate an Ed25519 key pair (with a passphrase). Locate both files, print the public one, and write two sentences: what the server stores, and what gets proven at login time without the private key ever being transmitted. Add the public key to your GitHub account (Settings → SSH keys) and verify with `ssh -T git@github.com`.

2. **Free shell practice.** Get a bash prompt you can treat as a "server": easiest is your existing WSL2 Ubuntu (`wsl` from PowerShell). Run the orientation ritual from this chapter (whoami, pwd, df, free, ss) and write one line about what each told you.

3. **Session-death demo.** In WSL2, start `python3 -m http.server 8080`, confirm it responds, then close the terminal window entirely and check again. Explain in your notes exactly why this proves you need a process manager, and name two things that fill that role.

4. **SSH config file.** Create `~/.ssh/config` with a `Host` alias (GitHub works: alias `gh` for `github.com` with your key) and demonstrate the short form connecting.

5. **PaaS X-ray.** Read the docs homepage of one PaaS (Render, Railway, or Fly.io) and produce a two-column table: left, each thing the platform says it handles (builds, HTTPS, restarts, logs, scaling); right, the raw-server tool or task from this chapter it replaces (nginx, systemd, journalctl, ...). This table *is* the chapter's thesis — keep it.

6. **(Optional side quest, timeboxed to one weekend.)** If a provider offers you a free/cheap VPS, do the full checklist once: non-root user, key-only SSH, firewall, one static page behind nginx. Then tear it down. Once is genuinely enough — the point is to have felt everything the PaaS will do for you from here on.
