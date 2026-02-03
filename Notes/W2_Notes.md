# Introduction to Working in the Terminal: A Complete Beginner's Guide

---

## What is a Shell?

A **shell** is a program that acts as an intermediary between you and the operating system. When you type a command, the shell interprets it and tells the operating system what to do.

### Why Does the Shell Exist?

The operating system kernel (the core of Linux) doesn't understand human-friendly commands like "show me my files." It only understands low-level system calls. The shell translates your commands into actions the kernel can perform.

Think of it like this:
- **You** → speak English
- **Shell** → translator
- **Kernel** → speaks machine language

### Bash: The Default Shell

We'll be using **Bash** (Bourne Again SHell), which is:
- The default shell on most Linux distributions
- Available on macOS (though zsh is now default)
- Available on Windows via WSL or Git Bash

### Two Ways to Use the Shell

| Method | Description | Example |
|--------|-------------|---------|
| **Interactive** | Type commands one at a time, see results immediately | Typing `ls` and pressing Enter |
| **Shell Scripts** | Store multiple commands in a file, run them all at once | A `.sh` file that backs up your data |

### Shell Built-ins vs External Programs

The shell has two types of commands:

| Type | What It Is | Examples |
|------|------------|----------|
| **Built-ins** | Features built directly into the shell | `cd`, `echo`, `help`, `export` |
| **External Programs** | Separate programs installed on your system | `ls`, `grep`, `python` |

**Why does this matter?** Built-ins run faster (no need to start a new process) and can modify the shell's own state (like changing directories).

#### Exploring Built-ins

To see all Bash built-ins:
```bash
help
```

To get help on a specific built-in:
```bash
help cd
help test
```

#### Check Your Current Shell

```bash
echo $SHELL
```

**Sample output:**
```
/bin/bash
```

### What My Professor Didn't Explain

- **`$SHELL`** is an **environment variable**—a named value stored in your shell session. The `$` tells the shell to substitute the variable's value.
- Other popular shells include **zsh** (macOS default), **fish** (user-friendly), and **sh** (minimal, POSIX-compliant).
- Your **login shell** (what `$SHELL` shows) might differ from your **current shell**. To see what you're actually running right now:
  ```bash
  echo $0
  ```

---

## What is a Process?

A **process** is a running instance of a program. It's not just the code—it's everything the program needs to execute.

### Why Understanding Processes Matters

