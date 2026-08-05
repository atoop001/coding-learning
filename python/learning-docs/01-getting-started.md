# Chapter 1: Getting Started with Python

## Overview

Python is one of the most widely used programming languages in the world, and it is the language behind a huge share of back-end web development, automation, data work, and scripting. It reads almost like English, which makes it an excellent first "serious" language — but it is not a toy. Companies like Instagram, Spotify, and Reddit run large parts of their back-ends on Python; Netflix leans on it heavily too, though more for internal tooling, data pipelines, and machine learning than for the consumer-facing back-end itself.

This chapter gets your machine ready: installing Python on Windows, running code three different ways (the REPL, script files, and VS Code), and understanding what actually happens when you "run Python." Everything else in this track builds on the setup you do here, so take your time and verify each step works before moving on.

By the end of this chapter you will be able to:

- Install Python on Windows and verify the installation
- Use the interactive REPL to experiment
- Write a `.py` file and run it from the terminal
- Set up VS Code as your Python editor
- Understand the difference between `python`, `py`, and `pip`

## Definitions & Explanations

**Python interpreter** — the program that reads your Python code and executes it, line by line. When people say "install Python," they mean installing the interpreter (plus the standard library and some tools that come with it).

**Source file / script** — a plain text file ending in `.py` that contains Python code. You run it by passing it to the interpreter: `python my_script.py`.

**REPL (Read-Eval-Print Loop)** — an interactive prompt where you type one line of Python at a time and see the result immediately. You start it by typing `python` in a terminal with no filename. It **R**eads what you type, **E**valuates it, **P**rints the result, and **L**oops back for more. The REPL is your scratchpad — use it constantly to test small ideas.

**Terminal / shell** — the text-based interface where you type commands. On Windows you have several options: **PowerShell** (recommended), **Command Prompt** (cmd), or the terminal built into VS Code. Commands in this track are written for PowerShell unless noted.

**The `py` launcher** — Windows installs a small helper called `py` alongside Python. Typing `py` runs the newest Python you have installed; `py -3.12` runs a specific version. If `python` ever behaves strangely on Windows (it can be shadowed by a Microsoft Store stub), `py` is the reliable fallback.

**pip** — Python's package installer. It downloads and installs third-party libraries (like Flask or requests) from the Python Package Index (PyPI). It comes bundled with Python. We'll use it seriously in Chapter 10.

**PATH** — an environment variable that tells Windows which folders to search when you type a command. During installation you'll check a box that adds Python to PATH, which is what makes `python` work in any terminal.

### Installing Python on Windows

