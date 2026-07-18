# Chapter 7: Package Managers — Installing Tools from the CLI

## Overview

Developers install software constantly: languages, CLIs, libraries, utilities. Doing this by browsing websites and clicking through installers is slow, unrepeatable, and error-prone. **Package managers** fix that: one command searches a curated catalog, downloads the right version, installs it, and can later upgrade or remove it cleanly. This chapter covers **winget** (Windows apps), **npm** (Node/JavaScript packages), and **pip** (Python packages), plus the Linux counterpart **apt** you'll meet on servers — and crucially, how installation interacts with PATH from Chapter 6, because "installed but not found" is the #1 post-install complaint.

## Definitions & Explanations

**Package manager** — A tool that installs, upgrades, and removes software from a central catalog ("repository"), handling downloads, dependencies, and (usually) PATH wiring for you. Reproducible: `winget install Git.Git` is a one-line, shareable instruction; "download the installer and click next 6 times" is not.

**Two distinct species — don't confuse them:**
1. *System/app package managers* install **programs onto your machine**: winget (Windows), apt/dnf (Linux), Homebrew (macOS), Chocolatey/Scoop (Windows alternatives).
2. *Language package managers* install **libraries and tools for a language ecosystem**: npm (JavaScript), pip (Python), cargo (Rust), NuGet (.NET). These often install *into a project* rather than globally.

**winget** — Microsoft's official Windows package manager, preinstalled on Windows 11. Catalog IDs look like `Publisher.Product` (`Git.Git`, `Microsoft.PowerShell`, `Python.Python.3.12`).

