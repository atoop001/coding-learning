# Chapter 6: Environment & Configuration — PATH, Variables, Profiles & Aliases

## Overview

Ever wondered how typing `git` finds `C:\Program Files\Git\cmd\git.exe`? Or why a freshly installed tool gives "command not found" until you restart the terminal? The answers live in the **environment**: a set of named values every program inherits, chief among them **PATH**. This chapter demystifies environment variables, teaches you to read and change them safely, and then makes your shell *yours* — profiles, aliases, and functions that turn ten keystrokes into two. It closes with **processes and ports**: listing what's running, stopping a stuck one, and figuring out what's squatting on a port your dev server needs — the "address already in use" frustration every web developer hits early. Understanding this chapter permanently cures the most common categories of terminal confusion.

## Definitions & Explanations

**Environment variable** — A named string value (like `PATH`, `HOME`, `TEMP`) stored in a process's environment. When a shell starts a program, the program inherits a *copy* of the shell's variables. They configure behavior everywhere: `JAVA_HOME` tells tools where Java lives, `NODE_ENV` switches app modes, `EDITOR` picks your default editor.

**PATH** — The most important variable: a list of directories, separated by `;` on Windows and `:` on Linux. When you type a bare command name, the shell checks each PATH directory *in order* for a matching executable and runs the first hit. Not on PATH = "not recognized"/"command not found", even if the program is installed.

**Scopes of variables (Windows)** — Three layers:
1. *Process* — exists only in the current shell and its children; gone when the window closes. Set with `$env:NAME = "value"`.
2. *User* — stored in the registry for your account; applies to new processes you start.
3. *Machine* — system-wide, needs admin.
Changing User/Machine variables does **not** update already-running terminals — they keep their startup copy. This single fact explains most "I installed it but it's not found" mysteries.

**Shell variables vs. environment variables** — Both shells also have plain variables (`$x = 5` in PowerShell, `x=5` in Bash) that are *not* inherited by child programs. Bash promotes a variable to the environment with `export NAME=value`. In PowerShell, anything in `$env:` is environment; anything else (`$x`) is shell-only.

**Profile (startup script)** — A script your shell runs automatically at startup — the place to define aliases, functions, variables, and prompt customizations so they exist in every session.
- PowerShell: the path lives in the automatic variable `$PROFILE` (typically `...\Documents\PowerShell\Microsoft.PowerShell_profile.ps1`).
- Bash: `~/.bashrc` (interactive shells; the file for daily customization), with `~/.bash_profile` / `~/.profile` involved at login on some systems.

**Alias** — A short nickname for a command. PowerShell aliases (`Set-Alias g git`) can only rename a command — no arguments baked in. For "shortcut with arguments," PowerShell uses **functions**. Bash aliases *can* embed arguments: `alias gs='git status'`.

**Function** — A named block of shell code callable like a command. The right tool for multi-step shortcuts in both shells.

**Process** — A running instance of a program, identified by a numeric **PID** (process ID). Every terminal session, every dev server, every background tool is a process, and the OS tracks them all — same idea in both shells, different commands to see them.

**Port** — A numbered endpoint (0–65535) a process can *bind* to so network traffic can find it. A dev server "running on port 3000" means some process holds a listening socket on `3000`; only one process can hold a given port at a time, which is exactly why a second `npm run dev` fails with "address already in use."

**Execution policy (PowerShell/Windows only)** — A safety gate controlling whether `.ps1` files — including your own profile — are allowed to run at all. Fresh Windows machines often ship with `Restricted`, which silently blocks a freshly created profile from loading along with every other script. Check the current setting with `Get-ExecutionPolicy`; the standard developer fix, no admin required, is `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` — local scripts run freely, internet-downloaded ones must be signed or unblocked first. It's a seatbelt against accidental execution, not a security boundary. Chapter 8 revisits this in full alongside script authoring, but you need it now: if your profile changes in this chapter don't seem to take effect, an unset execution policy is a prime suspect.

## Command Examples

Inspecting the environment:

```powershell
# PowerShell
Get-ChildItem Env:                 # all environment variables
# Name       Value
# ----       -----
# APPDATA    C:\Users\atoop\AppData\Roaming
# PATH       C:\Windows\system32;C:\Program Files\Git\cmd;...
# TEMP       C:\Users\atoop\AppData\Local\Temp

$env:USERNAME                      # read one variable
# atoop

$env:PATH -split ';'               # PATH as a readable list, one dir per line
# C:\Windows\system32
# C:\Program Files\Git\cmd
# ...

Get-Command git                    # WHERE does 'git' resolve from?
# CommandType  Name     Source
# -----------  ----     ------
# Application  git.exe  C:\Program Files\Git\cmd\git.exe
```

