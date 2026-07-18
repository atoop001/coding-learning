# Chapter 1: What Version Control Is & Setting Up Git on Windows

## Overview

Version control is a system that records changes to files over time so you can recall specific versions later, see who changed what and why, and experiment without fear of destroying your work. Git is the dominant version control system in the software industry — nearly every professional team uses it, and GitHub (a hosting service built around Git) is where most open-source and much commercial development happens.

You already use a GitHub repo to sync your learning workspace across devices, which means you've been benefiting from Git without necessarily understanding it. This chapter builds the mental model from scratch and gets your Windows machine configured the way professionals configure theirs. By the end of the track, "sync my workspace" will be just one small trick in a much larger toolkit: safe experimentation, time travel through history, parallel lines of development, and collaboration workflows used by real teams.

Why it matters professionally: interviews assume Git fluency, onboarding at any job starts with cloning a repo, and the difference between a junior who panics at a merge conflict and one who calmly resolves it is mostly just familiarity.

## Definitions & Explanations

**Version control system (VCS)** — software that tracks the history of a set of files. You tell it "record the current state" (a *commit*), and it stores a snapshot you can return to, compare against, or branch away from.

**Centralized vs. distributed** — older systems (SVN, CVS) kept history on one central server; you needed a network connection for most operations. Git is *distributed*: every copy of a repository contains the **entire history**. Your laptop has the full history. GitHub has the full history. Your other device has the full history. This is why Git works offline and why "losing the server" doesn't lose the project.

```
Centralized (SVN):                Distributed (Git):

      [Server: full history]        [GitHub: full history]
        /        \                      /         \
 [You: files   [Teammate:        [You: full     [Teammate: full
  only]         files only]       history]       history]
```

