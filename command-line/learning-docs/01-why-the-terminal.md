# Chapter 1: Why the Terminal — Orientation and First Commands

## Overview

The terminal (also called the command line, console, or shell) is a text-based way of talking to your computer. Instead of clicking icons, you type commands and read text responses. Every serious development workflow eventually runs through it: installing tools, running builds, using git, deploying to servers, debugging production machines. GUIs come and go per tool; the terminal is the common language.

Why bother when a GUI exists?

- **Speed**: typing `git status` beats hunting through menus.
- **Automation**: anything you can type, you can script and repeat. A GUI click cannot be saved and replayed; a command can.
- **Remote work**: Linux servers usually have *no* GUI. SSH gives you a terminal — that's it. If you can't work in a shell, you can't work on a server.
- **Precision and reproducibility**: "run these 4 commands" is an exact recipe; "click the third tab, then the gear icon" is not.
- **Tool access**: many developer tools (npm, pip, cargo, docker, kubectl) are CLI-first or CLI-only.

This track teaches **PowerShell** as your daily driver (it's the native modern shell on Windows) with **Bash** equivalents throughout, because Bash is what you'll meet on Linux servers, macOS, CI systems, and Git Bash.

## Definitions & Explanations

**Terminal (terminal emulator)** — The *window* that displays text and accepts keystrokes. Windows Terminal, the old console host (conhost), iTerm2 on macOS — these are terminals. The terminal itself doesn't understand commands; it just hosts a shell.

**Shell** — The *program running inside the terminal* that reads your commands, interprets them, and runs programs. PowerShell, cmd.exe, Bash, and Zsh are all shells. Terminal : Shell :: TV : Channel. You can run different shells inside the same terminal.

**PowerShell** — Microsoft's modern shell and scripting language. Two flavors exist:
- *Windows PowerShell 5.1* (`powershell.exe`) — ships with Windows, frozen in time.
- *PowerShell 7+* (`pwsh.exe`) — the actively developed, cross-platform version. Prefer this; install it if you haven't.
PowerShell's superpower: commands pass **objects** (structured data) to each other, not just text. More on that in Chapter 5.

**cmd.exe (Command Prompt)** — The legacy Windows shell, descended from MS-DOS. You'll occasionally see `.bat` files or old tutorials using it. Know it exists; don't invest in it. Commands like `dir` and `copy` live here (though PowerShell accepts many of them as aliases).

**Bash (Bourne Again Shell)** — The default shell on most Linux distributions and the standard for servers, Docker containers, and CI pipelines. On Windows you can run Bash through **WSL** (Windows Subsystem for Linux — a real Linux environment inside Windows, Chapter 10) or **Git Bash** (a lightweight Bash environment installed with Git for Windows).

**Prompt** — The text the shell prints when it's ready for input. Examples:

- PowerShell: `PS D:\atoop\coding-projects>`
- Bash: `user@machine:~/projects$`

The prompt usually shows *where you are* (current directory). `~` in Bash means your home directory. A `#` ending on Linux means you're root (administrator) — be careful. When you see a prompt in documentation (`PS>`, `$`), you don't type the prompt itself, only the command after it.

**Command anatomy** — Most commands look like:

```
command  -flag  --long-option  argument
```

- *Command*: the program or built-in to run (`git`, `Get-ChildItem`).
- *Flags/options*: modify behavior (`-r`, `--verbose`, `-Recurse`).
- *Arguments*: what to operate on (a file name, a URL).

**Cmdlet** — PowerShell's name for its built-in commands, always `Verb-Noun`: `Get-ChildItem`, `Set-Location`, `Remove-Item`. Verbose but self-describing. Most have short **aliases** (`ls`, `cd`, `rm`) so day-to-day typing is fast.

**Case sensitivity** — PowerShell and Windows paths are case-*insensitive* (`Get-ChildItem` = `get-childitem`, `D:\Code` = `d:\code`). Bash and Linux are case-*sensitive* (`File.txt` and `file.txt` are different files; `ls` works, `LS` does not). This bites Windows users constantly on servers — start caring about case now.

## Command Examples

Open Windows Terminal (Win key, type "terminal"). It should default to PowerShell.

```powershell
# Who and where am I?
whoami
# Expected output (yours will differ):
# desktop-abc123\atoop

Get-Location          # print current directory ("pwd" also works)
# Path
# ----
# C:\Users\atoop

# What shell and version is this?
$PSVersionTable.PSVersion
# Major  Minor  Patch
# -----  -----  -----
# 7      4      1        <- 7.x means modern PowerShell; 5.1 means the legacy one

# List what's in the current folder
Get-ChildItem         # aliases: ls, dir, gci
#     Directory: C:\Users\atoop
# Mode     LastWriteTime      Length Name
# ----     -------------      ------ ----
# d----    2026-07-01 09:12          Documents
# d----    2026-07-01 09:12          Downloads
# -a---    2026-06-15 14:03     1024 notes.txt

# Print text to the screen
Write-Output "Hello, terminal"    # alias: echo
# Hello, terminal

# Clear the screen when it gets cluttered
Clear-Host            # alias: clear or cls

# Get help on any command (the most important habit to build)
Get-Help Get-ChildItem -Examples
# Shows usage examples for the command
```

The same session in Bash (Git Bash or WSL):

```bash
whoami
# atoop

pwd                   # print working directory
# /home/atoop

bash --version        # what shell version?
# GNU bash, version 5.2.21(1)-release ...

ls                    # list current folder
# Documents  Downloads  notes.txt

echo "Hello, terminal"
# Hello, terminal

clear                 # clear the screen

man ls                # manual page for ls; press q to quit
# (full-screen help text)
```

Running a program vs. running a command:

```powershell
# Anything on your PATH (Chapter 6) runs by name:
git --version
# git version 2.45.0.windows.1

# If you get an error instead:
# git : The term 'git' is not recognized as the name of a cmdlet...
# ...it means the program isn't installed or isn't on your PATH. Chapter 6 explains why.
```

Useful keyboard survival skills (both shells):

```text
Up / Down arrows   - cycle through previous commands
Tab                - autocomplete file names and commands (Chapter 2)
Ctrl+C             - cancel the currently running command
Ctrl+L             - clear screen (Bash and PS7)
Ctrl+R             - search command history as you type (try it!)
exit               - close the shell
```

## Common Pitfalls

**Pitfall: confusing terminal, shell, and Windows Terminal profiles.**
Windows Terminal can open tabs running PowerShell, cmd, or WSL Bash — each tab is a different shell. If a tutorial's commands fail bizarrely (`ls: command not found` in cmd, or `dir /s` erroring in Bash), you're likely in the wrong shell. Check the tab title or the prompt style.
*Correction*: look at the prompt. `PS ...>` is PowerShell, `C:\...>` is cmd, `user@host:...$` is Bash.

**Pitfall: typing the prompt from a tutorial.**
Docs write `$ git status` or `PS> ls`. Typing the `$` or `PS>` yourself produces errors.
*Correction*: type only the command after the prompt symbol.

**Pitfall: copy-pasting commands you don't understand.**
Random internet commands can delete files or change system settings. This is the terminal equivalent of running an unknown .exe.
*Correction*: before running anything with `Remove-Item`, `rm`, `Set-`, `sudo`, or a pipe to a shell (`| iex`, `| bash`), read it, and use `Get-Help <command>` or `man <command>` to understand it.

**Pitfall: "The term 'X' is not recognized" / "command not found" panic.**
This is the single most common beginner error. It means the shell searched its PATH list and didn't find a program with that name. It does *not* mean your computer is broken.
*Correction*: check spelling first, then whether the tool is installed, then whether the terminal was opened *before* the tool was installed (restart the terminal — PATH is read at startup). Full story in Chapter 6.

**Pitfall: assuming Windows habits work on Linux.**
Backslashes in paths, case-insensitive names, and `C:\` drives are Windows-only. Linux uses forward slashes, is case-sensitive, and has one root `/`.
*Correction*: whenever both are shown in this track, read both. The Bash column is your future server life.

## Practice Exercises

1. Open Windows Terminal. Create one tab running PowerShell and (if Git for Windows is installed) one running Git Bash. In each, run the command that prints your current directory and the command that prints your username. Note how the prompts differ.
2. In PowerShell, run `$PSVersionTable.PSVersion`. If the major version is 5, use the internet (or `winget search PowerShell`) to find out how you'd install PowerShell 7 — you'll do the install in Chapter 7.
3. Use `Get-Help Get-Date -Examples` in PowerShell to figure out how to print only the current year. Then find the Bash equivalent using `man date` (hint: `date` takes a `+FORMAT` argument).
4. Press the Up arrow to re-run your last command, then use Ctrl+R and type a fragment of an earlier command to recall it. Practice until recalling history feels faster than retyping.
5. In your own words (a text file or notebook), write one sentence each defining: terminal, shell, prompt, cmdlet, alias, PATH. You'll refine these definitions as the track continues.