```bash
# Bash
env                                # all environment variables (also: printenv)
echo $USER                         # read one
# atoop
echo $PATH | tr ':' '\n'           # PATH as a list
# /usr/local/bin
# /usr/bin
# ...
which git                          # where does 'git' resolve from?
# /usr/bin/git
type ls                            # even better: reveals aliases/functions too
# ls is aliased to `ls --color=auto'
```

Setting variables:

```powershell
# PowerShell — current session only (vanishes when window closes):
$env:API_URL = "http://localhost:3000"
$env:PATH += ";D:\tools\bin"           # append a dir to PATH, this session

# Persist for your user account (new shells see it; current ones do NOT):
[Environment]::SetEnvironmentVariable("API_URL", "http://localhost:3000", "User")

# Shell-only variable (NOT inherited by programs you launch):
$project = "cli-track"
```

```bash
# Bash — current session:
export API_URL="http://localhost:3000"
export PATH="$PATH:$HOME/tools/bin"    # append to PATH, this session

# Persist: add the export line to ~/.bashrc, then reload:
echo 'export PATH="$PATH:$HOME/tools/bin"' >> ~/.bashrc
source ~/.bashrc                       # re-run profile in current shell

# Shell-only (no export = children don't see it):
project="cli-track"
```

Creating and editing your profile:

```powershell
# PowerShell
$PROFILE                            # where is my profile?
# C:\Users\atoop\Documents\PowerShell\Microsoft.PowerShell_profile.ps1

Test-Path $PROFILE                  # exists yet?
# False
New-Item $PROFILE -Force            # create it (and parent folders)
notepad $PROFILE                    # open in an editor (or: code $PROFILE)

# --- Example profile contents ---
# Set-Alias g git
# function gs { git status }                 # alias-with-arguments = function
# function ll { Get-ChildItem -Force @args } # @args passes along any parameters
# function proj { Set-Location D:\atoop\coding-projects }
# $env:EDITOR = "code"
# --------------------------------

. $PROFILE                          # reload profile into current session (dot + space)
gs                                  # your new shortcut works
```

```bash
# Bash
nano ~/.bashrc                      # edit profile (nano: Ctrl+O save, Ctrl+X exit)

# --- Example ~/.bashrc additions ---
# alias gs='git status'
# alias ll='ls -lah'
# alias proj='cd ~/projects'
# export EDITOR=nano
# myip() { curl -s ifconfig.me; echo; }     # a function
# -----------------------------------

source ~/.bashrc                    # reload (or: . ~/.bashrc)
gs                                  # works
```

Diagnosing "command not found" like a pro:

```powershell
# The checklist, as commands:
Get-Command mytool                  # 1. Does the shell know it at all?
$env:PATH -split ';'                # 2. Is its folder in PATH?
Test-Path "D:\tools\mytool\mytool.exe"   # 3. Does the file actually exist there?
# If installed just now: open a NEW terminal (old ones have stale PATH).
```

Processes & ports — listing, stopping, and finding what's listening:

```powershell
# PowerShell — list and inspect processes
Get-Process                          # every running process
Get-Process -Name node                # filter by name
# Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
# -------  ------    -----      -----     ------     --  -- -----------
#     150      20   45000      52000       3.14    9124   1 node

Stop-Process -Id 9124                 # kill by PID
Stop-Process -Name node -Force        # kill by name (Force = don't prompt)

# Find what's listening on a port:
Get-NetTCPConnection -LocalPort 3000 -State Listen
# LocalAddress LocalPort ...  OwningProcess
# 0.0.0.0      3000      ...  9124
(Get-NetTCPConnection -LocalPort 3000).OwningProcess | Get-Process   # jump to the process

netstat -ano | findstr :3000          # older, universal: PID is the last column
# TCP    0.0.0.0:3000    0.0.0.0:0    LISTENING    9124
```

```bash
# Bash — list and inspect processes
ps aux                                # every running process
ps aux | grep node                    # filter by name

kill 9124                             # ask nicely (SIGTERM) by PID
kill -9 9124                          # force (SIGKILL) — last resort

# Find what's listening on a port:
lsof -i :3000                         # Unix/macOS/WSL: PID, command, user, all at once
# COMMAND   PID  USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
# node     9124  atoop  ...  TCP   *:3000 (LISTEN)

netstat -tulpn | grep :3000           # Linux alternative if lsof isn't installed
```

The full walkthrough — "port 3000 is taken":

```powershell
# 1. Start the dev server — it fails:
npm run dev
# Error: listen EADDRINUSE: address already in use :::3000

# 2. Find who's squatting on the port:
Get-NetTCPConnection -LocalPort 3000 -State Listen | Select-Object OwningProcess
# OwningProcess
# -------------
#          9124

# 3. Confirm it's what you think before killing it:
Get-Process -Id 9124
# Id     ProcessName
# --     -----------
# 9124   node

# 4. Kill it, then restart:
Stop-Process -Id 9124
npm run dev
# Server started on port 3000
```

```bash
# 1. Start the dev server — it fails:
npm run dev
# Error: listen EADDRINUSE: address already in use :::3000

# 2 & 3. Find and confirm who's squatting on the port:
lsof -i :3000
# COMMAND   PID  USER   ...  NAME
# node     9124  atoop  ...  *:3000 (LISTEN)