**npm** — Installs JavaScript packages. Two modes: *local* (into the current project's `node_modules/`, recorded in `package.json` — the default and usually correct) and *global* (`-g`, for CLI tools you want available everywhere, like `typescript` or `vercel`). `npx <tool>` runs a package's CLI without permanently installing it.

**pip** — Installs Python packages. The safest invocation on Windows is `py -m pip install ...` (or `python -m pip`), which guarantees pip matches the Python you think you're using. **Virtual environments** (`venv`) give each project its own isolated package set — the Python equivalent of npm's per-project `node_modules`, and the professional default.

**apt** — Debian/Ubuntu's package manager, which you'll use inside WSL and on most servers: `sudo apt update` refreshes the catalog, `sudo apt install <pkg>` installs. `sudo` = run as administrator (Chapter 10).

**Dependency** — A package your package needs. Managers resolve chains of these automatically — the core value proposition.

**PATH implications** — Installing a CLI tool only helps if its executable's folder is on PATH. Good installers add it (to the *User* or *Machine* scope) — but running terminals won't see the change until restarted (Chapter 6). Language managers add their own bin folders (npm's global prefix, Python's `Scripts\` folder); when those aren't on PATH, tools install "successfully" yet can't be run.

## Command Examples

winget — managing Windows applications:

```powershell
winget search "powershell"          # find packages
# Name              Id                    Version
# ----              --                    -------
# PowerShell        Microsoft.PowerShell  7.4.1.0
# ...

winget install Microsoft.PowerShell     # install by exact Id (preferred over name)
# Found PowerShell [Microsoft.PowerShell] Version 7.4.1.0
# Downloading ...
# Successfully installed

winget list                          # everything installed (winget-aware)
winget list --upgrade-available      # what has updates?
winget upgrade Microsoft.PowerShell  # upgrade one
winget upgrade --all                 # upgrade everything
winget uninstall Microsoft.PowerShell
winget show Git.Git                  # details before installing

# A realistic developer setup, one line each:
winget install Git.Git
winget install Microsoft.VisualStudioCode
winget install Python.Python.3.12
winget install OpenJS.NodeJS.LTS
# After installing: open a NEW terminal before expecting git/node/python to resolve.
```

npm — JavaScript packages:

```powershell
node --version && npm --version     # confirm Node is installed and found
# v20.11.0
# 10.2.4

# LOCAL install (inside a project folder) — the default you usually want:
mkdir demo-app; cd demo-app
npm init -y                          # create package.json
npm install express                  # into ./node_modules, saved to package.json
# added 64 packages in 2s

npm install --save-dev prettier      # dev-only dependency
npm list --depth=0                   # what's installed here?
# demo-app@1.0.0
# +-- express@4.19.2
# +-- prettier@3.2.5

# GLOBAL install — for standalone CLI tools only:
npm install -g typescript
tsc --version                        # now available everywhere
# Version 5.4.5

npm root -g                          # where do global packages live?
npx cowsay "no install needed"       # run a tool without installing it
npm uninstall express                # remove local package
npm outdated                         # local packages with newer versions
```

pip — Python packages (with the venv habit):

```powershell
py --version                         # Windows Python launcher
# Python 3.12.2

# The professional pattern: one virtual environment per project
py -m venv .venv                     # create env in .venv folder
.\.venv\Scripts\Activate.ps1         # activate it — prompt gains (.venv) prefix
# (.venv) PS D:\...\demo>

py -m pip install requests           # installs INTO the venv only
# Successfully installed requests-2.32.3 ...
py -m pip list                       # what's in this env?
py -m pip freeze > requirements.txt  # snapshot exact versions for teammates
py -m pip install -r requirements.txt  # ...and restore from a snapshot
deactivate                           # leave the venv
```

```bash
# Bash/Linux equivalents (WSL or server)
python3 -m venv .venv
source .venv/bin/activate            # note: source, and bin/ not Scripts\
pip install requests
pip freeze > requirements.txt
deactivate
```

apt — a preview of server life (run inside WSL, Chapter 10):

```bash
sudo apt update                      # refresh package catalog (do this first)
sudo apt install htop                # install
htop                                 # run it (q to quit)
sudo apt upgrade                     # upgrade everything
apt list --installed | head          # what's installed
sudo apt remove htop                 # uninstall
```

## Common Pitfalls

**Pitfall: tool installed, terminal says "not recognized."**
The install succeeded; your already-open terminal has a stale PATH.
*Correction*: open a **new terminal window** first — this fixes it 90% of the time. If not: find where the tool installed, check whether that folder is in `$env:PATH -split ';'`, and add it (User scope) if missing. This is Chapter 6 knowledge paying rent.

**Pitfall: `npm install -g` used for everything.**
Global installs of project libraries cause version conflicts between projects and "works on my machine" bugs.
*Correction*: libraries are local (plain `npm install` inside the project); only standalone CLI tools earn `-g`. If a project README says `npm install`, it means local.

**Pitfall: pip installing into the wrong Python.**
Multiple Pythons (Microsoft Store, python.org, WSL) each have their own pip. `pip install x` targets whichever pip PATH finds — then `python` runs a *different* interpreter that lacks the package: `ModuleNotFoundError`.
*Correction*: always `py -m pip install ...` (Windows) or `python3 -m pip ...` (Linux) so pip and interpreter are guaranteed to match. Better: use a venv per project and the problem largely disappears.

**Pitfall: skipping venvs and polluting global Python.**
Every project's dependencies pile into one shared space until versions conflict and nothing is reproducible.
*Correction*: make `py -m venv .venv` + activate the reflexive first step of any Python project. If your prompt doesn't show `(.venv)`, you're not in one.

**Pitfall: `sudo apt install` on a fresh box fails with "Unable to locate package."**
The local catalog is empty or stale.
*Correction*: `sudo apt update` first, then install. On any new WSL distro or server, `sudo apt update` is always step one.

**Pitfall: execution policy blocks venv activation.**
`.\.venv\Scripts\Activate.ps1 : ...running scripts is disabled on this system.`
*Correction*: this is PowerShell's script execution policy, covered fully in Chapter 8. The standard fix: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`, then retry.

**Pitfall: installing from random scripts piped into a shell.**
`curl something | bash` or `irm something | iex` executes arbitrary internet code with your permissions.
*Correction*: prefer the package manager's catalog. When a vendor only offers the pipe-to-shell method, read the script first (download it, inspect, then run) and make sure the source is the official domain.

## Practice Exercises

1. Use `winget search` to find three tools: a terminal-based text editor, a JSON processor called `jq`, and anything else that intrigues you from `winget search cli`. For each, run `winget show` and note the publisher, version, and install location hints. Install `jq` and prove it works with `'"hello"' | jq .` (new terminal if needed!).
2. Verify (or set up) Node and Python via winget. Then demonstrate the PATH lifecycle: before opening a new terminal, show the command isn't found; after opening a new one, show it is. Write down which PATH entry each tool added (`$env:PATH -split ';'` diff, or `Get-Command`).
3. Create a scratch folder, make it an npm project, install any two packages locally and one CLI tool globally. Show: the local packages listed at depth 0, the global tool running from a *different* directory, and where the global bin folder is (`npm root -g`, then find the adjacent bin location on PATH).
4. Create a Python venv in a scratch project, activate it, install `requests` and `rich`, export `requirements.txt`, deactivate, delete the venv folder entirely, recreate it, and restore both packages from the requirements file. This round-trip is the core professional Python workflow.
5. (Requires WSL or any Linux box — park it until Chapter 10 if needed.) Run the full apt cycle: update, install `tree`, run `tree` on a directory, then remove it. Compare each step to its winget equivalent in a two-column note.