Every command you run creates a process. Understanding processes helps you:
- Figure out why your computer is slow (what's using resources?)
- Stop programs that are stuck or misbehaving
- Run programs in the background
- Understand how Linux manages multiple programs simultaneously

### What a Process Includes

| Component | Description |
|-----------|-------------|
| **Program code** | The instructions being executed |
| **Memory** | Space for variables, data structures |
| **File descriptors** | Connections to files, network sockets |
| **I/O channels** | Standard input, output, and error streams |
| **Process ID (PID)** | Unique identifier assigned by the kernel |
| **Parent process ID (PPID)** | The process that created this one |

### The Life Cycle of a Process

Let's trace what happens when you run `python script.py`:

```
┌─────────────────────────────────────────────────────────────────┐
│  1. You type: python script.py                                  │
├─────────────────────────────────────────────────────────────────┤
│  2. The shell (itself a process) receives your command          │
├─────────────────────────────────────────────────────────────────┤
│  3. The shell "forks" — creates a copy of itself                │
├─────────────────────────────────────────────────────────────────┤
│  4. The copy "execs" — replaces itself with the python program  │
├─────────────────────────────────────────────────────────────────┤
│  5. Python runs your script                                     │
├─────────────────────────────────────────────────────────────────┤
│  6. Script finishes → Python process terminates                 │
├─────────────────────────────────────────────────────────────────┤
│  7. Shell receives notification, shows you a new prompt         │
└─────────────────────────────────────────────────────────────────┘
```

### What My Professor Didn't Explain

- **Fork** means the shell creates an almost-exact copy of itself. This is how Linux creates new processes.
- **Exec** means "replace this process with a different program." The forked shell becomes Python.
- This **fork-exec** pattern is fundamental to how Linux works. Every process (except PID 1) was created this way.

---

## Process IDs (PIDs)

Every process has a unique **Process ID (PID)**—a number assigned by the kernel when the process starts.

### Key Facts About PIDs

| Fact | Explanation |
|------|-------------|
| PIDs are unique | No two running processes have the same PID |
| PIDs are temporary | If you close and reopen a program, it gets a new PID |
| PIDs are assigned sequentially | But they wrap around and get reused |
| **PID 1 is special** | Always the init system (systemd on modern Linux) |

### Why PID 1 is Special

PID 1 is the **first process** started by the kernel at boot. It's the ancestor of all other processes. On modern Linux systems, this is usually **systemd**.

If PID 1 dies, the system crashes. That's why it's protected.

### Viewing Process Information

#### The `ps` Command

```bash
ps -ef
```

**What the options mean:**
- `-e` — show **every** process (not just yours)
- `-f` — **full** format (more columns)

**Sample output:**
```
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 09:00 ?        00:00:03 /usr/lib/systemd/systemd
root         2     0  0 09:00 ?        00:00:00 [kthreadd]
user      1234  1100  0 09:15 pts/0    00:00:00 bash
user      5678  1234  0 09:20 pts/0    00:00:00 ps -ef
```

**Column meanings:**

| Column | Meaning |
|--------|---------|
| `UID` | User who owns the process |
| `PID` | Process ID |
| `PPID` | Parent Process ID (who created this process) |
| `C` | CPU utilization |
| `STIME` | Start time |
| `TTY` | Terminal associated with the process (`?` means none) |
| `TIME` | CPU time consumed |
| `CMD` | The command that started this process |

#### The `/proc` Directory

Linux exposes process information through a virtual filesystem at `/proc`. Each running process has a directory named after its PID:

```bash
ls /proc
```

You'll see numbered directories (PIDs) and various system information files.

---

## Process States

Processes aren't always actively running—they spend most of their time waiting. The **state** tells you what a process is currently doing.

### Viewing Process States

```bash
ps -ax
```

**Sample output:**
```
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     0:03 /usr/lib/systemd/systemd
  234 ?        S      0:00 /usr/sbin/cron
 1234 pts/0    R+     0:00 ps -ax
```

The **STAT** column shows the state.

### Process State Codes Explained

#### Primary States (First Letter)

| Code | Name | Meaning |
|------|------|---------|
| `R` | Running | Actively using CPU or ready to run |
| `S` | Sleeping (interruptible) | Waiting for something (input, timer, etc.) |
| `D` | Sleeping (uninterruptible) | Waiting for I/O (usually disk)—can't be interrupted |
| `T` | Stopped | Paused by a signal (like Ctrl+Z) |
| `Z` | Zombie | Finished but parent hasn't acknowledged yet |
| `I` | Idle | Kernel thread doing nothing |

#### Additional Flags (Following Letters)

| Code | Meaning |
|------|---------|
| `s` | Session leader (started a group of processes) |
| `+` | In the foreground (attached to terminal) |
| `l` | Multi-threaded |
| `<` | High priority |
| `N` | Low priority (nice) |
| `L` | Has memory pages locked |

### Understanding Common States

**`Ss`** — Sleeping, session leader
- Example: Your shell waiting for you to type a command

**`R+`** — Running, in foreground
- Example: The `ps` command itself while it's executing

**`S+`** — Sleeping, in foreground
- Example: A program waiting for user input

### What My Professor Didn't Explain

- **Most processes are sleeping most of the time.** Even a "running" web browser spends 99%+ of its time waiting for network data, user input, or timers.
- **Zombie processes** (`Z`) aren't harmful in small numbers. They're just entries in the process table waiting for their parent to call `wait()`. If you see many zombies, the parent process has a bug.
- **Uninterruptible sleep** (`D`) usually means waiting for disk I/O. If a process is stuck in `D` state, it might indicate disk problems.

### Finding More Information

To see the full documentation on process states:
```bash
man ps
```
Then search for "PROCESS STATE CODES" by typing `/PROCESS STATE` and pressing Enter.

---

## Exit Status: How Programs Report Success or Failure

When a program finishes, it returns an **exit status** (also called exit code or return code)—a number that indicates whether it succeeded or failed.

### The Convention

| Exit Status | Meaning |
|-------------|---------|
| `0` | Success—the program did what it was supposed to do |
| `1-255` | Failure—something went wrong |

### Checking Exit Status

The special variable **`$?`** holds the exit status of the most recently executed command:

```bash
ls /home
echo $?
```

**Output:**
```
user1  user2
0
```

The `0` means `ls` succeeded.

### Example: A Failed Command

```bash
ls /nonexistent-directory
echo $?
```

**Output:**
```
ls: cannot access '/nonexistent-directory': No such file or directory
2
```

The `2` indicates failure (specifically, "no such file" for `ls`).

### Example: Command Not Found

```bash
sweaty-klingons
echo $?
```

**Output:**
```
bash: sweaty-klingons: command not found
127
```

Exit code `127` specifically means "command not found."

### Common Exit Codes

| Code | Typical Meaning |
|------|-----------------|
| `0` | Success |
| `1` | General error |
| `2` | Misuse of command (bad arguments) |
| `126` | Command found but not executable |
| `127` | Command not found |
| `128+N` | Killed by signal N (e.g., 137 = killed by SIGKILL) |
| `130` | Terminated by Ctrl+C (SIGINT) |

### Why Exit Status Matters

Exit status is crucial for:
- **Shell scripts** — deciding what to do next based on whether a command succeeded
- **Automation** — CI/CD pipelines check exit codes to know if builds passed
- **Chaining commands** — `&&` only runs the next command if the previous one succeeded

**Example:**
```bash
mkdir new_folder && cd new_folder
```
This only `cd`s into `new_folder` if `mkdir` succeeded.

### What My Professor Didn't Explain

- **`$?` is overwritten by every command.** If you want to save it, assign it to a variable immediately:
  ```bash
  some_command
  status=$?
  echo "The exit status was: $status"
  ```
- Programs can use any number 0-255, but the conventions above are widely followed.
- **`set -e`** in a script makes it exit immediately if any command fails (returns non-zero).

---

## What Happens When You Run a Command?

Understanding this process helps you troubleshoot "command not found" errors and customize your shell.

### Step-by-Step Breakdown

Let's trace what happens when you type:
```bash
ls -la $HOME
```

#### Step 1: Tokenization

The shell breaks your input into **tokens** (pieces separated by whitespace):

| Token | What It Is |
|-------|------------|
| `ls` | The command |
| `-la` | Options/flags |
| `$HOME` | An argument (a variable) |

#### Step 2: Expansion

The shell performs various **expansions**—replacing special syntax with actual values:

| Expansion Type | Before | After |
|----------------|--------|-------|
| **Variable expansion** | `$HOME` | `/home/username` |
| **Glob expansion** | `*.txt` | `file1.txt file2.txt file3.txt` |
| **Tilde expansion** | `~` | `/home/username` |
| **Brace expansion** | `{a,b,c}` | `a b c` |

After expansion, our command becomes:
```bash
ls -la /home/username
```

#### Step 3: Command Lookup

The shell searches for the command (`ls`) in this order:

| Priority | Location | Example |
|----------|----------|---------|
| 1 | **Aliases** | `alias ls='ls --color=auto'` |
| 2 | **Shell functions** | Custom functions in `.bashrc` |
| 3 | **Built-ins** | `cd`, `echo`, `help` |
| 4 | **PATH directories** | `/usr/bin/ls` |

#### Step 4: Execution

Once found, the shell:
1. Forks a new process
2. Executes the command in that process
3. Waits for it to finish
4. Displays the prompt again

### The PATH Variable

**PATH** is an environment variable containing a colon-separated list of directories where the shell looks for commands.

```bash
echo $PATH
```

**Sample output:**
```
/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin
```

This means the shell will search:
1. `/usr/local/bin`
2. `/usr/bin`
3. `/bin`
4. `/usr/local/sbin`
5. `/usr/sbin`

**In that order.** The first match wins.

### What My Professor Didn't Explain

- **Order matters in PATH.** If you have two programs named `python`, the one in the first matching directory runs.
- **Current directory is NOT in PATH by default.** This is a security feature. To run a program in the current directory:
  ```bash
  ./my_script.sh
  ```
- To see where a command is located:
  ```bash
  which ls        # Shows the path
  type ls         # Shows if it's an alias, function, or file
  ```
- To see all locations of a command:
  ```bash
  type -a ls
  ```

---

## Essential Terminal Commands

### `man` — The Manual Pages

#### Purpose

The `man` command displays documentation for commands, system calls, and configuration files. It's your primary reference for learning how Linux tools work.

#### Why It Exists

Linux has thousands of commands, each with dozens of options. Instead of memorizing everything, you look it up in the manual.

#### Syntax

```bash
man <command>
```

#### Practical Examples

**Read the manual for `ls`:**
```bash
man ls
```

**Read the manual for `man` itself:**
```bash
man man
```

#### Navigating Man Pages

Man pages open in a **pager** (usually `less`). Key controls:

| Key | Action |
|-----|--------|
| `Space` or `f` | Next page |
| `b` | Previous page |
| `j` / `k` | Scroll down / up one line |
| `/pattern` | Search forward for "pattern" |
| `n` | Next search result |
| `N` | Previous search result |
| `q` | Quit |
| `h` | Help (show all controls) |

#### Searching for Man Pages

Don't know the exact command name? Search by keyword:

```bash
man -k user
```

This searches all man page titles and descriptions for "user."

#### Man Page Sections

Man pages are organized into numbered sections:

| Section | Contents | Example |
|---------|----------|---------|
| 1 | User commands | `man 1 ls` |
| 2 | System calls | `man 2 open` |
| 3 | Library functions | `man 3 printf` |
| 4 | Special files | `man 4 null` |
| 5 | File formats | `man 5 passwd` |
| 6 | Games | `man 6 fortune` |
| 7 | Miscellaneous | `man 7 regex` |
| 8 | Admin commands | `man 8 useradd` |

**Why sections matter:** Some names exist in multiple sections. For example, `passwd` is both a command (section 1) and a file format (section 5):

```bash
man passwd      # Shows section 1 (the command)
man 5 passwd    # Shows section 5 (the file format)
```

#### What My Professor Didn't Explain

- The **SYNOPSIS** section shows command syntax. Square brackets `[]` mean optional; angle brackets `<>` mean required.
- The **SEE ALSO** section at the bottom lists related commands.
- **`info`** is an alternative documentation system with more detailed tutorials for some GNU tools:
  ```bash
  info coreutils
  ```

---

### `cd` — Change Directory

#### Purpose

Navigate between directories in the filesystem.

#### Why It's a Built-in

`cd` must be a shell built-in because it changes the shell's own working directory. An external program couldn't do this—it would change its own directory, then exit, leaving the shell unchanged.

#### Syntax

```bash
cd [directory]
```

#### Practical Examples

**Move into a subdirectory (relative path):**
```bash
cd Documents
cd Documents/school
```

**Move to an absolute path:**
```bash
cd /etc/apt
cd /var/log
```

**Move to your home directory:**
```bash
cd ~
cd $HOME
cd          # Just 'cd' alone also works!
```

**Move to the previous directory:**
```bash
cd -
```

**Move up one directory:**
```bash
cd ..
```

**Move up two directories:**
```bash
cd ../..
```

#### Understanding Paths

| Path Type | Starts With | Example | Meaning |
|-----------|-------------|---------|---------|
| **Absolute** | `/` | `/home/user/Documents` | Full path from root |
| **Relative** | Anything else | `Documents` or `./Documents` | Relative to current location |
| **Home** | `~` | `~/Documents` | Relative to home directory |

#### Special Directory Names

| Symbol | Meaning |
|--------|---------|
| `.` | Current directory |
| `..` | Parent directory |
| `~` | Home directory |
| `-` | Previous directory (for `cd` only) |

#### What My Professor Didn't Explain

- **Tab completion** is your friend! Type `cd Doc` and press Tab to autocomplete to `cd Documents/`.
- If a directory name has spaces, quote it or escape the spaces:
  ```bash
  cd "My Documents"
  cd My\ Documents
  ```

---

### `pwd` — Print Working Directory

#### Purpose

Shows the full absolute path of your current location in the filesystem.

#### When to Use It

- When you're lost and need to know where you are
- In scripts that need to know their location
- When copying paths to use elsewhere

#### Syntax

```bash
pwd
```

#### Practical Example

```bash
cd /var/log
pwd
```

**Output:**
```
/var/log
```

#### What My Professor Didn't Explain

- Your shell prompt often shows the current directory, but it might be abbreviated (e.g., `~` for home, or just the last directory name).
- `pwd -P` shows the **physical** path, resolving any symbolic links.

---

### `ls` — List Directory Contents

#### Purpose

Display files and directories. This is probably the command you'll use most often.

#### Syntax

```bash
ls [options] [directory]
```

#### Practical Examples

**List files in current directory:**
```bash
ls
```

**List with details (long format):**
```bash
ls -l
```

**Sample output:**
```
-rw-r--r-- 1 user group  4096 Jan 15 10:30 file.txt
drwxr-xr-x 2 user group  4096 Jan 14 09:00 Documents
```

**Understanding the output:**

```
-rw-r--r-- 1 user group 4096 Jan 15 10:30 file.txt
│├──┴──┴─┤ │  │    │     │       │          │
│    │    │  │    │     │       │          └── Filename
│    │    │  │    │     │       └── Modification date/time
│    │    │  │    │     └── Size in bytes
│    │    │  │    └── Group owner
│    │    │  └── User owner
│    │    └── Number of hard links
│    └── Permissions (owner/group/others)
└── File type (- = file, d = directory, l = link)
```

**List all files including hidden (starting with `.`):**
```bash
ls -a
```

**Combine options:**
```bash
ls -la
ls -al    # Same thing, order doesn't matter
```

**List a specific directory:**
```bash
ls /etc
```

**List recursively (show subdirectories):**
```bash
ls -R
ls -Rla /etc
```

**Human-readable file sizes:**
```bash
ls -lh
```

**Output:**
```
-rw-r--r-- 1 user group 4.0K Jan 15 10:30 file.txt
-rw-r--r-- 1 user group 2.3M Jan 15 10:30 video.mp4
```

#### Common Options Summary

| Option | Meaning |
|--------|---------|
| `-l` | Long format (details) |
| `-a` | All files (including hidden) |
| `-h` | Human-readable sizes |
| `-R` | Recursive (include subdirectories) |
| `-t` | Sort by modification time |
| `-S` | Sort by size |
| `-r` | Reverse sort order |

#### What My Professor Didn't Explain

- **Hidden files** in Linux start with a dot (`.`). Examples: `.bashrc`, `.ssh/`
- **`ls -la`** is so common that many people create an alias: `alias ll='ls -la'`
- Colors in `ls` output indicate file types (directories, executables, links, etc.). This is usually enabled by default via an alias.

---

### `cp` — Copy Files and Directories

#### Purpose

Create copies of files or directories.

#### Syntax

```bash
cp [options] <source> <destination>
```

#### Practical Examples

**Copy a file to another location:**
```bash
cp file.txt /backup/
```

**Copy and rename:**
```bash
cp file.txt file_backup.txt
```

**Copy a directory (must use `-r` for recursive):**
```bash
cp -r Documents/ /backup/Documents/
```

**Copy multiple files to a directory:**
```bash
cp file1.txt file2.txt file3.txt /backup/
```

**Preserve permissions and timestamps:**
```bash
cp -p file.txt /backup/
```

#### Common Options

| Option | Meaning |
|--------|---------|
| `-r` | Recursive (required for directories) |
| `-i` | Interactive (prompt before overwriting) |
| `-v` | Verbose (show what's being copied) |
| `-p` | Preserve permissions, ownership, timestamps |

#### ⚠️ Warning

`cp` will **silently overwrite** existing files by default. Use `-i` to be prompted:
```bash
cp -i file.txt /backup/
```

---

### `mv` — Move or Rename Files

#### Purpose

Move files to a different location, or rename them (which is technically the same operation in Linux).

#### Syntax

```bash
mv [options] <source> <destination>
```

#### Practical Examples

**Rename a file:**
```bash
mv old_name.txt new_name.txt
```

**Move a file to another directory:**
```bash
mv file.txt Documents/
```

**Move and rename simultaneously:**
```bash
mv file.txt Documents/renamed_file.txt
```

**Move multiple files:**
```bash
mv file1.txt file2.txt Documents/
```

#### Common Options

| Option | Meaning |
|--------|---------|
| `-i` | Interactive (prompt before overwriting) |
| `-v` | Verbose (show what's being moved) |
| `-n` | No clobber (don't overwrite existing files) |

#### What My Professor Didn't Explain

- **`./`** means "current directory." So `mv file ./dir` moves `file` into `dir` which is in the current directory.
- Unlike `cp`, `mv` doesn't need `-r` for directories—it just updates the directory's location in the filesystem.

---

### `rm` — Remove Files and Directories

#### Purpose

Delete files and directories permanently.

#### ⚠️ CRITICAL WARNING

> **`rm` is NOT a trash can. Deleted files are GONE FOREVER.**
> 
> There is no "undo." There is no recycle bin. The data is immediately unlinked from the filesystem.

#### Syntax

```bash
rm [options] <file(s)>
```

#### Practical Examples

**Remove a single file:**
```bash
rm file.txt
```

**Remove multiple files:**
```bash
rm file1.txt file2.txt file3.txt
```

**Remove files matching a pattern:**
```bash
rm *.txt      # All .txt files
rm *.log      # All .log files
```

**Remove a directory and its contents:**
```bash
rm -r directory/
```

**Force removal (no prompts, ignore errors):**
```bash
rm -f file.txt
```

**Remove directory forcefully (DANGEROUS):**
```bash
rm -rf directory/
```

#### Common Options

| Option | Meaning |
|--------|---------|
| `-r` | Recursive (required for directories) |
| `-f` | Force (don't prompt, ignore nonexistent files) |
| `-i` | Interactive (prompt for each file) |
| `-v` | Verbose (show what's being deleted) |

#### ⚠️ Dangerous Commands to NEVER Run

```bash
# NEVER run these commands:
rm -rf /                    # Deletes EVERYTHING
rm -rf /*                   # Deletes EVERYTHING
rm -rf ~                    # Deletes your entire home directory
rm -rf $VARIABLE/           # If VARIABLE is empty, this becomes rm -rf /
```

#### Best Practices

1. **Use `-i` when deleting multiple files:**
   ```bash
   rm -ri old_files/
   ```

2. **Double-check glob patterns with `ls` first:**
   ```bash
   ls *.txt          # See what would be deleted
   rm *.txt          # Then delete
   ```

3. **Use `trash-cli` for a safer alternative:**
   ```bash
   # Install: sudo apt install trash-cli
   trash file.txt    # Moves to trash instead of deleting
   ```

---

### `mkdir` — Make Directories

#### Purpose

Create new directories.

#### Syntax

```bash
mkdir [options] <directory_name(s)>
```

#### Practical Examples

**Create a single directory:**
```bash
mkdir projects
```

**Create multiple directories:**
```bash
mkdir dir1 dir2 dir3
```

**Create nested directories (parent directories too):**
```bash
mkdir -p projects/2024/january
```

Without `-p`, this would fail if `projects/` or `projects/2024/` don't exist.

#### Common Options

| Option | Meaning |
|--------|---------|
| `-p` | Create parent directories as needed |
| `-v` | Verbose (show directories as they're created) |

---

### `echo` — Print Text to Standard Output

#### Purpose

Display text or variable values. Essential for scripts and debugging.

#### Syntax

```bash
echo [options] <text>
```

#### Practical Examples

**Print a message:**
```bash
echo "Hello, World!"
```

**Print a variable:**
```bash
echo $HOME
echo $PATH
echo $USER
```

**Print without trailing newline:**
```bash
echo -n "Enter your name: "
```

**Print with escape sequences:**
```bash
echo -e "Line 1\nLine 2\tTabbed"
```

**Output:**
```
Line 1
Line 2	Tabbed
```

#### What My Professor Didn't Explain

- **Single quotes** preserve everything literally:
  ```bash
  echo '$HOME'    # Prints: $HOME
  ```
- **Double quotes** allow variable expansion:
  ```bash
  echo "$HOME"    # Prints: /home/username
  ```

---

### `cat` — Concatenate and Display Files

#### Purpose

Display file contents, or combine multiple files.

#### Why It's Called "cat"

Short for "concatenate"—its original purpose was to join files together. Displaying a single file is just concatenating one file with nothing.

#### Syntax

```bash
cat [options] <file(s)>
```

#### Practical Examples

**Display a file:**
```bash
cat .bashrc
```

**Display with line numbers:**
```bash
cat -n .bashrc
```

**Concatenate multiple files:**
```bash
cat file1.txt file2.txt > combined.txt
```

**Display non-printing characters:**
```bash
cat -A file.txt    # Shows tabs as ^I, line endings as $
```

#### Common Options

| Option | Meaning |
|--------|---------|
| `-n` | Number all lines |
| `-b` | Number non-blank lines only |
| `-A` | Show all (non-printing characters, tabs, line endings) |
| `-s` | Squeeze multiple blank lines into one |

#### What My Professor Didn't Explain

- For **large files**, use `less` instead of `cat`:
  ```bash
  less large_file.log
  ```
- To see just the beginning or end of a file:
  ```bash
  head -20 file.txt    # First 20 lines
  tail -20 file.txt    # Last 20 lines
  ```

---

### `uname` — Display System Information

#### Purpose

Show information about the system (kernel, architecture, hostname).

#### Syntax

```bash
uname [options]
```

#### Practical Examples

**Show kernel name:**
```bash
uname
```
**Output:** `Linux`

**Show kernel release version:**
```bash
uname -r
```
**Output:** `5.15.0-52-generic`

**Show all information:**
```bash
uname -a
```
**Output:**
```
Linux hostname 5.15.0-52-generic #58-Ubuntu SMP x86_64 GNU/Linux
```

#### Common Options

| Option | Meaning |
|--------|---------|
| `-a` | All information |
| `-r` | Kernel release |
| `-s` | Kernel name |
| `-n` | Network hostname |
| `-m` | Machine hardware name (architecture) |
| `-o` | Operating system |

---

### `ps` — Process Snapshot

#### Purpose

Display information about running processes.

#### Syntax

```bash
ps [options]
```

#### Practical Examples

**Show your processes:**
```bash
ps
```

**Show all processes (BSD style):**
```bash
ps aux
```

**Show all processes (System V style):**
```bash
ps -ef
```

**Show specific information:**
```bash
ps -eo pid,comm,rss,%mem,%cpu
```

This shows: PID, command name, memory (RSS), memory %, CPU %

**Find a specific process:**
```bash
ps -ef | grep bash
ps aux | grep firefox
```

**Show memory usage of bash:**
```bash
ps -eo comm,rss,pid | grep -i bash
```

#### Common Options

| Option | Meaning |
|--------|---------|
| `-e` | Every process |
| `-f` | Full format |
| `-a` | All with terminals |
| `-u` | User-oriented format |
| `-x` | Include processes without terminals |
| `-o` | Custom output format |

#### What My Professor Didn't Explain

- **`ps aux`** and **`ps -ef`** show similar information but in different formats (BSD vs System V style).
- For real-time process monitoring, use **`top`** or **`htop`**:
  ```bash
  top
  htop    # More user-friendly, may need to install
  ```

---

## Quick Reference Card

### Navigation
```bash
cd directory      # Change directory
cd ..             # Go up one level
cd ~              # Go to home
cd -              # Go to previous directory
pwd               # Print current directory
```

### Viewing Files
```bash
ls                # List files
ls -la            # List all with details
cat file          # Display file contents
less file         # Page through file
head -n 20 file   # First 20 lines
tail -n 20 file   # Last 20 lines
```

### File Operations
```bash
cp src dest       # Copy file
cp -r src dest    # Copy directory
mv src dest       # Move/rename
rm file           # Delete file
rm -r dir         # Delete directory
mkdir dir         # Create directory
mkdir -p a/b/c    # Create nested directories
```

### Getting Help
```bash
man command       # Manual page
command --help    # Quick help
help builtin      # Help for shell built-ins
type command      # What type of command is it?
which command     # Where is the command?
```

### Process Information
```bash
ps aux            # All processes
ps -ef            # All processes (different format)
echo $?           # Exit status of last command
```

---

## Additional Resources

Many of these resources are available through the [BCIT Library's O'Reilly Learning access](https://libguides.bcit.ca/az.php?a=o).

- [Bash Manual](https://www.gnu.org/software/bash/manual/) — Official GNU Bash documentation
- [Linux for System Administrators](https://learning.oreilly.com/library/view/linux-for-system/9781803247946/) by Viorel Rudareanu and Daniil Baturin
  - Chapter 2: The Shell and Its Commands
  - Chapter 4: Processes and Process Control
- [The Linux Command Line](https://linuxcommand.org/tlcl.php) — Free book by William Shotts
- [ExplainShell](https://explainshell.com/) — Paste any command to see what each part does