1. Go to <https://www.python.org/downloads/> and download the latest stable installer (3.12 or newer).
2. Run the installer. **Check the box that says "Add python.exe to PATH"** at the bottom of the first screen. This is the single most commonly missed step.
3. Click "Install Now."
4. Open a **new** PowerShell window (existing windows won't see the updated PATH) and verify:

```powershell
python --version
# Python 3.12.x

pip --version
# pip 24.x from ...
```

If `python` opens the Microsoft Store instead of printing a version, either use `py --version`, or disable the Store alias: Settings → Apps → Advanced app settings → App execution aliases → turn off the `python.exe` and `python3.exe` entries.

### Setting up VS Code

1. Install Visual Studio Code from <https://code.visualstudio.com/>.
2. Open VS Code, go to the Extensions panel (Ctrl+Shift+X), and install the **Python** extension published by Microsoft.
3. Open a folder for your work: File → Open Folder → create something like `D:\atoop\coding-projects\python\practice`.
4. Create a new file `hello.py`, and VS Code will activate Python support: syntax highlighting, error squiggles, autocompletion, and a Run button (the triangle, top right).
5. Open the integrated terminal with `` Ctrl+` `` — this is a normal PowerShell running inside VS Code, and it's where you'll run your scripts.

## Code Examples

### Your first REPL session

Open PowerShell and type `python`. You'll see a prompt like `>>>`. Try these:

```python
>>> 2 + 3
5
>>> "hello" + " " + "world"
'hello world'
>>> print("Python is running!")
Python is running!
>>> 10 / 4
2.5
>>> # Lines starting with # are comments — Python ignores them
>>> exit()   # leaves the REPL (Ctrl+Z then Enter also works on Windows)
```

Notice: in the REPL, the value of an expression is printed automatically. In a script file, nothing is shown unless you explicitly `print()` it. This difference trips up almost every beginner at least once.

### Your first script

Create a file called `hello.py` with this content:

```python
# hello.py — my first Python script

print("Hello, world!")          # print() writes text to the terminal
print("2 + 3 =", 2 + 3)         # print can take several values, separated by commas

name = "Alex"                    # a variable (much more on these in Chapter 2)
print("Welcome,", name)
```

Run it from the terminal (make sure you're in the folder containing the file):

```powershell
cd D:\atoop\coding-projects\python\practice
python hello.py
```

Expected output:

```
Hello, world!
2 + 3 = 5
Welcome, Alex
```

### A slightly more realistic script

```python
# greeting.py — asks a question and responds

# input() pauses the program, shows the prompt text,
# and returns whatever the user types (always as text).
name = input("What is your name? ")

print(f"Nice to meet you, {name}!")   # an f-string — covered fully in Chapter 3

# input() ALWAYS returns a string, so numbers must be converted:
birth_year = input("What year were you born? ")
age = 2026 - int(birth_year)          # int(...) converts text to a whole number
print(f"You are about {age} years old.")
```

Run it with `python greeting.py` and answer the prompts.

### Running code three ways — a summary

```powershell
# 1. The REPL: interactive, for experiments
python

# 2. A script file: how real programs run
python my_script.py

# 3. A one-liner, occasionally handy for quick checks
python -c "print(7 * 6)"
```

## Common Pitfalls

**1. `python` is "not recognized"**

```
python : The term 'python' is not recognized...
```

You didn't check "Add to PATH" during install, or you're using a terminal opened before installing. Fix: open a new terminal; if that fails, re-run the installer, choose "Modify," and enable "Add Python to environment variables." Or just use `py` instead.

**2. Running the file from the wrong folder**

```powershell
PS C:\Users\you> python hello.py
python: can't open file 'hello.py': [Errno 2] No such file or directory
```

The terminal looks for `hello.py` in the *current* folder. Use `cd` to navigate to where the file lives first, or pass the full path: `python D:\atoop\coding-projects\python\practice\hello.py`.

**3. Typing REPL prompts into a script file**

```python
# WRONG — this is REPL output pasted into a file, it won't run
>>> print("hi")
hi
```

```python
# RIGHT — a script contains only the code itself
print("hi")
```

**4. Expecting scripts to auto-print values like the REPL does**

```python
# In a file, this line computes 5 and silently throws it away:
2 + 3

# To see it, you must print:
print(2 + 3)
```

**5. Naming your file after a module you use**

If you name a file `random.py` and later try `import random`, Python imports *your* file instead of the standard library, causing baffling errors. Avoid filenames like `random.py`, `string.py`, `test.py`, `json.py`, `flask.py`.

## Practice Exercises

1. **Verify everything.** In a fresh terminal, run `python --version`, `pip --version`, and `py --version`. Then start the REPL, compute `365 * 24`, and exit cleanly.
2. **About-me script.** Write `about_me.py` that prints your name, your favorite food, and one goal for learning Python — three separate `print()` calls. Run it from the terminal.
3. **Interactive greeter.** Write `greeter.py` that asks for the user's name and their favorite hobby with `input()`, then prints one sentence combining both answers.
4. **REPL explorer.** In the REPL, try `7 / 2`, `7 // 2`, `7 % 2`, and `7 ** 2`. Write down (in a comment in a file, or on paper) what you think each operator does. You'll confirm your guesses in Chapter 4.
5. **Break it on purpose.** In a script, write `print("unclosed` (missing the closing quote) and run it. Read the error message carefully — note the filename, line number, and error type. Getting comfortable *reading* errors is a skill you'll use every single day.