**Repository (repo)** — a directory whose history Git is tracking. The history itself lives in a hidden `.git` subfolder. Delete `.git` and you have an ordinary folder again (and no history — don't do this casually).

**Commit** — a recorded snapshot of your project at a moment in time, plus metadata: author, date, and a message explaining the change. Commits chain together to form history.

**GitHub** — a website that hosts Git repositories and adds collaboration features on top: pull requests, issues, code review, automation. Git works fine without GitHub; GitHub is where teams meet.

**Git Bash vs. PowerShell vs. VS Code terminal** — on Windows, Git ships with "Git Bash," a Unix-style shell. Git commands work identically in PowerShell, cmd, and Git Bash. This track's commands work in any of them; when a command differs by shell, it will be noted. VS Code's integrated terminal can run whichever shell you prefer.

**Your identity** — every commit records an author name and email. Git requires you to configure these once. Use the same email as your GitHub account so GitHub links commits to your profile.

## Command Examples

### Installing Git on Windows

Download from https://git-scm.com/download/win and run the installer, or use winget:

```powershell
# In PowerShell (run as your normal user)
winget install --id Git.Git -e --source winget

# After install, open a NEW terminal window and verify:
git --version
# Expected output (version number will vary):
# git version 2.45.1.windows.1
```

Installer choices that matter (defaults are fine for the rest):
- **Default editor**: choose "Use Visual Studio Code as Git's default editor" if offered.
- **Adjusting PATH**: choose "Git from the command line and also from 3rd-party software" (the recommended middle option).
- **Line endings**: choose "Checkout Windows-style, commit Unix-style" (`core.autocrlf=true`) — the standard for Windows developers.

### First-time configuration

```bash
# Set your identity (used in every commit you make)
git config --global user.name "Your Name"
git config --global user.email "admin@tnt-tutoring.com"   # match your GitHub email

# Make new repos start on a branch named 'main' (matches GitHub's default)
git config --global init.defaultBranch main

# Use VS Code for commit messages and other editing tasks
git config --global core.editor "code --wait"

# Windows line-ending handling (if you didn't set it in the installer)
git config --global core.autocrlf true

# Verify everything
git config --global --list
# Expected output:
# user.name=Your Name
# user.email=admin@tnt-tutoring.com
# init.defaultBranch=main
# core.editor=code --wait
# core.autocrlf=true
```

`--global` writes to `C:\Users\<you>\.gitconfig` and applies to all repos. Omitting `--global` inside a repo writes settings for that repo only.

### Your first repository

```bash
# Create a practice folder and turn it into a repo
mkdir git-practice
cd git-practice
git init
# Expected output:
# Initialized empty Git repository in D:/atoop/git-practice/.git/

# Ask Git what it sees
git status
# Expected output:
# On branch main
# No commits yet
# nothing to commit (create/copy files and use "git add" to track)
```

`git status` is the command you will run more than any other. It never changes anything — it only reports. When in doubt, run `git status`.

### Git in VS Code

Open the folder in VS Code (`code .`). Notice:
- The **Source Control** icon in the left sidebar (branching diagram icon) shows changed files.
- The bottom-left status bar shows the current branch name.
- Creating a file makes it appear under "Changes" with a `U` (untracked) badge.

VS Code's Git UI is a front end for the same commands you'll learn. This track teaches the commands first, because the commands are what every GUI is built on, what documentation and Stack Overflow answers use, and what you'll need when a GUI misbehaves. Use the VS Code UI freely once you know what it's doing underneath.

### Getting help

```bash
git help commit        # opens the full manual page for 'commit' in a browser
git commit -h          # short usage summary in the terminal
```

## Common Pitfalls

**"git is not recognized as an internal or external command"** — you opened the terminal before installing Git, or PATH wasn't updated. Recovery: close *all* terminal windows (and VS Code) and reopen. If it persists, re-run the installer and pick the recommended PATH option.

**Wrong name/email on commits** — commits show "PC User" or an old email. Recovery: fix the config with the commands above. Old commits keep the old identity (rewriting them is possible but rarely worth it in practice repos); all *future* commits use the new one.

**Running `git init` in the wrong folder (like your home directory or `C:\`)** — suddenly Git thinks thousands of files are untracked, and VS Code shows "10k+ pending changes." This scares people badly, but nothing is damaged. Recovery: delete the `.git` folder that was created in the wrong place:

```powershell
# From the folder where you accidentally ran git init:
Remove-Item -Recurse -Force .git
```

**Nested repos** — running `git init` inside a folder that's already inside another repo causes confusing behavior. Check first: `git status` says "not a git repository" when you're safe to init.

**Line-ending warnings** — messages like `warning: LF will be replaced by CRLF` are normal on Windows with `autocrlf=true`. They are informational, not errors. Nothing is wrong.

**Fear of the terminal** — you cannot break your computer with the commands in this track. The genuinely destructive Git commands are few, clearly flagged in Chapter 4 and 9, and almost always recoverable via techniques in Chapter 10.

## Practice Exercises

1. **Install and verify.** Install Git (or verify your existing install), then run `git --version` and `git config --global --list`. Confirm your name, email, `init.defaultBranch=main`, and editor are all set correctly.

2. **Inspect the machinery.** Create a new folder, run `git init`, then explore the `.git` directory (enable "hidden items" in File Explorer, or run `ls -Force` in PowerShell). Find the `HEAD` file and open it in a text editor. What does it contain, and what do you think it means? Write your guess down — you'll confirm it in Chapter 5.

3. **Status detective.** In your new repo, run `git status`. Create a text file, run `git status` again. Rename the file, run it again. Delete it, run it again. For each step, note exactly what changed in the output before moving on.

4. **VS Code cross-check.** Open the same repo in VS Code. Create two files. Compare what the Source Control panel shows against what `git status` prints in the terminal. Identify which UI element corresponds to which line of terminal output.

5. **Config scavenger hunt.** Open `C:\Users\<you>\.gitconfig` in a text editor and read it. Then run `git config --global alias.st status`, look at the file again, and try `git st` in a repo. You've created your first alias — decide whether you want to keep it or remove it (find the command to unset it using `git help config`).
