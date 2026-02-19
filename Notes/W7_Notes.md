# Comprehensive Bash Scripting Study Guide

> **Who this guide is for:** Beginners who are learning Linux and Bash scripting. No prior Linux experience is assumed. Every concept is explained from the ground up — not just *what* a command does, but *why* it exists and *when* you'd actually use it.

---

## Table of Contents

1. [Bash Expansions](#1-bash-expansions)
2. [Debugging Bash Scripts](#2-debugging-bash-scripts)
3. [Parsing Options with `getopts`](#3-parsing-options-with-getopts)
4. [Here Documents (Heredocs)](#4-here-documents-heredocs)
5. [Getting User Input with `read`](#5-getting-user-input-with-read)
6. [Writing Functions in Bash](#6-writing-functions-in-bash)

---

---

# 1. Bash Expansions

## What Is an Expansion?

When you type a command in the terminal and press Enter, Bash does not send your raw text directly to the operating system. First, it **transforms** your input through a series of processes called **expansions**. Each expansion finds certain patterns in your command and replaces them with something else before the command actually runs.

Think of it like a word processor doing Find & Replace on your command before submitting it.

**Why does this matter?** Understanding expansions helps you:

- Understand why a command behaves differently than you expect
- Write more powerful and concise scripts
- Avoid common bugs caused by expansion order or quoting mistakes

## The Seven Expansion Types (in Order)

Bash performs expansions in this exact order. The order matters because one expansion can produce output that feeds into the next.

1. **Brace expansion** — generates strings from patterns like `{1..5}` or `{a,b,c}`
2. **Tilde expansion** — converts `~` into your home directory path
3. **Parameter and variable expansion** — replaces `$variable` with its value
4. **Command substitution** — replaces `$(command)` with the output of that command
5. **Arithmetic expansion** — evaluates `$(( math ))` and replaces it with the result
6. **Word splitting** — splits unquoted results containing spaces into separate words
7. **Filename expansion (globbing)** — expands wildcard patterns like `*.txt` into actual filenames

A Funny way to remember the order:
**The PC Builder**
1. Big (Brace)
2. Tower (Tilde)
3. PCs (Parameter)
4. Cost (Command)
5. A (Arithmetic)
6. Whole (Word splitting)
7. Fortune (Filename)

---

## 1.1 Brace Expansion

### What It Is

Brace expansion generates a list of strings from a compact pattern. The shell expands the braces *before* looking at the filesystem, so it works even if the files or directories don't exist yet. This makes it especially useful for creating, renaming, or referencing multiple files at once.

### Syntax

```bash
# Sequence: {start..end} or {start..end..step}
echo {1..5}        # 1 2 3 4 5
echo {1..10..2}    # 1 3 5 7 9  (every 2nd number)
echo {a..f}        # a b c d e f
echo {a..f..2}     # a c e  (every 2nd letter)

# Comma-separated list
echo {apple,banana,cherry}    # apple banana cherry

# With a preamble (prefix) and/or postscript (suffix)
echo file{1..3}         # file1 file2 file3
echo file{1..3}.txt     # file1.txt file2.txt file3.txt
echo {pre_}file{1..3}   # pre_file1 pre_file2 pre_file3
```

### Real-World Example: Creating a Directory Structure

Without brace expansion, creating a nested directory structure requires multiple commands:

```bash
mkdir project
mkdir project/src
mkdir project/tests
mkdir project/docs
```

With brace expansion, you do it in one command:

```bash
mkdir -p project/{src,tests,docs}
```

The shell expands this to `mkdir -p project/src project/tests project/docs` before running.

You can even nest brace expansions:

```bash
mkdir -p outer{1..3}/inner{a..c}
# Creates: outer1/innera, outer1/innerb, outer1/innerc,
#          outer2/innera, outer2/innerb ... outer3/innerc
```

> **What My Professor Didn't Explain:** The `-p` flag on `mkdir` means "create parent directories as needed." Without it, `mkdir outer1/innera` would fail if `outer1` doesn't exist yet. With `-p`, it creates both `outer1` and `outer1/innera` in one step.

### ⚠️ Common Gotcha: No Spaces Around Commas

```bash
echo {a, b, c}   # ❌ WRONG — spaces break the expansion, you get {a, b, c} literally
echo {a,b,c}     # ✅ CORRECT — a b c
```

---

## 1.2 Tilde Expansion

### What It Is

When Bash sees an **unquoted** `~` at the start of a word, it replaces it with the home directory of the current user. This is called **tilde expansion**.

### Why It Exists

Every user on a Linux system has a home directory — a personal folder where their files, settings, and configurations live. These paths look like `/home/username`. Typing the full path every time is tedious and non-portable (someone else's username is different from yours).

The `~` shortcut solves this.

```bash
echo ~           # /home/yourname
cd ~             # takes you to your home directory
ls ~/Documents   # lists your Documents folder
```

### Why Scripts Use `$HOME` Instead of `~`

You'll notice that Bash scripts almost always use the `$HOME` variable instead of `~`:

```bash
# ❌ Fragile in scripts:
cp config.txt ~/backup/

# ✅ Safer in scripts:
cp config.txt "$HOME/backup/"
```

**The reason:** When `~` is inside double quotes (`"~"`), tilde expansion does **not** happen — it stays as a literal `~` character. Observe:

```bash
echo "~"         # prints: ~   (no expansion — it's just a tilde character)
echo ~           # prints: /home/yourname  (expansion happens)
echo "$HOME"     # prints: /home/yourname  (always works, even inside quotes)
```

Since variables inside scripts are often quoted to prevent word splitting (more on that later), `$HOME` is the safer choice.

---

## 1.3 Parameter and Variable Expansion

### What It Is

**Parameter expansion** is how Bash replaces a variable name with its value. The basic form is `$variable` or `${variable}`. But it goes far beyond simple substitution — you can extract substrings, get the length of a variable, replace patterns, and more.

### Basic Substitution

```bash
first="Nathan"
last="McNinch"

echo $first         # Nathan
echo ${first}       # Nathan (same thing, explicit form)
echo "${first}_${last}"  # Nathan_McNinch
```

> **What My Professor Didn't Explain (the `{}` question):** If you remove the curly braces and write `echo "$first_$last"`, Bash will try to find a variable called `$first_` (everything up to the next `$`). Since `$first_` doesn't exist, it expands to nothing, and you'd get `McNinch` — not what you wanted. The curly braces `{}` tell Bash exactly where the variable name ends.

### Getting the Length of a Variable

Prefix the variable name with `#` inside the braces:

```bash
first="Nathan"
echo "${#first}"    # 6  (the string "Nathan" has 6 characters)

arr=(one two three)
echo "${#arr[@]}"   # 3  (the array has 3 elements)
```

### String Replacement

You can find and replace text within a variable's value without using external tools like `sed`:

```bash
letters="ababc"

echo "${letters/ab/XY}"   # XYabc  — replaces only the FIRST match
echo "${letters//ab/XY}"  # XYXYc  — replaces ALL matches (note the //)
```

The pattern is: `${variable/find/replace}` for first match, `${variable//find/replace}` for all matches.

### Finding Variable Names by Prefix

```bash
prefix_a="one"
prefix_b="two"

echo ${!prefix_*}   # prints: prefix_a prefix_b
```
The Anatomy of `${!prefix_*}`:
- The `!` before the variable name tells Bash to return the *names* of all variables that start with `prefix_`. This is useful when you have a group of related variables and want to loop through them.
- `prefix_*`: This tells Bash to look for any variable name that begins with "prefix_".
- `*` (or `@`): The wildcard that instructs Bash to find all matches.
### String Slicing

Extract a portion of a string using `${variable:start:length}`:

```bash
name="Nathan"
echo "${name:0:2}"  # Na  — start at index 0, take 2 characters
echo "${name:2:2}"  # th  — start at index 2, take 2 characters
echo "${name:4}"    # an  — start at index 4, take everything to the end
```

> Indexes in Bash start at **0**, not 1.

### Extracting File Extensions

```bash
file="report.final.txt"
echo "${file##*.}"   # txt — removes the longest match of "*." from the front
echo "${file#*.}"    # final.txt — removes the shortest match of "*." from the front
```

The `#` removes from the **left** (front), and `%` removes from the **right** (end):

```bash
echo "${file%.*}"    # report.final — removes the shortest match of ".*" from the end
echo "${file%%.*}"   # report — removes the longest match of ".*" from the end
```

This is how you cleanly extract or strip file extensions without using `cut` or `sed`.

### Default Values

```bash
# Use a default if the variable is unset or empty
echo "${username:-"guest"}"   # prints "guest" if $username is empty/unset
```

---

## 1.4 Command Substitution

### What It Is

**Command substitution** lets you run a command and use its output as a value — like assigning the result of a command to a variable or embedding it inside another string.

```bash
# Store the current date in a variable
now=$(date)
echo "The time is: $now"

# Use a command's output inside another command
dir=$(realpath "$1")   # converts a relative path to an absolute path
echo "Working in: $dir"
```

The `$(...)` syntax is the modern form. You may also see the older backtick form `` `command` ``, but `$(...)` is preferred because it's easier to read and can be nested.

### Nesting Command Substitutions

```bash
# Get the size of the largest file in the current directory
largest=$(du -sh $(ls -t | head -1))
```

### The New Inline Command Substitution Form

Bash recently introduced a newer form:

```bash
${| command; }
```

This executes the command and captures its output via the `$REPLY` variable. It's useful for inline conditional logic:

```bash
filename="report.txt"

ext=${|
  [[ $filename == *.* ]] && REPLY=${filename##*.} || REPLY=""
}

echo "Extension: $ext"   # Extension: txt
```

Think of this as a mini anonymous function — like a lambda. The Python equivalent would be:

```python
ext = (lambda f: f.rsplit(".", 1)[1] if "." in f else "")(filename)
```

> **Note:** This newer `${| }` syntax requires a recent version of Bash (5.3+). If you're on an older system, it may not be available.

---

## 1.5 Arithmetic Expansion

### What It Is

Arithmetic expansion lets you do integer math inside a script. The syntax is `$(( expression ))`.

```bash
echo "$(( 3 + 4 ))"        # 7
echo "$(( (3 + 4) * 5 ))"  # 35
echo "$(( 10 / 3 ))"       # 3  (integer division — no decimals!)
echo "$(( 10 % 3 ))"       # 1  (modulo — remainder after division)
```

You can also use variables:

```bash
x=10
y=3
echo "$(( x * y ))"   # 30
echo "$(( x ** y ))"  # 1000  (exponentiation)
```

### ⚠️ Warning: Bash Is Bad at Math

Bash arithmetic only handles **integers** (whole numbers). There are no decimals:

```bash
echo "$(( 7 / 2 ))"   # 3  (not 3.5!)
```

For anything requiring decimals, percentages, or complex math, use `bc` (a calculator tool) or `awk`:

```bash
echo "scale=2; 7 / 2" | bc    # 3.50
awk 'BEGIN { printf "%.2f\n", 7/2 }'   # 3.50
```

The good news: you can call `bc` or `awk` from inside a Bash script with command substitution and continue using the result in your script.

---

## 1.6 Word Splitting

### What It Is

After parameter expansion, command substitution, and arithmetic expansion, Bash scans the results for **whitespace characters** (spaces, tabs, newlines). If the result was **not** inside double quotes, Bash splits it into separate words at each whitespace boundary.

### Why This Matters

```bash
# Without quotes — word splitting occurs:
nums1=( $(echo "one two three") )
echo "${#nums1[@]}"   # 3 — three separate array elements

# With quotes — no word splitting:
nums2=( "$(echo "one two three")" )
echo "${#nums2[@]}"   # 1 — one element (the whole string)
```

This is why you'll see `"$variable"` with quotes everywhere in professional scripts — to prevent word splitting from splitting your data into unexpected pieces.

### The IFS Variable

Word splitting is controlled by the **Internal Field Separator** variable, `IFS`. Its default value is whitespace (space, tab, newline).

You can change it temporarily to split on different characters:

```bash
data="alice:bob:carol"
IFS=":"
read -r a b c <<< "$data"
echo "$a $b $c"   # alice bob carol
```

### ⚠️ The Most Common Beginner Bug

```bash
filename="my file.txt"   # filename with a space in it

# This breaks because word splitting turns it into two arguments: "my" and "file.txt"
cp $filename /tmp/

# This is correct — quotes preserve the space
cp "$filename" /tmp/
```

**Golden rule: always quote your variables unless you specifically want word splitting.**

---

## 1.7 Filename Expansion (Globbing)

### What It Is

**Filename expansion** (also called **globbing**) lets you refer to multiple files using wildcard patterns. When Bash sees certain special characters in an unquoted word, it replaces that word with a sorted list of all matching filenames.

### The Three Wildcard Characters

| Character | Meaning | Example |
|-----------|---------|---------|
| `*` | Matches any sequence of characters (including none) | `*.txt` matches all `.txt` files |
| `?` | Matches exactly one character | `file?.txt` matches `file1.txt` but not `file10.txt` |
| `[...]` | Matches any one character listed inside the brackets | `file[123].txt` matches `file1.txt`, `file2.txt`, `file3.txt` |

### Examples

```bash
ls *.txt          # list all .txt files in current directory
ls file?.log      # list file1.log, file2.log, filea.log, etc.
ls file[1-3].txt  # list file1.txt, file2.txt, file3.txt
ls d*             # list everything starting with 'd'
ls *[1-2]         # list everything ending with 1 or 2
```

### Expansion Order in Action

Remember: brace expansion happens *before* filename expansion. So this command:

```bash
echo file{1..3}*.txt
```

First expands to: `echo file1*.txt file2*.txt file3*.txt`

Then each of those patterns undergoes filename expansion, matching actual files on disk.

> **What My Professor Didn't Explain:** If a glob pattern matches *no* files, by default Bash leaves the pattern as a literal string (e.g., `*.xyz` stays as `*.xyz`). This can cause bugs — a command might receive the literal string `*.xyz` instead of failing cleanly. You can change this behavior with `shopt -s nullglob` (which causes unmatched globs to expand to nothing).

---

---

# 2. Debugging Bash Scripts

## Why Bash Debugging Is Different

Most programming languages (Python, Java, etc.) come with interactive debuggers — tools that let you pause a program mid-execution, inspect variable values, and step through code line by line.

**Bash has no built-in interactive debugger.** Instead, Bash provides a different set of debugging tools:

- **Runtime debugging** using the `set` builtin (to change how Bash runs your script)
- **Static analysis** using `shellcheck` (to find bugs without running the script)

Both approaches are genuinely useful and complement each other — use `shellcheck` before running your script to catch obvious problems, and use `set` options while running to trace what's happening live.

---

## 2.1 Debugging with the `set` Builtin

### What Is `set`?

`set` is a Bash builtin command that changes the shell's behavior. When used for debugging, certain `set` options make Bash print extra information or exit early when something goes wrong.

You can place `set` options:
- **At the top of your script** to apply them for the whole script
- **Around a specific section** to debug just that part

---

### `set -x` — Trace Every Command Before It Runs

**Purpose:** `set -x` tells Bash to print each command to the terminal *after expansions* but *before execution*. Each traced line is prefixed with `+`.

```bash
#!/usr/bin/env bash

set -x

name="Yan"
echo "Hello, $name"
```

**Output:**

```
+ name=Yan
+ echo 'Hello, Yan'
Hello, Yan
```

Notice how `echo "Hello, $name"` shows up as `echo 'Hello, Yan'` in the trace — you can see the result after variable expansion. This is incredibly useful for verifying that your variables contain what you think they contain.

**Turning tracing off for just a section:**

```bash
#!/usr/bin/env bash

declare -A settings=(
    [host]="localhost"
    [port]="8080"
    [env]="dev"
)

# Only trace this loop:
set -x
for key in "${!settings[@]}"; do
    echo "Key: $key = ${settings[$key]}"
done
set +x   # Turn tracing OFF after the loop

echo "Script continues silently..."
```

> **What `+x` means:** In `set`, a **minus sign (`-`) turns an option ON**, and a **plus sign (`+`) turns it OFF**. This is counterintuitive, but that's how it works.

---

### `set -u` — Treat Unset Variables as Errors

**Purpose:** By default, Bash silently treats an unset variable as an empty string. This hides bugs — especially typos in variable names.

```bash
# Without set -u:
echo "$usernmae"   # typo! But Bash just prints an empty line — no error
```

With `set -u`, Bash exits immediately with an error message:

```bash
#!/usr/bin/env bash

set -u

echo "$username"   # Error: username: unbound variable
```

**Output:**

```
script.sh: line 5: username: unbound variable
```

**Use this to:**
- Catch typos in variable names early
- Detect missing environment variables before they cause silent failures

> **Gotcha:** If you use `set -u` and need to test whether a variable *is* set, use `${variable:-default}` or check with `[[ -v variable ]]` before accessing it. Accessing an unset variable directly will cause your script to exit.

---

### `set -e` — Exit Immediately on Any Error

**Purpose:** Normally, Bash keeps running even if a command fails. `set -e` changes this — the script exits immediately if any command returns a non-zero exit status (which in Linux means "something went wrong").

```bash
#!/usr/bin/env bash

set -e

cp missing_file.txt /tmp/   # This will fail
echo "This line will NEVER run"
```

**Output:**

```
cp: cannot stat 'missing_file.txt': No such file or directory
```

The script exits at `cp` and never reaches the `echo`.

**When to use it:** Use `set -e` in automation scripts or deployment scripts where continuing after a failure would make things worse (e.g., deploying a broken version of code).

**⚠️ Important Exceptions to `set -e`:**

`set -e` does **not** exit if a failing command is:
- Part of an `if` condition: `if cp file /tmp/; then ...`
- Part of a `while` or `until` condition
- Followed by `||` or `&&`
- A pipeline (unless combined with `pipefail`)

These exceptions exist because checking exit codes in conditions is a normal and intentional pattern.

---

### Combining Options: "Safe Mode"

You'll frequently see this at the top of professional Bash scripts:

```bash
#!/usr/bin/env bash

set -euo pipefail
```

This combines three options:
- `-e` — exit on error
- `-u` — treat unset variables as errors
- `-o pipefail` — if any command in a **pipeline** (`cmd1 | cmd2 | cmd3`) fails, treat the whole pipeline as failed

**Why `pipefail` matters:**

```bash
# Without pipefail, this passes silently even though "grep" found nothing:
cat file.txt | grep "keyword" | sort

# With pipefail, if grep returns exit code 1 (no match), the pipeline fails
```

Using `set -euo pipefail` is a common best practice for writing robust scripts.

---

## 2.2 Static Analysis with `shellcheck`

### What Is `shellcheck`?

`shellcheck` is an external tool — not part of Bash itself — that reads your script and reports potential bugs **without running it**. Think of it as a spell-checker, but for Bash scripts. It catches issues like:

- Unquoted variables that could cause word splitting bugs
- Incorrect use of `$@` vs `$*`
- Useless `cat` usage (piping `cat` into a command when the command can read files directly)
- Syntax errors
- Many subtle Bash pitfalls that are easy to miss

### Installing `shellcheck`

```bash
# Search for it in Debian's package repositories
apt search shellcheck

# Get information about the package
apt info shellcheck

# Install it (requires sudo because it modifies the system)
sudo apt install shellcheck
```

> **What My Professor Didn't Explain:** You need `sudo` (superuser privileges) for `apt install` because installing software modifies system directories that regular users can't write to. You don't need `sudo` for `apt search` or `apt info` because those commands only *read* information.

### Using `shellcheck`

```bash
shellcheck my_script.sh
```

**Example — a buggy script:**

```bash
#!/usr/bin/env bash
filename="my file.txt"
cp $filename /tmp/   # unquoted variable!
```

**shellcheck output:**

```
In my_script.sh line 3:
cp $filename /tmp/
   ^--------^ SC2086: Double quote to prevent globbing and word splitting.
```

shellcheck doesn't just tell you *what's* wrong — it tells you *why* it's wrong and usually provides a link to a longer explanation.

### Why `shellcheck` Is Important

- It catches bugs **before** you run your script (no surprises in production)
- It teaches you Bash best practices through its explanations
- It's used widely in professional CI/CD pipelines — many companies require scripts to pass `shellcheck` before they can be deployed

> **Best Practice:** Make running `shellcheck` a habit before every script you write, especially when you're still learning.

---

---

# 3. Parsing Options with `getopts`

## What Are Command-Line Options (Flags)?

When you run commands in Linux, you often pass **options** (also called **flags**) to change the command's behavior:

```bash
ls -l -a        # -l for long format, -a for all files (including hidden)
cp -r dir /tmp  # -r for recursive copy
grep -i "word" file.txt  # -i for case-insensitive search
```

When you write your own Bash scripts, you can add this same functionality using the `getopts` builtin. This lets your script accept and process flags like a real Linux command.

---

## What Is `getopts`?

`getopts` is a Bash builtin that parses the positional parameters passed to your script (`$@`) and processes them one option at a time. It handles **short options** — single-letter flags prefixed with a dash, like `-a`, `-b`, `-v`.

> **Note:** `getopts` only handles short options. For long options like `--verbose` or `--output`, you need a different tool (`getopt`, without the 's') or manual parsing.

---

## 3.1 Basic `getopts` Usage

### Syntax

```bash
while getopts "option_string" variable_name; do
  case "$variable_name" in
    option1) ... ;;
    option2) ... ;;
    *) exit 1 ;;
  esac
done
```

The **option string** lists all the flags your script accepts. For example, `"ab"` means your script accepts `-a` and `-b`.

### Example: Simple Flags

```bash
#!/usr/bin/env bash

while getopts "ab" opt; do
  case "${opt}" in
    a)
      echo "You supplied option -a"
      ;;
    b)
      echo "You supplied option -b"
      ;;
    *)
      echo "Unknown option"
      exit 1
      ;;
  esac
done

exit 0
```

Running this script:

```bash
./script.sh -a        # You supplied option -a
./script.sh -b        # You supplied option -b
./script.sh -a -b     # You supplied option -a
                      # You supplied option -b
./script.sh -c        # Unknown option (then exits)
```

**How `getopts` works with `while`:** `getopts` processes one option per call and returns `0` (success) if it found a valid option, or `1` (failure) when there are no more options to process. The `while` loop keeps calling it until there are no more options. Each time, the current option letter is stored in `opt`.

---

## 3.2 Options That Take Arguments (OPTARG)

Some flags take a value. For example, `grep -n 5 file.txt` — the `-n` flag takes the argument `5`. In `getopts`, you mark these options with a colon (`:`) after the letter in the option string.

When an option takes an argument, that argument is automatically stored in the special variable `OPTARG`.

### Example: Flags with Arguments

```bash
#!/usr/bin/env bash

vara=""
varb=""

while getopts ":a:b:" opt; do
  case "${opt}" in
    a)
      vara="$OPTARG"   # OPTARG holds the value passed after -a
      ;;
    b)
      varb="$OPTARG"   # OPTARG holds the value passed after -b
      ;;
    :)
      # This case runs when a flag is used WITHOUT its required argument
      echo "Error: -${OPTARG} requires an argument"
      exit 1
      ;;
    ?)
      # This case runs for unrecognized flags
      exit 1
      ;;
  esac
done

if [[ -n "$vara" ]]; then
  echo "Option a was: $vara"
fi

if [[ -n "$varb" ]]; then
  echo "Option b was: $varb"
fi
```

Running this script:

```bash
./script.sh -a hello          # Option a was: hello
./script.sh -a hello -b world # Option a was: hello
                              # Option b was: world
./script.sh -a                # Error: -a requires an argument
```

### Decoding the Option String `":a:b:"`

| Character | Meaning |
|-----------|---------|
| Leading `:` | **Silent mode** — prevents getopts from printing its own error messages; lets your script handle errors |
| `a:` | The `-a` flag accepts an argument (the `:` after the letter means "requires argument") |
| `b:` | The `-b` flag also accepts an argument |

---

## 3.3 The `shift` Command and OPTIND

### What Is `shift`?

`shift` moves the positional parameters (`$1`, `$2`, `$3`, ...) to the left by a given number of positions (default: 1). The leftmost parameter is discarded.

```bash
#!/usr/bin/env bash
# Run as: ./script.sh 1 2 3 4

echo "$1 $2 $3"   # 1 2 3

shift             # shift left by 1

echo "$1 $2 $3"   # 2 3 4

shift 2           # shift left by 2

echo "$1 $2 $3"   # 4 (only one value left; $2 and $3 are empty)
```

### What Is OPTIND?

`OPTIND` (Option Index) is a variable automatically managed by `getopts`. It tracks the index of the next positional parameter to be processed. After `getopts` finishes processing all flags, `OPTIND` points to the first non-flag argument (a regular positional argument).

To remove the flags from `$@` after `getopts` is done (so that `$1` now points to the first non-flag argument), use:

```bash
shift $((OPTIND - 1))
```

### Complete Example: Flags + Positional Arguments

This is a real-world pattern — a script that accepts optional flags *and* a required filename:

```bash
#!/usr/bin/env bash

usage() {
  echo "Usage: $0 [-a value] [-b value] filename"
  exit 1
}

vara=""
varb=""

# Parse flags first
while getopts ":a:b:" opt; do
  case "${opt}" in
    a)
      vara="$OPTARG"
      ;;
    b)
      varb="$OPTARG"
      ;;
    :)
      echo "Error: -${OPTARG} requires an argument"
      exit 1
      ;;
    ?)
      usage
      ;;
  esac
done

# Remove the processed flags from $@
# Now $1 is the first non-flag argument (the filename)
shift $((OPTIND - 1))

# Check that at least one positional argument remains
if [[ $# -lt 1 ]]; then
  usage
fi

echo "Filename: $1"

[[ -n "$vara" ]] && echo "Option a: $vara"
[[ -n "$varb" ]] && echo "Option b: $varb"
```

Running this:

```bash
./script.sh -a "hello" -b "world" myfile.txt
# Filename: myfile.txt
# Option a: hello
# Option b: world

./script.sh myfile.txt
# Filename: myfile.txt

./script.sh -a "hello"
# Usage: ./script.sh [-a value] [-b value] filename
```

> **Key Rule:** Flags (`-a`, `-b`) must always come **before** positional arguments when using `getopts`. This is the standard Unix convention. `getopts` stops parsing as soon as it hits a non-flag argument.

---

---

# 4. Here Documents (Heredocs)

## What Is a Heredoc?

A **here document** (heredoc) is a way to provide multi-line text input to a command directly inside a script, without needing a separate file. Instead of reading input from the keyboard or a file, the input is written inline between two delimiter markers.

### The Problem Heredocs Solve

Imagine you want a script to send several commands over SSH to a remote server, or to generate a multi-line configuration file. Without heredocs, you'd have to write awkward chains of `echo` statements:

```bash
echo "line one"
echo "line two"
echo "line three"
```

With a heredoc, you write it naturally:

```bash
cat << EOF
line one
line two
line three
EOF
```

---

## 4.1 Basic Heredoc Syntax

```bash
command << DELIMITER
  line 1
  line 2
  line 3
DELIMITER
```

**Rules:**
- `DELIMITER` can be any word you choose — `EOF` (End Of File) is the convention, but anything works
- The opening `<< DELIMITER` goes on the same line as the command
- The closing `DELIMITER` must be on a line **by itself**, with no leading spaces and no trailing spaces
- Everything between the delimiters is passed as standard input to the command

```bash
cat << EOF
Hello
This is a multi-line
string passed to cat.
EOF
```

**Output:**

```
Hello
This is a multi-line
string passed to cat.
```

> **What My Professor Didn't Explain:** This works because `cat` reads from **standard input** when not given a filename. The `<<` operator redirects input from the heredoc text instead of the keyboard. So `cat << EOF ... EOF` is equivalent to typing that text at a `cat` prompt.

---

## 4.2 Creating a File with a Heredoc

You can redirect a heredoc to a file using `>`:

```bash
cat << EOF > groceries.txt
apples
carrots
bananas
EOF
```

Read this right to left: "take the heredoc input, feed it to `cat`, then redirect `cat`'s output to `groceries.txt`."

After running this, `groceries.txt` contains:

```
apples
carrots
bananas
```

---

## 4.3 Variable Expansion in Heredocs

By default, heredocs expand variables, run command substitutions, and evaluate arithmetic — just like a regular double-quoted string.

```bash
name="Miho"

cat << EOF
Hello, $name
Today is $(date)
2 + 2 = $((2 + 2))
EOF
```

**Output:**

```
Hello, Miho
Today is Fri Feb 13 10:23:45 UTC 2026
2 + 2 = 4
```

### Disabling Expansion with Quoted Delimiters

If you want the heredoc content treated as **literal text** (no variable expansion), put the delimiter in single quotes:

```bash
cat << 'EOF'
Hello, $name
Today is $(date)
2 + 2 = $((2 + 2))
EOF
```

**Output:**

```
Hello, $name
Today is $(date)
2 + 2 = $((2 + 2))
```

Everything is printed literally. This is useful when generating scripts, configuration files, or templates that contain `$`, backticks, or `$(...)` that should not be interpreted by the current shell.

---

## 4.4 Indenting Heredocs with `<<-`

Heredocs normally require the delimiter to start at the very beginning of the line, which can make scripts look messy when they're inside functions or if-blocks. The `<<-` operator lets you indent the body using **tabs**:

```bash
if true; then
    cat <<- EOF
	Indented line one
	Indented line two
	EOF
fi
```

**Important:** `<<-` strips **tabs** from the beginning of each line, but **not spaces**. If you indent with spaces, the spaces will remain in the output. This is one reason some Bash style guides recommend using tabs for indentation in heredocs.

---

## 4.5 Using Heredocs with SSH

One of the most powerful uses of heredocs is running multiple commands on a remote machine over SSH in one go:

```bash
#!/usr/bin/env bash

REMOTE_USER="student"
SSH_KEY="$HOME/.ssh/ocean"
REMOTE_HOST="debian-vm"

ssh -i "$SSH_KEY" "${REMOTE_USER}@${REMOTE_HOST}" << EOF
  sudo apt update
  sudo apt upgrade -y
  sudo apt autoremove -y
EOF
```

All three `apt` commands are sent to the remote machine and executed there, one after another.

```
Your machine
    |
    |--- SSH connection ----> Remote machine
    |    [heredoc block]          runs: sudo apt update
    |                             runs: sudo apt upgrade -y
    |                             runs: sudo apt autoremove -y
```

### Variable Expansion: Local vs. Remote

This is a critical distinction when using heredocs with SSH.

**Unquoted delimiter — variables expand on YOUR machine first:**

```bash
MYVAR="local-value"

ssh user@remote << EOF
  echo "$MYVAR"    # prints "local-value" — expanded before being sent
EOF
```

**Quoted delimiter — variables expand on the REMOTE machine:**

```bash
ssh user@remote << 'EOF'
  echo "$HOSTNAME"   # prints the remote machine's hostname
EOF
```

This distinction matters a lot. If you want to pass a value *from* your local machine to the remote command, use an unquoted delimiter. If you want the remote machine to evaluate its own variables, use a quoted delimiter.

---

## 4.6 Heredoc Pitfalls

| Pitfall | What Happens | Fix |
|---------|-------------|-----|
| Closing delimiter has trailing spaces | Heredoc never ends — script hangs | Remove trailing whitespace after delimiter |
| Closing delimiter is indented (when using `<<`) | Not recognized as the end | Use `<<-` with tab indentation, or remove indentation |
| Wrong case on delimiter | `EOF` ≠ `eof` | Delimiter is case-sensitive — be consistent |
| Using spaces with `<<-` | Spaces are NOT stripped, only tabs | Use actual tab characters for indentation |

### When to Use a Heredoc vs. a File

- **Use a heredoc** when the content is short, script-specific, and doesn't need to be reused
- **Use a file** when the content is large, complex, or reused in multiple scripts
- **Use a heredoc instead of multiple `echo` statements** — it's cleaner and easier to read

---

---

# 5. Getting User Input with `read`

## What Is `read`?

`read` is a Bash builtin command that reads a line of input — either from the user typing in the terminal, or from a file/stream — and stores it into one or more variables.

It's the primary way to make a Bash script **interactive** — asking the user for information and then using that information in the script's logic.

---

## 5.1 Basic Usage

```bash
read variable_name
```

When Bash reaches this line, the script pauses and waits for the user to type something and press Enter. Whatever the user types is stored in `variable_name`.

```bash
read name
echo "You said your name is: $name"
```

**Terminal:**

```
alice
You said your name is: alice
```

---

## 5.2 Reading Into Multiple Variables

You can provide multiple variable names to `read`. Bash splits the input into words (based on `$IFS`, which defaults to whitespace) and assigns them left to right. **Any leftover words go into the last variable.**

```bash
read first rest
```

Input: `one two three four`

```bash
echo "$first"   # one
echo "$rest"    # two three four  ← all remaining words go here
```

This is useful for parsing simple structured input.

---

## 5.3 The `-p` Option: Prompting the User

Without `-p`, you need a separate `echo` to tell the user what to enter:

```bash
echo "Enter your age:"
read age
```

The `-p` option lets you combine the prompt and the `read` in one line, which is cleaner:

```bash
read -p "Enter your age: " age
echo "You are $age years old."
```

---

## 5.4 Silent Input for Passwords (`-s`)

When asking for a password, you don't want what the user types to appear on the screen. The `-s` flag makes input **silent** — the user types, but nothing is displayed:

```bash
read -s -p "Enter password: " password
echo ""   # print a newline (the user's Enter is silent too)
echo "Password stored (not shown)"
```

You can combine multiple flags: `-s` for silent and `-p` for prompt.

---

## 5.5 Using `read` in a Case Statement

Here's a practical example from a script that prompts the user to enter a character and then responds based on what they entered:

```bash
#!/usr/bin/env bash

read -p "Enter a character: " character

case "$character" in
  A)
    echo "You entered A"
    ;;
  B|b)
    echo "You entered B or b"
    ;;
  C)
    echo "You entered C"
    ;;
  [1-3])
    echo "You entered 1, 2, or 3"
    ;;
  *)
    echo "Unrecognized input"
    ;;
esac
```

The `|` in `B|b` means "B or b" — case statements support multiple patterns per branch.

---

## 5.6 Reading Files Line by Line with a `while` Loop

A very common pattern in Bash is using `read` with a `while` loop to process a file one line at a time:

```bash
#!/usr/bin/env bash

while IFS= read -r line; do
  echo "Hello, $line!"
done < names.txt
```

**Anatomy of this pattern:**

| Part | Meaning |
|------|---------|
| `IFS=` | Set IFS to empty — prevents `read` from trimming leading/trailing whitespace from each line |
| `read -r` | The `-r` flag prevents backslashes in the input from being treated as escape sequences |
| `line` | The variable that holds the current line |
| `< names.txt` | Redirect the file as standard input to the `while` loop |

If `names.txt` contains:

```
Alice
Mo
Jaz
```

Output:

```
Hello, Alice!
Hello, Mo!
Hello, Jaz!
```

---

## 5.7 `read` vs. `mapfile` vs. `cat`

Choosing the right tool depends on what you're trying to do:

| Tool | Best For | When to Use |
|------|----------|-------------|
| `read` | Interactive input from user, or streaming through a file line-by-line | When you need to process lines as they come, or get keyboard input |
| `mapfile` | Loading an entire file into an array | When you want random access to any line, or clean syntax |
| `cat` / `less` | Displaying a file's contents | When you just want to see the file, not process it |

### `mapfile` Example (for Comparison)

```bash
#!/usr/bin/env bash

mapfile -t lines < names.txt

for line in "${lines[@]}"; do
  echo "Hello, $line!"
done

# Random access:
echo "${lines[0]}"   # prints: Alice (first line)
echo "${lines[2]}"   # prints: Jaz (third line)
```

The `-t` flag strips the trailing newline from each line (so lines don't end with `\n`).

> **Which should you use?** Use `read` in a `while` loop when the file is large (it processes line-by-line without loading everything into memory). Use `mapfile` when the file is manageable in size and you want clean, readable code with the ability to access any line by index.

---

---

# 6. Writing Functions in Bash

## What Is a Function?

A function in Bash is a named block of code that you define once and can call (run) many times by name. Functions help you:

- **Avoid repetition** — write a block of logic once, use it everywhere
- **Organize scripts** — break a large script into smaller, named, logical units
- **Create reusable tools** — like a `usage()` function that prints help text

---

## 6.1 Defining and Calling Functions

There are two valid syntaxes. The second (without the `function` keyword) is more common and portable:

```bash
# Syntax 1: with the 'function' keyword
function greet {
    echo "Hello!"
}

# Syntax 2: POSIX-compatible (preferred)
greet() {
    echo "Hello!"
}

# Calling the function:
greet   # Hello!
```

> **Critical Rule:** Functions must be **defined before they are called**. If you call a function that hasn't been defined yet (i.e., it appears later in the script), Bash will report "command not found."

---

## 6.2 Passing Arguments to Functions

Arguments are passed to functions the same way arguments are passed to scripts — using positional parameters `$1`, `$2`, `$3`, etc.

```bash
say_hi() {
    echo "Hi, $1!"
}

say_hi "Worf"    # Hi, Worf!
say_hi "Deanna"  # Hi, Deanna!
```

`$@` inside a function gives you all arguments passed to the function (not to the whole script):

```bash
print_all() {
    for name in "$@"; do
        echo "Argument: $name"
    done
}

print_all "Alice" "Bob" "Carol"
# Argument: Alice
# Argument: Bob
# Argument: Carol
```

Note that inside a function, `$0` still refers to the **script name** (not the function name).

---

## 6.3 Default Argument Values

You can provide a fallback value for a function argument using `${1:-"default"}`:

```bash
say_hi_default() {
    local name="${1:-"Guest"}"   # Use $1 if provided; otherwise use "Guest"
    echo "Hello, $name!"
}

say_hi_default          # Hello, Guest!
say_hi_default "Bob"    # Hello, Bob!
```

The `:-` operator means "use this default if the variable is unset OR empty."

---

## 6.4 Local Variables

**By default, all variables in Bash are global** — even variables declared inside a function. This means a function can accidentally overwrite a variable from the main script.

```bash
# Dangerous: global variable leak
set_value() {
    result="inside_function"   # This is a global variable!
}

result="before"
set_value
echo "$result"   # inside_function — the function changed it!
```

Use the `local` keyword to restrict a variable to the function's scope:

```bash
safe_function() {
    local result="inside_function"   # Only exists inside this function
    echo "$result"
}

result="before"
safe_function    # prints: inside_function
echo "$result"   # prints: before — unchanged!
```

**Best Practice:** Always use `local` for variables inside functions unless you specifically intend them to be global.

---

## 6.5 Return Values in Bash (It's Not Like Other Languages)

This is one of the most misunderstood parts of Bash functions. In Python or JavaScript, `return` sends a value back to the caller. **In Bash, `return` only sets the exit status** (a number from 0–255).

- **Exit status 0** = success (like `true`)
- **Any other value** = failure (like an error code)

```bash
is_even() {
    (( $1 % 2 == 0 ))   # Arithmetic expression: 0 means true/success
}

is_even 4
echo $?   # 0 — 4 is even, success

is_even 7
echo $?   # 1 — 7 is not even, failure
```

`$?` holds the exit status of the last command.

### How to Actually Return Data from a Function

Since `return` can't send data back, you have two options:

**Option 1: Print the value and capture it with command substitution:**

```bash
get_name() {
    echo "Alice"
}

name="$(get_name)"   # run get_name, capture its output
echo "Name is: $name"   # Name is: Alice
```

**Option 2: Set a global variable:**

```bash
get_name() {
    result="Alice"   # Set a global variable (no 'local')
}

get_name
echo "Name is: $result"   # Name is: Alice
```

Option 1 is generally cleaner. Option 2 is used when you need to return multiple values.

### Returning an Error Status

```bash
fail() {
    echo "Something went wrong" >&2   # Write error to stderr (not stdout)
    return 1                           # Signal failure with exit status 1
}

fail
echo "Exit status: $?"   # Exit status: 1
```

The `>&2` redirects the error message to **standard error** — the conventional place for error messages in Linux. This keeps error messages separate from normal output and allows calling scripts to distinguish between them.

---

## 6.6 Returning Data via a Local Variable (Advanced Pattern)

Here's an advanced but clean pattern — using a local variable and command substitution together:

```bash
#!/usr/bin/env bash

get_result() {
    local func_result="some computed value"
    echo "$func_result"
}

output="$(get_result)"
echo "Result: $output"   # Result: some computed value
```

This function doesn't pollute global scope (thanks to `local`) and cleanly returns a value via `echo`.