# 4. Kill it, then restart:
kill 9124
npm run dev
# Server started on port 3000
```

## Common Pitfalls

**Pitfall: "I installed it, but it's still not recognized."**
The installer updated the *User* PATH in the registry, but your open terminal inherited PATH at launch and never rereads it.
*Correction*: open a new terminal window (in Windows Terminal, a whole new window is safest — new tabs sometimes reuse the parent environment). If a new window still fails, the installer didn't add a PATH entry: find the executable's folder and add it yourself.

**Pitfall: destroying PATH instead of appending.**
`$env:PATH = "D:\tools\bin"` (or `export PATH="~/tools/bin"`) *replaces* the entire PATH — suddenly nothing works, not even `git`.
*Correction*: always append to the existing value: `$env:PATH += ";D:\tools\bin"` / `export PATH="$PATH:$HOME/tools/bin"`. If you break a session's PATH, just close and reopen the terminal — session changes don't persist. Be triply careful with the *persistent* setters; consider copying the current value somewhere first.

**Pitfall: editing the profile but seeing no change.**
You add an alias to `$PROFILE` or `~/.bashrc` and it "doesn't work" — because profiles run at *startup*, and your current session already started.
*Correction*: reload (`. $PROFILE` / `source ~/.bashrc`) or open a new terminal. Also confirm you edited the right file — `$PROFILE` differs between PowerShell 5.1 and 7, and Bash may use `.bash_profile` vs `.bashrc` depending on login mode.

**Pitfall: PowerShell alias with arguments.**
`Set-Alias gs "git status"` fails or misbehaves — PowerShell aliases map to a single command *name* only.
*Correction*: use a function: `function gs { git status }`. Rule of thumb: rename → alias; shortcut with arguments/steps → function.

**Pitfall: profile errors break every new shell.**
A typo in your profile makes every new terminal print red errors on startup.
*Correction*: profiles are just scripts — test additions by pasting them into a live session first. If startup is broken, launch with `pwsh -NoProfile` (or `bash --norc`), fix the file, reopen.

**Pitfall: expecting `$env:` changes to affect other windows or persist.**
Session variables are per-process copies; other terminals and future sessions never see them.
*Correction*: know your three scopes. Session experiments: `$env:` / `export`. Keep forever: profile file or `[Environment]::SetEnvironmentVariable(..., "User")`.

**Pitfall: killing the wrong PID (or needing elevation).**
PIDs are reused constantly — a PID from a tutorial screenshot, or from a command you ran five minutes ago, means nothing now; killing the wrong one can close your editor or an unrelated service. On Windows, some listening processes refuse `Stop-Process` without an elevated/admin terminal; on Linux, `kill` on another user's process needs `sudo`.
*Correction*: always look up the PID freshly with `Get-NetTCPConnection`/`lsof -i` immediately before killing — never reuse an old one. Sanity-check the process name (`Get-Process -Id`/`ps -p`) first. If the kill is refused, re-open the terminal as Administrator (Windows) or prefix with `sudo` (Linux) — but double-check the target before you do, since elevated kills have no undo.

## Practice Exercises

1. Print your PATH one directory per line in PowerShell. For each of the first five entries, note one executable that lives there (list the directory's `.exe` files). Then use `Get-Command` on three tools you use (e.g., `git`, `node`, `python`) and match each back to its PATH entry.
2. Set a session variable `PRACTICE_MODE=on`, prove a *child* process sees it (hint: start a nested `pwsh` and echo it, then `exit`), and prove a plain shell variable is NOT inherited the same way. Repeat the experiment in Bash with and without `export`.
3. Create your PowerShell profile if it doesn't exist. Add: one alias, one function that takes you to `D:\atoop\coding-projects`, and one function `ll` that lists files including hidden ones. Reload and test all three. Then do the Bash equivalents in `~/.bashrc` (Git Bash or WSL).
4. Simulate the classic failure: in one terminal, temporarily prepend a nonsense directory to your session PATH and confirm commands still work (why?). Then *replace* PATH entirely with just that nonsense directory and observe what breaks. Close the terminal, reopen, confirm everything is healed, and write two sentences explaining why no permanent harm occurred.
5. Create a folder `D:\tools\bin`, put a tiny script in it (a `.cmd` file containing `@echo hello from my tool` is enough), add that folder to your session PATH, and run your "tool" by bare name from a different directory. Bonus: make it permanent with the User-scope setter, open a new terminal, and verify.
6. Start a dev server on port 3000 (any framework, or `py -m http.server 3000` / `npx serve -l 3000`). From a second terminal: list the process by name, find its PID via the port, and confirm the two match. Do it once with `Get-NetTCPConnection`/`Get-Process` and once with `netstat -ano`/`findstr` in PowerShell; once with `lsof`/`ps` in Bash (WSL or Git Bash).
7. Deliberately trigger "address already in use": start the same dev server twice in two terminals without stopping the first. Read the error, find the PID occupying the port, kill only that process, and successfully start the second instance. Write down the exact sequence of commands you ran.
