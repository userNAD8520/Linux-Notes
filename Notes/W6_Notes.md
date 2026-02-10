## Table of Contents

1. [File Permissions in Linux](#1-file-permissions-in-linux)
2. [The Shebang: Making Scripts Executable](#2-the-shebang-making-scripts-executable)
3. [Writing Your First Bash Script](#3-writing-your-first-bash-script)
4. [Variables in Bash](#4-variables-in-bash)
5. [Conditionals in Bash](#5-conditionals-in-bash)
6. [Loops in Bash](#6-loops-in-bash)

---

# 1. File Permissions in Linux

## What Are File Permissions and Why Do They Exist?

In Linux, **every single file and directory** has a set of permissions attached to it. These permissions determine three fundamental things:

- **Who** can access the file
- **What** they can do with it (read it, modify it, or execute it as a program)
- **How** the system protects your data from unauthorized access

Unlike Windows (where permissions can be complex and hidden), Linux makes permissions visible and manageable through simple commands. This design philosophy comes from Linux's origins as a multi-user operating system where many people might use the same computer simultaneously.

### The Three Permission Types

Every file has three types of permissions:

1. **Read (r)** - Permission to view the file's contents or list a directory's contents
2. **Write (w)** - Permission to modify the file or create/delete files in a directory
3. **Execute (x)** - Permission to run a file as a program or enter a directory

### The Three Permission Categories

These permissions apply to three different categories of users:

1. **Owner (u)** - The user who created the file (usually you)
2. **Group (g)** - Users who are members of the file's group
3. **Others (o)** - Everyone else on the system

## Viewing File Permissions with `ls -l`

### Purpose

The `ls -l` command shows you the **long listing format** of files and directories, which includes detailed information about permissions, ownership, size, and modification dates.

### Syntax

```bash
ls -l [directory_or_file]
```

- `ls` - List directory contents
- `-l` - Use long listing format (shows detailed information)
- `[directory_or_file]` - Optional: specify what to list (defaults to current directory)

### Example

```bash
ls -l
```

**Sample Output:**

```
-rw-r--r-- 1 john developers  4096 Jan 28 10:30 notes.md
drwxr-xr-x 2 john developers  4096 Jan 27 14:22 scripts
-rwxr-xr-x 1 john developers   512 Jan 26 09:15 backup.sh
```

### Breaking Down the Output

Let's examine the first line in detail:

```
-rw-r--r-- 1 john developers  4096 Jan 28 10:30 notes.md
```

Here's what each part means:

| Part | Example | Meaning |
|------|---------|---------|
| File type | `-` | Regular file (use `d` for directory, `l` for symbolic link) |
| Owner permissions | `rw-` | Owner can read and write, but not execute |
| Group permissions | `r--` | Group members can only read |
| Other permissions | `r--` | Everyone else can only read |
| Link count | `1` | Number of hard links to this file |
| Owner name | `john` | The user who owns the file |
| Group name | `developers` | The group that owns the file |
| Size | `4096` | File size in bytes |
| Modification date | `Jan 28 10:30` | When the file was last modified |
| Filename | `notes.md` | The name of the file |

### Understanding the Permission String

The permission string `-rw-r--r--` can be broken down into four parts:

```
-  rw-  r--  r--
│   │    │    │
│   │    │    └─ Others: read only
│   │    └────── Group: read only  
│   └─────────── Owner: read and write
└─────────────── File type: regular file
```

**Common file type indicators:**

- `-` Regular file (text file, script, binary)
- `d` Directory (folder)
- `l` Symbolic link (shortcut to another file)
- `b` Block device (like a hard drive)
- `c` Character device (like a terminal)

## Changing File Permissions with `chmod`

### Purpose

The `chmod` command (change mode) allows you to modify who can read, write, or execute a file. This is essential when:

- Making a script executable so you can run it
- Protecting sensitive files from being read by others
- Allowing multiple users to collaborate on files

### Why You'll Need This

When you write a Bash script, Linux treats it as a regular text file by default. To actually **run** the script as a program, you must add execute permission using `chmod`.

### Syntax

```bash
chmod [who][operation][permission] filename
```

**Components:**

- **who**: `u` (user/owner), `g` (group), `o` (others), or `a` (all)
- **operation**: `+` (add), `-` (remove), or `=` (set exactly)
- **permission**: `r` (read), `w` (write), `x` (execute)

### Example: Making a Script Executable

```bash
chmod u+x my_script.sh
```

**What this does:**

- `u` - Apply to the owner (user)
- `+` - Add permission (don't remove existing ones)
- `x` - Execute permission
- `my_script.sh` - The file to modify

**Before:**
```
-rw-r--r-- 1 john developers  512 Jan 28 10:30 my_script.sh
```

**After:**
```
-rwxr--r-- 1 john developers  512 Jan 28 10:30 my_script.sh
```

Notice the first permission group changed from `rw-` to `rwx`.

### Common `chmod` Examples

```bash
# Make a script executable for everyone
chmod a+x script.sh

# Remove write permission for group and others (protect the file)
chmod go-w important.txt

# Give the owner all permissions, but remove all permissions for others
chmod u+rwx,go-rwx secret.sh

# Make a file read-only for everyone (including yourself)
chmod a-w document.txt
```

## Numeric (Octal) Permissions

### What My Professor Didn't Explain

Permissions can also be represented as **three-digit numbers** (like `755` or `644`). This is often faster once you understand it, and you'll see it frequently in documentation and scripts.

Each digit represents one category (owner, group, others), and each number is the **sum** of permission values:

- **Read (r) = 4**
- **Write (w) = 2**
- **Execute (x) = 1**
- **No permission (-) = 0**

### Permission Calculation Table

| Permission Pattern | Calculation | Decimal Value |
|-------------------|-------------|---------------|
| `---` | 0 + 0 + 0 | 0 (no permissions) |
| `--x` | 0 + 0 + 1 | 1 (execute only) |
| `-w-` | 0 + 2 + 0 | 2 (write only) |
| `-wx` | 0 + 2 + 1 | 3 (write and execute) |
| `r--` | 4 + 0 + 0 | 4 (read only) |
| `r-x` | 4 + 0 + 1 | 5 (read and execute) |
| `rw-` | 4 + 2 + 0 | 6 (read and write) |
| `rwx` | 4 + 2 + 1 | 7 (all permissions) |

### Putting It Together

A three-digit permission like `755` means:

```
7       5       5
│       │       │
│       │       └─ Others: r-x (4+1=5)
│       └───────── Group:  r-x (4+1=5)
└───────────────── Owner:  rwx (4+2+1=7)
```

### Example: Numeric vs. Symbolic

These commands do the same thing:

```bash
# Symbolic method
chmod u+x script.sh

# Numeric method (assuming current permissions are rw-r--r--)
chmod 744 script.sh
```

**Result:** `-rwxr--r--`

### Common Numeric Permission Patterns

```bash
# 755 - Owner can do everything, others can read and execute
# Common for scripts and programs
chmod 755 my_script.sh

# 644 - Owner can read/write, others can only read
# Common for data files and documents
chmod 644 notes.txt

# 600 - Owner can read/write, no one else can access
# Common for private files like SSH keys
chmod 600 ~/.ssh/id_rsa

# 700 - Owner has full control, no one else can access
# Common for private directories
chmod 700 ~/private_data
```

### Beginner Tip: Which Method to Use?

- Use **symbolic** (`u+x`) when making small changes to existing permissions
- Use **numeric** (`755`) when setting permissions from scratch or when you know exactly what you want

## Warnings and Best Practices

### ⚠️ Dangerous Permissions

```bash
# DON'T DO THIS - Makes your files world-writable
chmod 777 file.txt  # Anyone can modify your files!

# DON'T DO THIS - Removes all your permissions
chmod 000 file.txt  # Even you can't access it anymore!
```

### ✅ Safe Practices

1. **Start restrictive, then add permissions as needed**
   ```bash
   # Create files with minimal permissions, then add as needed
   chmod 600 newfile.txt
   chmod u+x newfile.txt  # Add execute only when necessary
   ```

2. **Never use `777` unless absolutely necessary** (and it's almost never necessary)

3. **Protect sensitive files immediately**
   ```bash
   # Good for SSH keys, API tokens, passwords
   chmod 600 ~/.ssh/id_rsa
   ```

4. **Use `chmod +x` for scripts, not `chmod 777`**
   ```bash
   # Good
   chmod u+x script.sh
   
   # Bad
   chmod 777 script.sh
   ```

---

# 2. The Shebang: Making Scripts Executable

## What Is a Shebang and Why Does It Exist?

A **shebang** (also called a hashbang) is a special line at the very beginning of a script file that tells the operating system **which program should run this script**.

### The Problem It Solves

Imagine you create a file called `my_script`. When you try to run it, how does Linux know whether it should:

- Run it with Bash?
- Run it with Python?
- Run it with Perl?
- Try to execute it directly as binary code?

Without a shebang, Linux doesn't know. **The shebang solves this ambiguity** by explicitly declaring the interpreter.

### What It Looks Like

```bash
#!/bin/bash
```

Or:

```bash
#!/usr/bin/env bash
```

### Breaking Down the Shebang

```
#  !  /bin/bash
│  │  │
│  │  └─ Path to the interpreter program
│  └──── Bang (exclamation mark)
└─────── Hash (also called pound or number sign)
```

Together, `#!` is pronounced "hash-bang" → "shebang"

## How the Shebang Works

### Step-by-Step Execution

When you run a script with a shebang:

1. You type `./my_script` in the terminal
2. The Linux kernel opens the file and reads the first two bytes
3. If those bytes are `#!`, the kernel knows this is an interpreted script
4. The kernel reads the rest of the line to find the interpreter path
5. The kernel launches that interpreter and passes your script to it
6. The interpreter runs your code

### Example

**File: `hello.sh`**
```bash
#!/bin/bash
echo "Hello, World!"
```

When you run `./hello.sh`, the kernel essentially does:

```bash
/bin/bash ./hello.sh
```

## Common Shebang Patterns

### Bash Scripts

```bash
#!/bin/bash
```

**When to use:** Most Bash scripts, especially when you need Bash-specific features (arrays, extended pattern matching, etc.)

**Advantages:**

- Runs with Bash specifically
- Consistent behavior across different systems
- Access to all Bash features

### Using `env` to Find Bash

```bash
#!/usr/bin/env bash
```

**When to use:** When you want maximum portability across different Unix-like systems
   - Widely accepted as **best practice**, this method covers the vast majority of use cases—particularly when sharing code or working across different operating systems 

**How it works:**

- `env` is a program that finds executables in the user's `PATH`
- Instead of hard-coding `/bin/bash`, `env` searches for `bash` wherever it's installed
- On most systems, Bash is in `/bin/bash`, but on some (like macOS with Homebrew), it might be in `/usr/local/bin/bash` or `/opt/homebrew/bin/bash`

**Advantages:**

- More portable across different Unix-like systems
- Respects user's environment (useful in virtual environments)

**Trade-offs:**

- Slightly slower (has to search for the interpreter)
- Depends on `PATH` configuration

### POSIX Shell

```bash
#!/bin/sh
```

**When to use:** When you need maximum compatibility and don't use Bash-specific features

**What My Professor Didn't Explain:**

`sh` is **not the same as Bash**. On different systems, `/bin/sh` might be:

- Dash (Debian/Ubuntu) - faster but fewer features
- Bash in POSIX mode (macOS, older systems)
- Original Bourne shell (very old systems)

If your script uses Bash-specific features (like arrays or `[[` conditionals), using `#!/bin/sh` will cause it to fail on some systems.

### Other Interpreters

**Python:**
```python
#!/usr/bin/env python3

print("Hello from Python")
```

**Perl:**
```perl
#!/usr/bin/env perl

print "Hello from Perl\n";
```

**Ruby:**
```ruby
#!/usr/bin/env ruby

puts "Hello from Ruby"
```

## Why Use a Shebang Instead of Typing the Interpreter?

### Without Shebang (Manual Method)

You have to remember and type the interpreter each time:

```bash
python3 my_script.py
bash my_script.sh
perl my_script.pl
```

**Problems:**

- Extra typing every time
- Easy to forget which interpreter a script needs
- Scripts can't be used as regular commands
- Breaks automation and integration

### With Shebang (Automatic Method)

The script becomes self-documenting and executable:

```bash
./my_script
```

**Benefits:**

1. **Guaranteed Correct Interpreter**
   - The script always runs with the right interpreter
   - No guessing or remembering required
   
2. **Behaves Like a Real Command**
   - Can be placed in `PATH` directories like `/usr/local/bin`
   - Can be called by name: `my_script` instead of `./my_script`
   
3. **Better for Automation**
   - Cron jobs can call scripts without knowing the interpreter
   - Other scripts can execute it reliably
   
4. **Self-Documenting**
   - Opening the file immediately shows what language it's written in
   - Reduces confusion in large projects with mixed languages

### Real-World Example

Imagine you have a directory with scripts in multiple languages:

```
scripts/
├── backup.sh          (Bash)
├── analyze_logs.py    (Python)
├── process_data.pl    (Perl)
└── deploy.rb          (Ruby)
```

**Without shebangs:** You need to remember which is which
```bash
bash backup.sh
python3 analyze_logs.py
perl process_data.pl
ruby deploy.rb
```

**With shebangs:** They all work the same way
```bash
./backup.sh
./analyze_logs.py
./process_data.pl
./deploy.rb
```

## Critical Requirements for Shebangs

### 1. Must Be the First Line

```bash
# WRONG - shebang must be first, no blank lines before it

#!/bin/bash
```

```bash
# CORRECT
#!/bin/bash

# Your script starts here
```

### 2. Must Start at Column 1

```bash
# WRONG - has a space before it
 #!/bin/bash
```

```bash
# CORRECT
#!/bin/bash
```

### 3. No Extra Spaces After `#!`

```bash
# ACCEPTABLE but not recommended
#! /bin/bash

# BEST PRACTICE
#!/bin/bash
```

## Making a Script Executable

Having a shebang is only half the battle. The file also needs **execute permission**.

### Complete Steps to Create and Run a Script

```bash
# 1. Create the script
cat > hello.sh << 'EOF'
#!/bin/bash
echo "Hello, World!"
EOF

# 2. Make it executable
chmod u+x hello.sh

# 3. Run it
./hello.sh
```

### What My Professor Didn't Explain: Why `./`?

When you type `hello.sh` without the `./`, Bash looks for the command in your `PATH` (standard directories like `/usr/bin`, `/bin`, etc.). Your current directory (`.`) is usually **not** in the `PATH` for security reasons.

The `./` explicitly tells Bash: "Run the executable file named `hello.sh` in the current directory (`.`)."

## Common Beginner Mistakes

### Mistake 1: Forgetting Execute Permission

```bash
# Creating a script
echo '#!/bin/bash' > script.sh
echo 'echo "Hello"' >> script.sh

# Trying to run it
./script.sh
# Error: Permission denied
```

**Solution:**
```bash
chmod u+x script.sh
./script.sh
```

### Mistake 2: Wrong Shebang Path

```bash
#!/usr/bin/bashh  # Typo! Extra 'h'
echo "This won't run"
```

**Error:** `bad interpreter: No such file or directory`

### Mistake 3: Using Wrong Interpreter

```bash
#!/bin/sh  # Using sh instead of bash

# But script uses Bash-specific features
declare -a my_array=("one" "two" "three")  # Arrays don't work in sh!
```

**Solution:** Use `#!/bin/bash` when you use Bash features

### Mistake 4: Windows Line Endings

If you created the script on Windows and transferred it to Linux, it might have Windows line endings (`\r\n` instead of just `\n`).

**Error:** `/bin/bash^M: bad interpreter`

**Solution:**
```bash
# Convert Windows line endings to Unix
dos2unix script.sh

# Or use sed
sed -i 's/\r$//' script.sh
```

## Best Practices

1. **Always include a shebang in executable scripts**
   
2. **Use `#!/usr/bin/env bash` for portability** unless you have a specific reason to hard-code the path
   
3. **Use `#!/bin/bash` for scripts that:**
   - Use Bash-specific features
   - Will only run on systems where Bash is guaranteed to be in `/bin`
   
4. **Use `#!/bin/sh` only when:**
   - Writing POSIX-compliant scripts
   - Avoiding all Bash-specific features
   - Targeting minimal systems

5. **Document your script** with comments after the shebang:
   ```bash
   #!/usr/bin/env bash
   
   # Purpose: Daily backup script
   # Author: Your Name
   # Date: 2025-02-08
   # Usage: ./backup.sh [destination]
   ```

---

# 3. Writing Your First Bash Script

## What Is a Bash Script?

A Bash script is simply a **text file containing a series of commands** that you would normally type at the command line, saved so you can run them all at once. Think of it as a recipe or checklist that the computer follows step by step.

### Why Write Scripts?

Scripts allow you to:

- **Automate repetitive tasks** (backups, file organization, system maintenance)
- **Chain multiple commands** together into a single workflow
- **Save complex command sequences** for later use
- **Reduce human error** by eliminating manual typing

## Creating Your First Script: Hello World

### Step-by-Step Instructions

#### 1. Open Your Text Editor

We'll use `vim`, a powerful terminal-based text editor available on virtually every Linux system.

```bash
vim hello
```

This creates a new file called `hello` (note: no `.sh` extension yet—we'll discuss why below).

#### 2. Enter Insert Mode

When Vim opens, you're in "normal mode" (for navigation). To type text:

- Press `i` to enter **insert mode**

You should see `-- INSERT --` at the bottom of the screen.

#### 3. Type Your Script

```bash
#!/usr/bin/env bash

echo "hello world"
```

**Line-by-line explanation:**

- `#!/usr/bin/env bash` - The shebang tells Linux to run this file with Bash
- (blank line) - Improves readability (optional but good practice)
- `echo "hello world"` - The `echo` command prints text to the screen

#### 4. Save and Exit Vim

- Press `Esc` to return to normal mode
- Type `:wq` and press `Enter`
  - `:w` = write (save)
  - `:q` = quit
  - Together: `:wq` = write and quit

#### 5. Make the Script Executable

```bash
chmod u+x hello
```

This adds execute permission for you (the owner).

**Verify it worked:**
```bash
ls -l hello
```

You should see: `-rwxr--r--` (notice the `x` in the owner permissions)

#### 6. Run Your Script

```bash
./hello
```

**Expected output:**
```
hello world
```

**Success!** You've just written and executed your first Bash script.

## Understanding the `./` Prefix

### What My Professor Didn't Explain

When you type a command like `ls` or `grep`, Bash finds these programs by searching directories listed in your `PATH` environment variable (typically `/usr/bin`, `/bin`, `/usr/local/bin`, etc.).

Your current directory (`.`) is **not** in the `PATH` by default for security reasons. If it were, a malicious user could create a fake `ls` program in their home directory, and you might accidentally run it instead of the real `ls`.

### The Solution: Explicit Path

The `./` prefix means "current directory." So `./hello` explicitly tells Bash:

"Run the executable file named `hello` that's in my current working directory (`.`)."

### Alternatives

If you want to run `hello` without `./`, you can:

1. **Add it to a directory in your PATH:**
   ```bash
   # Copy to a directory in PATH
   sudo cp hello /usr/local/bin/
   
   # Now you can run it from anywhere
   hello
   ```

2. **Add the current directory to PATH (not recommended for security):**
   ```bash
   export PATH=".:$PATH"  # Don't do this permanently!
   ```

## File Extensions: To `.sh` or Not to `.sh`?

### The Philosophy

In the Unix/Linux world, **commands don't usually have file extensions**. Look at standard commands:

```bash
ls      # Not ls.bash
grep    # Not grep.c
python  # Not python.exe
vim     # Not vim.c
```

This design makes the implementation invisible to users. Whether a command is written in C, Bash, Python, or compiled assembly doesn't matter—it's just a command.

### When to Use `.sh`

**Use `.sh` extension when:**

- You're learning and want clear indication that it's a shell script
- Working on a project with scripts in multiple languages
- Your organization or project has a style guide requiring it
- The script is a library or module, not meant to be a command

**Example:**
```bash
backup_library.sh  # Source this, don't execute it
helpers.sh         # Functions to be sourced
```

### When to Omit `.sh`

**Omit `.sh` extension when:**

- The script is meant to be used like a regular command
- You want implementation details hidden from users
- Following Unix conventions for distributed scripts

**Example:**
```bash
backup      # Looks like a real command
deploy      # User doesn't care that it's written in Bash
clean-logs  # Could be Bash, Python, or compiled C
```

### What the Shebang Does for Extensions

The shebang makes file extensions unnecessary because:

1. The kernel reads the shebang to find the interpreter
2. The interpreter is explicitly specified
3. Users don't need to guess from the extension

## Vim Tips for Beginners

### Setting Syntax Highlighting Without an Extension

If you create a file without a `.sh` extension, Vim might not automatically use Bash syntax highlighting.

#### Method 1: Add Shebang First, Then Reload

```bash
# 1. Open file
vim hello

# 2. Type the shebang as the first line
#!/usr/bin/env bash

# 3. Save and reload
:w
:e
```

The `:e` command (edit) reloads the current file. Vim will detect the shebang and apply Bash syntax highlighting.

#### Method 2: Manually Set Syntax

```bash
# While in Vim
:set syntax=bash
```

This immediately enables Bash syntax highlighting without saving and reloading.

#### Method 3: Make It Permanent

Add to your `~/.vimrc`:

```vim
" Automatically detect Bash scripts by shebang
autocmd BufRead,BufNewFile * if getline(1) =~ '^#!/bin/bash\|^#!/usr/bin/env bash' | set ft=bash | endif
```

### Essential Vim Commands for Beginners

| Command | Purpose |
|---------|---------|
| `i` | Enter insert mode (before cursor) |
| `a` | Enter insert mode (after cursor) |
| `A` | Enter insert mode at end of line |
| `o` | Open new line below and enter insert mode |
| `O` | Open new line above and enter insert mode |
| `Esc` | Return to normal mode |
| `:w` | Write (save) file |
| `:q` | Quit |
| `:wq` | Write and quit |
| `:q!` | Quit without saving |
| `dd` | Delete current line |
| `yy` | Copy current line |
| `p` | Paste |
| `u` | Undo |
| `Ctrl + r` | Redo |
| `/pattern` | Search for pattern |
| `n` | Next search result |
| `:set number` | Show line numbers |
| `:help <topic>` | Get help on a topic |

### Getting Help in Vim

Vim has extensive built-in documentation:

```vim
:help
:help edit
:help :w
:help insert
```

Press `:q` to exit help.

## The `echo` Command

### Purpose

`echo` prints text (or variable values) to the terminal. It's the simplest way to display information to users or debug scripts.

### Syntax

```bash
echo [options] [string]
```

### Basic Examples

```bash
# Print simple text
echo "Hello, World!"
# Output: Hello, World!

# Print without quotes (works if no special characters)
echo Hello World
# Output: Hello World

# Print multiple arguments (separated by spaces)
echo Hello     World
# Output: Hello World
# Note: Multiple spaces collapsed into one

# Print with variables
name="Alice"
echo "Hello, $name"
# Output: Hello, Alice

# Print special characters
echo "The cost is \$50"
# Output: The cost is $50
```

### Common Options

```bash
# -n: Don't print trailing newline
echo -n "Loading..."
# Output: Loading... (cursor stays on same line)

# -e: Enable interpretation of backslash escapes
echo -e "Line 1\nLine 2"
# Output:
# Line 1
# Line 2

echo -e "Column1\tColumn2"
# Output: Column1    Column2
```

### Escape Sequences (with `-e`)

| Sequence | Meaning |
|----------|---------|
| `\n` | Newline |
| `\t` | Tab |
| `\\` | Backslash |
| `\"` | Double quote |
| `\a` | Alert (bell) |

## Improving Your Hello Script

Let's make the script more professional:

```bash
#!/usr/bin/env bash

#
# hello - A simple greeting script
# Usage: ./hello
#

echo "Hello, World!"
echo "Welcome to Bash scripting!"
echo
echo "Today is $(date +%A), $(date +%B) $(date +%d), $(date +%Y)"
echo "Current user: $USER"
echo "Current directory: $PWD"
```

**New concepts introduced:**

- **Comments:** Lines starting with `#` (except the shebang) are ignored by Bash
- **Command substitution:** `$(command)` runs a command and inserts its output
- **Built-in variables:** `$USER` and `$PWD` are automatically set by Bash

**Sample output:**
```
Hello, World!
Welcome to Bash scripting!

Today is Saturday, February 08, 2025
Current user: john
Current directory: /home/john/scripts
```

## Common Beginner Mistakes

### Mistake 1: Forgetting to Make Script Executable

```bash
# Create script
vim hello

# Try to run it
./hello
# Error: Permission denied
```

**Solution:**
```bash
chmod u+x hello
./hello
```

### Mistake 2: Typo in Shebang

```bash
#!/bin/bish  # Typo: 'bish' instead of 'bash'
```

**Error:** `bad interpreter: No such file or directory`

### Mistake 3: Forgetting `./` Prefix

```bash
hello
# Error: command not found
```

**Solution:**
```bash
./hello
```

### Mistake 4: Using Unsaved Changes

Don't forget to save (`:w`) before running your script! Bash executes the saved version on disk, not what you see in your editor.

## Next Steps

Now that you have a basic script working, you can:

1. Add more `echo` commands to print different messages
2. Experiment with command substitution: `$(date)`, `$(whoami)`, `$(pwd)`
3. Try using variables (we'll cover this in detail in the next section)

---

# 4. Variables in Bash

## What Are Variables and Why Do They Exist?

A **variable** is a named container that stores data. Think of it like a labeled box where you can put information and retrieve it later.

### Why Use Variables?

Variables allow you to:

- **Store data for reuse** (avoiding repetition)
- **Make scripts flexible** (change one value instead of many)
- **Store command output** (capture results to use later)
- **Make code readable** (descriptive names instead of raw values)

### Real-World Analogy

Imagine you're writing instructions for mailing packages:

**Without variables (repetitive):**
```
Put "John Smith" on the label
Write "123 Main St" under "John Smith"
Add "John Smith" to the return address
Email "John Smith" a confirmation
```

**With variables (efficient):**
```
recipient = "John Smith"
address = "123 Main St"

Put recipient on the label
Write address under recipient
Add recipient to return address
Email recipient a confirmation
```

If the recipient changes, you only update one line instead of four.

## Creating Variables in Bash

### Syntax Rules (CRITICAL)

```bash
variable_name="value"
```

**Rules you MUST follow:**

1. **No spaces around `=`** (this is the #1 beginner mistake)
2. **Variable names must start with a letter or underscore**
3. **Variable names can contain letters, numbers, and underscores**
4. **Variable names are case-sensitive**

### Valid Variable Declarations

```bash
# Good examples
name="Alice"
age=25
_count=100
user_name="bob"
FILE_PATH="/home/user/data.txt"
myVar123="test"
```

### Invalid Variable Declarations

```bash
# WRONG: Space before =
name ="Alice"

# WRONG: Space after =
name= "Alice"

# WRONG: Spaces around =
name = "Alice"

# WRONG: Starts with a number
2fast="value"

# WRONG: Contains hyphens
my-var="value"

# WRONG: Contains spaces in name
my var="value"
```

### What My Professor Didn't Explain: Why No Spaces?

In Bash, spaces separate commands and arguments. If you write:

```bash
name = "Alice"
```

Bash interprets this as:

- `name` - A command
- `=` - First argument to that command
- `"Alice"` - Second argument to that command

It tries to run a program called `name` with arguments `=` and `"Alice"`, which fails.

## Using (Expanding) Variables

To **use** a variable's value, prefix its name with `$`:

```bash
name="Alice"
echo $name
# Output: Alice

echo "Hello, $name"
# Output: Hello, Alice
```

### Curly Braces: When and Why

You can (and often should) wrap variable names in curly braces:

```bash
name="Alice"
echo "Hello, ${name}"
# Output: Hello, Alice
```

### When Curly Braces Are Required

#### 1. Appending Text Directly to a Variable

```bash
file="report"

# WRONG: Bash thinks the variable is named 'file_2025'
echo "$file_2025.txt"
# Output: .txt (variable doesn't exist, prints empty string + .txt)

# CORRECT: Curly braces separate variable name from suffix
echo "${file}_2025.txt"
# Output: report_2025.txt
```

#### 2. Accessing Array Elements

```bash
my_array=("first" "second" "third")

# WRONG: Doesn't work as expected
echo "$my_array[0]"

# CORRECT
echo "${my_array[0]}"
# Output: first
```

#### 3. Parameter Expansion (Advanced)

```bash
name="Alice"

# Convert to uppercase
echo "${name^^}"
# Output: ALICE

# Get string length
echo "${#name}"
# Output: 5

# Default value if empty
echo "${name:-Unknown}"
# Output: Alice (because name is set)

unset name
echo "${name:-Unknown}"
# Output: Unknown (because name is not set)
```

### Best Practice

**Always use curly braces unless you have a specific reason not to.** It prevents bugs and makes your code more maintainable.

```bash
# Good
echo "User: ${username}"

# Also okay, but less safe
echo "User: $username"
```

## Types of Data Variables Can Store

Bash variables are **untyped**—they don't have a fixed data type. The same variable can hold different kinds of data:

### 1. Strings (Text)

```bash
greeting="Hello, World!"
path="/home/user/documents"
empty=""  # Empty string
```

### 2. Numbers

```bash
count=42
price=19.99  # Bash treats this as a string, but you can do math with it
negative=-5
```

**Important:** Bash stores everything as strings internally, but can perform arithmetic when needed.

### 3. Command Output (Command Substitution)

```bash
# Capture the current date
today=$(date +%Y-%m-%d)
echo "Today is $today"
# Output: Today is 2025-02-08

# Capture current directory
current_dir=$(pwd)
echo "You are in $current_dir"

# Capture number of files
file_count=$(ls | wc -l)
echo "There are $file_count files here"
```

**Older syntax (still works but less readable):**
```bash
today=`date +%Y-%m-%d`  # Backticks (deprecated style)
```

**Use `$()` instead of backticks**—it's clearer and can be nested.

### 4. Paths and Configuration

```bash
CONFIG_FILE="/etc/myapp/config.conf"
LOG_DIR="/var/log/myapp"
BACKUP_PATH="/backups/$(date +%Y%m%d)"
```

## Updating Your Hello Script to Use Variables

Let's modify the `hello` script to use a variable:

**Current version:**
```bash
#!/usr/bin/env bash

echo "hello world"
```

**New version:**
```bash
#!/usr/bin/env bash

my_first_name="Nathan"

echo "Hello $my_first_name"
```

**Test it:**
```bash
./hello
# Output: Hello Nathan
```

### Making It Interactive

Change the hard-coded name to use a positional parameter (explained next):

```bash
#!/usr/bin/env bash

my_first_name="$1"

echo "Hello $my_first_name"
```

Now you can pass a name when running the script:

```bash
./hello Alice
# Output: Hello Alice

./hello Bob
# Output: Hello Bob
```

## Special Variables in Bash

Bash automatically creates special variables that contain useful information:

### Positional Parameters (Script Arguments)

| Variable | Meaning | Example |
|----------|---------|---------|
| `$0` | Script name | `./hello` |
| `$1` | First argument | `Alice` (from `./hello Alice`) |
| `$2` | Second argument | `Smith` (from `./hello Alice Smith`) |
| `$3` - `$9` | Arguments 3-9 | Direct access |
| `${10}` | Tenth argument and beyond | Requires braces |
| `$#` | Number of arguments | `2` (from `./hello Alice Smith`) |
| `$@` | All arguments as separate words | `"Alice" "Smith"` |
| `$*` | All arguments as single string | `"Alice Smith"` |

### Example Script Using Positional Parameters

```bash
#!/usr/bin/env bash

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Total number of arguments: $#"
echo "All arguments: $@"
```

**Running it:**
```bash
./script.sh apple banana cherry
```

**Output:**
```
Script name: ./script.sh
First argument: apple
Second argument: banana
Total number of arguments: 3
All arguments: apple banana cherry
```

### Process and Exit Status Variables

| Variable | Meaning |
|----------|---------|
| `$$` | Process ID of current shell |
| `$?` | Exit status of last command (0 = success, non-zero = error) |
| `$!` | Process ID of last background command |

### Example: Checking Command Success

```bash
#!/usr/bin/env bash

# Try to create a directory
mkdir /tmp/mydir

# Check if it succeeded
if [[ $? -eq 0 ]]; then
  echo "Directory created successfully"
else
  echo "Failed to create directory"
fi
```

**Better approach using `if` directly:**
```bash
#!/usr/bin/env bash

if mkdir /tmp/mydir; then
  echo "Directory created successfully"
else
  echo "Failed to create directory"
fi
```

### Environment Variables (System-Provided)

These are set automatically by the system:

| Variable | Meaning |
|----------|---------|
| `$HOME` | User's home directory (`/home/username`) |
| `$USER` | Current username |
| `$PWD` | Present working directory |
| `$SHELL` | Path to user's default shell |
| `$PATH` | Directories to search for commands |
| `$HOSTNAME` | System hostname |
| `$LANG` | System language |

### Example: Using Environment Variables

```bash
#!/usr/bin/env bash

echo "You are logged in as: $USER"
echo "Your home directory is: $HOME"
echo "You are currently in: $PWD"
echo "Your shell is: $SHELL"
```

## Quotes: Single vs. Double vs. None

### Double Quotes `" "`

**Variables ARE expanded** (replaced with their values):

```bash
name="Alice"
echo "Hello, $name"
# Output: Hello, Alice
```

**Command substitution WORKS:**
```bash
echo "Today is $(date +%A)"
# Output: Today is Saturday
```

**Use double quotes when:** You want variables and commands to be expanded.

### Single Quotes `' '`

**Everything is literal** (no expansion):

```bash
name="Alice"
echo 'Hello, $name'
# Output: Hello, $name (literally)
```

**Use single quotes when:** You want to preserve the exact text, including `$` signs.

### No Quotes

**Dangerous for variables with spaces:**

```bash
filename="my document.txt"

# WRONG: Treats "my" and "document.txt" as separate arguments
cat $filename
# Error: cat: my: No such file or directory
#        cat: document.txt: No such file or directory

# CORRECT: Treats entire filename as one argument
cat "$filename"
# Works correctly
```

**What My Professor Didn't Explain:**

Without quotes, Bash performs **word splitting** (breaks on spaces, tabs, newlines) and **pathname expansion** (expands wildcards like `*`).

```bash
files="*.txt"

# Without quotes: Expands to all .txt files
echo $files
# Output: file1.txt file2.txt file3.txt (list of files)

# With quotes: Literal string
echo "$files"
# Output: *.txt (literal)
```

### Best Practice: Always Quote Variables

```bash
# Good
echo "Name: $name"
rm "$filename"
cd "$directory"

# Risky (only safe if you're certain there are no spaces)
echo Name: $name
```

## The `declare` Built-in

The `declare` command gives you more control over variables:

### Syntax

```bash
declare [options] variable_name=value
```

### Common Options

#### Read-Only Variables (Constants)

```bash
declare -r PI=3.14159

# Try to change it
PI=3.14
# Error: PI: readonly variable
```

**Use for:** Values that should never change (configuration constants, important paths).

#### Integer Variables

```bash
declare -i count=10

count=count+5  # Arithmetic without $(( ))
echo $count
# Output: 15

count="hello"  # Tries to convert "hello" to integer
echo $count
# Output: 0 (conversion failed)
```

#### Export Variables (Make Them Environment Variables)

```bash
declare -x DATABASE_URL="postgresql://localhost/mydb"

# Now DATABASE_URL is available to child processes
python3 myapp.py  # Can access DATABASE_URL
```

**Equivalent to:**
```bash
export DATABASE_URL="postgresql://localhost/mydb"
```

#### Array Variables

```bash
# Indexed array
declare -a fruits=("apple" "banana" "cherry")
echo "${fruits[0]}"
# Output: apple

# Associative array (like a dictionary/map)
declare -A colors
colors[red]="#FF0000"
colors[green]="#00FF00"
echo "${colors[red]}"
# Output: #FF0000
```

### When to Use `declare`

- **Use for read-only constants:** `declare -r CONFIG_FILE="/etc/myapp.conf"`
- **Use for integers doing arithmetic:** `declare -i counter=0`
- **Use for exporting variables:** `declare -x` or just `export`
- **Use for arrays:** `declare -a` or `declare -A`

**For simple variables, direct assignment is fine:**
```bash
name="Alice"  # No need for declare
```

## Environment Variables

### What Are Environment Variables?

**Environment variables** are variables that are passed to **child processes** (programs started from your script).

### Creating Environment Variables

```bash
# Method 1: Using export
export DATABASE_URL="postgresql://localhost/mydb"

# Method 2: Using declare -x
declare -x API_KEY="abc123"

# Method 3: Export existing variable
MY_VAR="value"
export MY_VAR
```

### Why This Matters

**Without `export`:**
```bash
MY_PASSWORD="secret"

# Run a Python script
python3 check_password.py
# Python can't see MY_PASSWORD (not exported)
```

**With `export`:**
```bash
export MY_PASSWORD="secret"

# Run a Python script
python3 check_password.py
# Python can access MY_PASSWORD via os.environ
```

### Example: Using Environment Variables Across Languages

**Bash script:**
```bash
#!/usr/bin/env bash

export API_KEY="abc123"
export DATABASE_URL="postgresql://localhost/mydb"

# Call a Python script (inherits environment variables)
python3 fetch_data.py
```

**Python script (`fetch_data.py`):**
```python
#!/usr/bin/env python3

import os

api_key = os.environ.get("API_KEY")
db_url = os.environ.get("DATABASE_URL")

print(f"API Key: {api_key}")
print(f"Database: {db_url}")
```

**Perl script example:**
```perl
#!/usr/bin/env perl

use v5.40;

say "Home directory: $ENV{HOME}";
say "Current user: $ENV{USER}";
```

### Viewing All Environment Variables

```bash
# Show all environment variables
env

# Or
printenv

# Show a specific variable
echo $HOME
printenv HOME
```

### Removing Variables

```bash
# Create a variable
MY_VAR="test"

# Remove it
unset MY_VAR

# Now it's gone
echo $MY_VAR
# Output: (empty)
```

**Warning:** `unset` also works on environment variables, so be careful:
```bash
unset PATH  # DON'T DO THIS - breaks your ability to run commands!
```

## Variable Naming Conventions

### Standard Conventions

1. **Environment variables:** ALL_UPPERCASE
   ```bash
   export DATABASE_URL="..."
   export API_KEY="..."
   ```

2. **Local script variables:** lowercase_with_underscores
   ```bash
   user_name="alice"
   file_count=10
   ```

3. **Read-only constants:** ALL_UPPERCASE
   ```bash
   declare -r MAX_RETRIES=3
   declare -r CONFIG_FILE="/etc/app.conf"
   ```

### Why Follow Conventions?

- Makes code more readable
- Prevents conflicts with environment variables
- Easier for others to understand your code

## Common Beginner Mistakes

### Mistake 1: Spaces Around `=`

```bash
# WRONG
name = "Alice"

# CORRECT
name="Alice"
```

### Mistake 2: Using `$` When Assigning

```bash
# WRONG
$name="Alice"

# CORRECT (no $ when assigning)
name="Alice"

# $ only when using the variable
echo $name
```

### Mistake 3: Not Quoting Variables with Spaces

```bash
filename="my document.txt"

# WRONG
rm $filename
# Tries to remove two files: "my" and "document.txt"

# CORRECT
rm "$filename"
```

### Mistake 4: Forgetting to Export

```bash
# This won't be visible to child processes
API_KEY="secret"
python3 script.py  # Can't see API_KEY

# This will be visible
export API_KEY="secret"
python3 script.py  # Can see API_KEY
```

## Practice Exercises

### Exercise 1: Personal Greeting

Create a script that:
1. Takes your name as an argument
2. Stores it in a variable
3. Prints a personalized greeting

```bash
#!/usr/bin/env bash

name="$1"
echo "Hello, ${name}!"
echo "Welcome to Bash scripting."
```

### Exercise 2: File Information

Create a script that displays information about a file:

```bash
#!/usr/bin/env bash

filepath="$1"
filename=$(basename "$filepath")
directory=$(dirname "$filepath")
size=$(stat -f%z "$filepath" 2>/dev/null || stat -c%s "$filepath" 2>/dev/null)

echo "File: $filename"
echo "Directory: $directory"
echo "Size: $size bytes"
```

### Exercise 3: System Information

```bash
#!/usr/bin/env bash

echo "System Information"
echo "=================="
echo "Hostname: $HOSTNAME"
echo "User: $USER"
echo "Home: $HOME"
echo "Shell: $SHELL"
echo "Current directory: $PWD"
echo "Date: $(date)"
```

---

# 5. Conditionals in Bash

## What Are Conditionals and Why Do They Exist?

**Conditionals** allow your scripts to make decisions and execute different code based on conditions. They're like the "if-then" logic you use in everyday life:

- **If** it's raining, **then** take an umbrella
- **If** the file exists, **then** process it, **else** show an error
- **If** the user is root, **then** allow the operation, **else** deny it

Without conditionals, scripts could only execute commands in a fixed sequence. Conditionals make scripts intelligent and adaptable.

## The `test` Command

### Purpose

The `test` command evaluates a condition and returns:

- **0** (true/success) if the condition is true
- **1** (false/failure) if the condition is false

### What My Professor Didn't Explain: Exit Codes

In Unix/Linux, commands return **exit codes** (also called exit status):

- **0** = Success/True
- **Non-zero** (1, 2, 3, etc.) = Failure/False

This is **opposite** to many programming languages where 0 means false and 1 means true. In Bash:

```bash
test 5 -gt 3  # 5 greater than 3?
echo $?       # Check exit code
# Output: 0 (true)

test 2 -gt 10  # 2 greater than 10?
echo $?        # Check exit code
# Output: 1 (false)
```

### Basic Syntax

```bash
test EXPRESSION
```

Or using the `[` notation (exactly equivalent):

```bash
[ EXPRESSION ]
```

**Important:** Spaces are required around `[` and `]`:

```bash
# CORRECT
[ 5 -gt 3 ]

# WRONG (no spaces)
[5 -gt 3]
```

### Numeric Comparisons

| Operator | Meaning | Example |
|----------|---------|---------|
| `-eq` | Equal to | `[ $a -eq $b ]` |
| `-ne` | Not equal to | `[ $a -ne $b ]` |
| `-lt` | Less than | `[ $a -lt $b ]` |
| `-le` | Less than or equal to | `[ $a -le $b ]` |
| `-gt` | Greater than | `[ $a -gt $b ]` |
| `-ge` | Greater than or equal to | `[ $a -ge $b ]` |

### Examples

```bash
# Test with immediate feedback
test 100 -ge 10 && echo "True" || echo "False"
# Output: True

test 5 -lt 3 && echo "True" || echo "False"
# Output: False

# Using variables
age=25
test $age -ge 18 && echo "Adult" || echo "Minor"
# Output: Adult
```

### Understanding `&&` and `||`

- `&&` - "AND" - Runs the next command only if the previous succeeded (exit code 0)
- `||` - "OR" - Runs the next command only if the previous failed (exit code non-zero)

```bash
# command1 && command2
# "If command1 succeeds, then run command2"

# command1 || command2
# "If command1 fails, then run command2"

# Combined
test 10 -gt 5 && echo "Yes" || echo "No"
# Translation: If test succeeds, print "Yes", otherwise print "No"
```

## File Test Operators

Bash includes powerful built-in operators for testing files and directories.

### Common File Tests

| Operator | Meaning | Example |
|----------|---------|---------|
| `-e FILE` | File exists (any type) | `[ -e /etc/passwd ]` |
| `-f FILE` | File exists and is a regular file | `[ -f config.txt ]` |
| `-d FILE` | File exists and is a directory | `[ -d /home/user ]` |
| `-r FILE` | File exists and is readable | `[ -r secret.txt ]` |
| `-w FILE` | File exists and is writable | `[ -w data.log ]` |
| `-x FILE` | File exists and is executable | `[ -x script.sh ]` |
| `-s FILE` | File exists and is not empty | `[ -s output.txt ]` |
| `-L FILE` | File exists and is a symbolic link | `[ -L /usr/bin/python ]` |

### Complete List

To see all file test operators:

```bash
help test
```

Or:

```bash
man test
```

### Examples

```bash
# Check if passwd file exists
test -f /etc/passwd && echo "/etc/passwd exists"

# Check if a directory exists
test -d /home/user && echo "User home directory found" || echo "Not found"

# Check if a script is executable
test -x myscript.sh && echo "Executable" || echo "Not executable"

# Check if a file is not empty
test -s logfile.txt && echo "Log has content" || echo "Log is empty"
```

### Practical Example: Checking Before Action

```bash
#!/usr/bin/env bash

config_file="/etc/myapp/config.conf"

# Only read the file if it exists and is readable
if test -f "$config_file" && test -r "$config_file"; then
  echo "Loading configuration from $config_file"
  source "$config_file"
else
  echo "Error: Configuration file not found or not readable"
  exit 1
fi
```

## String Comparisons

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` or `==` | Strings are equal | `[ "$a" = "$b" ]` |
| `!=` | Strings are not equal | `[ "$a" != "$b" ]` |
| `-z` | String is empty (zero length) | `[ -z "$str" ]` |
| `-n` | String is not empty | `[ -n "$str" ]` |

### Examples

```bash
name="Alice"

# Check if equal
test "$name" = "Alice" && echo "Match"
# Output: Match

# Check if not equal
test "$name" != "Bob" && echo "Different"
# Output: Different

# Check if empty
test -z "$empty_var" && echo "Variable is empty"
# Output: Variable is empty

# Check if not empty
test -n "$name" && echo "Variable has a value"
# Output: Variable has a value
```

### Common Mistake: Not Quoting Strings

```bash
name="Alice Smith"

# WRONG: Breaks if variable contains spaces
test $name = "Alice Smith" && echo "Match"
# Error: too many arguments

# CORRECT: Always quote variables in tests
test "$name" = "Alice Smith" && echo "Match"
# Output: Match
```

## `[` vs. `[[` - Single vs. Double Brackets

### Single Brackets `[ ]`

- Part of POSIX standard
- Works in all shells (sh, bash, zsh, etc.)
- Requires careful quoting
- Limited pattern matching

```bash
[ "$name" = "Alice" ]
```

### Double Brackets `[[ ]]`

- Bash-specific (not POSIX)
- Safer (handles empty variables better)
- Supports pattern matching
- Supports `&&` and `||` inside
- Prevents word splitting and pathname expansion

```bash
[[ "$name" = "Alice" ]]
```

### Key Differences

**Pattern matching:**
```bash
# Single brackets - doesn't support patterns
[ "$filename" = *.txt ]  # Treats *.txt as literal string

# Double brackets - supports patterns
[[ "$filename" = *.txt ]]  # Matches any .txt file
```

**Empty variables:**
```bash
name=""

# Single brackets - can fail without quotes
[ $name = "Alice" ]  # Error: unary operator expected

# Double brackets - handles empty variables
[[ $name = "Alice" ]]  # Works fine
```

**Logical operators:**
```bash
# Single brackets - must use -a and -o
[ "$age" -gt 18 -a "$age" -lt 65 ]

# Double brackets - can use && and ||
[[ $age -gt 18 && $age -lt 65 ]]
```

### Best Practice: Use `[[` in Bash Scripts

```bash
# Preferred (if writing for Bash)
if [[ -f "$file" ]]; then
  echo "File exists"
fi

# Acceptable (if POSIX compatibility needed)
if [ -f "$file" ]; then
  echo "File exists"
fi
```

## The `if` Statement

### Purpose

The `if` statement runs code **only if** a condition is true.

### Syntax

```bash
if CONDITION; then
  COMMANDS
fi
```

Or on multiple lines:

```bash
if CONDITION
then
  COMMANDS
fi
```

### Example

```bash
#!/usr/bin/env bash

if [[ -d /home/user/backups ]]; then
  echo "Backup directory exists"
fi
```

**Translation:** "If the directory `/home/user/backups` exists, then print a message."

### Using Variables in Conditions

```bash
#!/usr/bin/env bash

age=25

if [[ $age -ge 18 ]]; then
  echo "You are an adult"
fi
```

### Multiple Commands in `then` Block

```bash
#!/usr/bin/env bash

if [[ -f important.txt ]]; then
  echo "Found important.txt"
  echo "Processing..."
  cat important.txt
  echo "Done"
fi
```

## The `else` Clause

### Syntax

```bash
if CONDITION; then
  COMMANDS_IF_TRUE
else
  COMMANDS_IF_FALSE
fi
```

### Example

```bash
#!/usr/bin/env bash

if [[ -f config.txt ]]; then
  echo "Configuration file found"
  source config.txt
else
  echo "Configuration file not found"
  echo "Using default settings"
fi
```

### Important Note: `else` Is Optional

**You don't always need `else`.** Many beginners think every `if` must have an `else`, but that's not true.

**Example where `else` is unnecessary:**

```bash
#!/usr/bin/env bash

# Check if required directory exists
if [[ ! -d "$HOME/.config" ]]; then
  echo "Error: Configuration directory missing"
  exit 1
fi

# Continue with rest of script
echo "Configuration directory found"
# ... rest of code ...
```

**Translation:** "If the directory doesn't exist, print an error and exit. Otherwise, continue normally."

This pattern (check for errors and exit early) is cleaner than wrapping all your code in an `else` block.

## The `elif` Clause (Else-If)

### Purpose

Use `elif` when you have **multiple exclusive conditions** to check.

### Syntax

```bash
if CONDITION1; then
  COMMANDS1
elif CONDITION2; then
  COMMANDS2
elif CONDITION3; then
  COMMANDS3
else
  COMMANDS_IF_ALL_FALSE
fi
```

### Example: Grade Calculator

```bash
#!/usr/bin/env bash

score=$1

if [[ $score -ge 90 ]]; then
  echo "Grade: A"
elif [[ $score -ge 80 ]]; then
  echo "Grade: B"
elif [[ $score -ge 70 ]]; then
  echo "Grade: C"
elif [[ $score -ge 60 ]]; then
  echo "Grade: D"
else
  echo "Grade: F"
fi
```

**Running it:**
```bash
./grade.sh 85
# Output: Grade: B
```

### Example: File Type Checker

```bash
#!/usr/bin/env bash

filepath="$1"

if [[ ! -e "$filepath" ]]; then
  echo "Error: $filepath does not exist"
elif [[ -f "$filepath" ]]; then
  echo "$filepath is a regular file"
elif [[ -d "$filepath" ]]; then
  echo "$filepath is a directory"
elif [[ -L "$filepath" ]]; then
  echo "$filepath is a symbolic link"
else
  echo "$filepath is a special file"
fi
```

## Logical Operators

### Combining Conditions

| Operator | Meaning | Example |
|----------|---------|---------|
| `&&` | AND (both must be true) | `[[ -f file.txt && -r file.txt ]]` |
| `||` | OR (at least one must be true) | `[[ -f file.txt || -f file.md ]]` |
| `!` | NOT (negates condition) | `[[ ! -f file.txt ]]` |

### AND Examples

```bash
# File must exist AND be readable
if [[ -f "$file" && -r "$file" ]]; then
  echo "File exists and is readable"
fi

# Multiple conditions
age=25
has_license=true

if [[ $age -ge 18 && $has_license = true ]]; then
  echo "You can drive"
fi
```

### OR Examples

```bash
# Accept either .txt or .md files
if [[ "$filename" = *.txt || "$filename" = *.md ]]; then
  echo "Text document detected"
fi

# Check multiple possible locations
if [[ -f /etc/config.conf || -f ~/.config/app.conf ]]; then
  echo "Found configuration file"
fi
```

### NOT Examples

```bash
# If file does NOT exist
if [[ ! -f important.txt ]]; then
  echo "Error: important.txt is missing"
  exit 1
fi

# If directory does NOT exist, create it
if [[ ! -d logs ]]; then
  mkdir logs
fi
```

### Complex Combinations

```bash
# Must be readable AND (writable OR owned by user)
if [[ -r "$file" && ( -w "$file" || -O "$file" ) ]]; then
  echo "You have access to this file"
fi
```

## Improved `hello` Script with Error Handling

Let's enhance the `hello` script with proper error checking:

```bash
#!/usr/bin/env bash

#
# hello - A greeting script with error handling
# Usage: ./hello [name]
#

# Function to print error messages
# (We'll learn about functions in detail next week)
err() {
  local message="$*"
  echo "Error: $message" >&2  # Print to stderr
  echo "Usage: ${0##*/} [name]" >&2
  exit 1
}

# Check if exactly one argument was provided
if [[ $# -ne 1 ]]; then
  err "Missing or too many arguments."
fi

my_first_name="$1"

echo "Hello $my_first_name"
```

### What's New?

**1. The `err` function:**
```bash
err() {
  local message="$*"
  echo "Error: $message" >&2
  echo "Usage: ${0##*/} [name]" >&2
  exit 1
}
```

- `>&2` - Redirects output to stderr (standard error) instead of stdout
- `${0##*/}` - Strips the path from the script name (shows just `hello`, not `./hello`)
- `exit 1` - Exits the script with error code 1 (indicates failure)

**2. Argument count check:**
```bash
if [[ $# -ne 1 ]]; then
  err "Missing or too many arguments."
fi
```

- `$#` - Number of arguments passed to the script
- `-ne` - Not equal to

### Testing It

```bash
# No arguments
./hello
# Output: Error: Missing or too many arguments.
#         Usage: hello [name]

# Too many arguments
./hello Alice Bob
# Output: Error: Missing or too many arguments.
#         Usage: hello [name]

# Correct usage
./hello Alice
# Output: Hello Alice
```

## The `case` Statement

### Purpose

The `case` statement is used for **pattern matching against a single value**. It's cleaner than long `if/elif` chains when checking one variable against multiple patterns.

### When to Use `case`

Use `case` when:

- Checking one variable against multiple specific values
- Implementing menu systems
- Processing command-line options
- Matching file extensions or patterns

Use `if/elif` when:

- Testing complex conditions
- Comparing different variables
- Using numeric comparisons

### Syntax

```bash
case EXPRESSION in
  PATTERN1)
    COMMANDS
    ;;
  PATTERN2)
    COMMANDS
    ;;
  *)
    DEFAULT_COMMANDS
    ;;
esac
```

**Key parts:**

- `case EXPRESSION in` - Starts the case statement
- `PATTERN)` - Pattern to match (can use wildcards)
- `;;` - Ends each pattern block (like `break` in other languages)
- `*)` - Default case (matches anything)
- `esac` - Ends the case statement (`case` backwards)

### Basic Example

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
    echo "You did not enter A, B, b, C, or 1-3"
    ;;
esac
```

### Pattern Matching Rules

| Pattern | Meaning | Example |
|---------|---------|---------|
| `literal` | Exact match | `yes)` matches "yes" |
| `pattern1\|pattern2` | Multiple patterns | `yes\|y)` matches "yes" or "y" |
| `[abc]` | Any character in brackets | `[aeiou])` matches any vowel |
| `[a-z]` | Character range | `[0-9])` matches any digit |
| `?` | Any single character | `?)` matches any single character |
| `*` | Any string | `*.txt)` matches any .txt file |

### Example: Menu System

```bash
#!/usr/bin/env bash

echo "Main Menu"
echo "1. Backup files"
echo "2. Restore files"
echo "3. Check status"
echo "4. Exit"
echo

read -p "Enter your choice: " choice

case "$choice" in
  1)
    echo "Starting backup..."
    # backup commands here
    ;;
  2)
    echo "Starting restore..."
    # restore commands here
    ;;
  3)
    echo "Checking status..."
    # status check here
    ;;
  4)
    echo "Goodbye!"
    exit 0
    ;;
  *)
    echo "Invalid choice. Please enter 1-4."
    exit 1
    ;;
esac
```

### Example: File Extension Handler

```bash
#!/usr/bin/env bash

filename="$1"

case "$filename" in
  *.txt)
    echo "Text file detected"
    cat "$filename"
    ;;
  *.jpg|*.jpeg|*.png)
    echo "Image file detected"
    # Open with image viewer
    ;;
  *.sh)
    echo "Shell script detected"
    bash "$filename"
    ;;
  *.py)
    echo "Python script detected"
    python3 "$filename"
    ;;
  *)
    echo "Unknown file type"
    ;;
esac
```

### Example: Yes/No Prompt

```bash
#!/usr/bin/env bash

read -p "Do you want to continue? (yes/no): " answer

case "$answer" in
  yes|y|Yes|YES)
    echo "Continuing..."
    ;;
  no|n|No|NO)
    echo "Aborting..."
    exit 0
    ;;
  *)
    echo "Please answer yes or no"
    exit 1
    ;;
esac
```

### Advanced Pattern: Command-Line Options

```bash
#!/usr/bin/env bash

case "$1" in
  -h|--help)
    echo "Usage: $0 [-h|--help] [-v|--version]"
    ;;
  -v|--version)
    echo "Version 1.0.0"
    ;;
  -d|--debug)
    echo "Debug mode enabled"
    # Enable debugging
    ;;
  *)
    echo "Unknown option: $1"
    exit 1
    ;;
esac
```

## When to Use What

### Use `if` for:

```bash
# File existence checks
if [[ -f "$config_file" ]]; then
  source "$config_file"
fi

# Numeric comparisons
if [[ $age -ge 18 ]]; then
  echo "Adult"
fi

# Complex logical conditions
if [[ -r "$file" && -w "$file" ]]; then
  echo "File is readable and writable"
fi
```

### Use `case` for:

```bash
# Menu systems
case "$choice" in
  1) do_action_1 ;;
  2) do_action_2 ;;
esac

# Pattern matching
case "$filename" in
  *.txt) handle_text ;;
  *.jpg) handle_image ;;
esac

# Processing flags/options
case "$1" in
  -h|--help) show_help ;;
  -v|--version) show_version ;;
esac
```

## Common Beginner Mistakes

### Mistake 1: Missing Spaces in `[` Tests

```bash
# WRONG
[$name = "Alice"]

# CORRECT
[ "$name" = "Alice" ]
```

### Mistake 2: Using `=` Instead of `-eq` for Numbers

```bash
# WRONG (compares as strings, "10" > "9" is false!)
if [[ 10 > 9 ]]; then

# CORRECT (numeric comparison)
if [[ 10 -gt 9 ]]; then
```

### Mistake 3: Not Quoting Variables

```bash
name="Alice Smith"

# WRONG
if [[ $name = "Alice Smith" ]]; then

# CORRECT
if [[ "$name" = "Alice Smith" ]]; then
```

### Mistake 4: Forgetting `;;` in `case`

```bash
# WRONG (missing ;;)
case "$x" in
  1)
    echo "One"
  2)
    echo "Two"
esac

# CORRECT
case "$x" in
  1)
    echo "One"
    ;;
  2)
    echo "Two"
    ;;
esac
```

---

# 6. Loops in Bash

## What Are Loops and Why Do They Exist?

**Loops** allow you to repeat commands multiple times without writing the same code over and over. They're essential for:

- Processing lists of items (files, users, numbers)
- Repeating actions until a condition is met
- Automating repetitive tasks
- Reading files line by line

### Real-World Analogy

Without loops:
```
Process file1.txt
Process file2.txt
Process file3.txt
... (100 more lines)
Process file100.txt
```

With loops:
```
For each file in the directory:
  Process that file
```

## The `for` Loop

### Purpose

The `for` loop iterates over a **list of items**, running commands once for each item.

### Basic Syntax

```bash
for VARIABLE in LIST; do
  COMMANDS
done
```

Or on multiple lines:

```bash
for VARIABLE in LIST
do
  COMMANDS
done
```

### Example: Count to 10

```bash
#!/usr/bin/env bash

for num in {1..10}; do
  echo "$num"
done
```

**Output:**
```
1
2
3
4
5
6
7
8
9
10
```

### How It Works

1. `for num in {1..10}` - Expands `{1..10}` into the sequence `1 2 3 4 5 6 7 8 9 10`
2. Assigns each value to `num`, one at a time
3. Runs the code between `do` and `done` for each value
4. Repeats until all values are processed

### Brace Expansion `{}`

Bash automatically expands brace expressions:

```bash
# Numbers
echo {1..5}
# Output: 1 2 3 4 5

# Letters
echo {a..e}
# Output: a b c d e

# Step by 2
echo {0..10..2}
# Output: 0 2 4 6 8 10

# Reverse
echo {5..1}
# Output: 5 4 3 2 1
```

### Example: Loop Over a List of Words

```bash
#!/usr/bin/env bash

for color in red green blue yellow; do
  echo "Color: $color"
done
```

**Output:**
```
Color: red
Color: green
Color: blue
Color: yellow
```

### Example: Loop Over Files

```bash
#!/usr/bin/env bash

# Process all .txt files in current directory
for file in *.txt; do
  echo "Processing $file"
  # Count lines in file
  wc -l "$file"
done
```

### Example: Loop Over Command Output

```bash
#!/usr/bin/env bash

# Loop over all users in /etc/passwd
for user in $(cut -d: -f1 /etc/passwd); do
  echo "User: $user"
done
```

**Better approach (handles spaces in output):**
```bash
#!/usr/bin/env bash

while IFS=: read -r username rest; do
  echo "User: $username"
done < /etc/passwd
```

### Looping Over Arrays

```bash
#!/usr/bin/env bash

# Define an array
fruits=("apple" "banana" "cherry" "date")

# Loop over array elements
for fruit in "${fruits[@]}"; do
  echo "I like $fruit"
done
```

**Output:**
```
I like apple
I like banana
I like cherry
I like date
```

### What My Professor Didn't Explain: Array Syntax

- `fruits[@]` - Expands to all array elements
- `"${fruits[@]}"` - Expands to all elements, preserving spaces in each element
- Without quotes, elements with spaces would be split

```bash
# Array with spaces
names=("Alice Smith" "Bob Jones")

# WRONG: Breaks "Alice Smith" into two elements
for name in ${names[@]}; do
  echo "Name: $name"
done
# Output:
# Name: Alice
# Name: Smith
# Name: Bob
# Name: Jones

# CORRECT: Preserves spaces
for name in "${names[@]}"; do
  echo "Name: $name"
done
# Output:
# Name: Alice Smith
# Name: Bob Jones
```

## Advanced `hello` Script: Reading from a File

Let's update the `hello` script to read names from a file and greet each person.

### Step 1: Create the Name List File

Create a file called `hello-list`:

```bash
vim hello-list
```

Add these names (or your own):

```
Bruce Wayne
Clark Kent
Kara Danvers
Rachel Roth
Augustus Freeman
Jaime Reyes
Diana Prince
John Jones
Zatanna Zatara
Cassandra Cain
Wesley Dodds
Barry Allen
John Stewart
Arthur Curry
Victor Stone
Dick Grayson
Koriand'r
Damian Wayne
Harleen Quinzel
Slade Wilson
```

**Note:** This is just a data file, not a script. It doesn't need:
- A shebang
- Execute permission

### Step 2: Updated `hello` Script

```bash
#!/usr/bin/env bash

#
# hello - Greet everyone in a file
# Usage: ./hello [filename]
#

# Error handling function
err() {
  local message="$*"
  echo "Error: $message" >&2
  echo "Usage: ${0##*/} [filename]" >&2
  exit 1
}

# Check for exactly one argument
if [[ $# -ne 1 ]]; then
  err "Filename argument is required."
fi

input_file="$1"

# Verify file exists and is readable
if [[ ! -f "$input_file" ]]; then
  err "File '$input_file' does not exist or is not a regular file."
fi

# Load file into array (one element per line)
mapfile -t names < "$input_file"

# Loop through names
for name in "${names[@]}"; do
  # Skip empty lines and whitespace-only lines
  [[ -z "${name// }" ]] && continue
  
  echo "Hello $name"
done
```

### How to Run It

```bash
./hello hello-list
```

**Output:**
```
Hello Bruce Wayne
Hello Clark Kent
Hello Kara Danvers
... (all names)
```

### Understanding `mapfile`

```bash
mapfile -t names < "$input_file"
```

**What it does:**

- Reads the entire file into an array called `names`
- Each line becomes one array element
- `-t` removes trailing newlines from each line
- `< "$input_file"` redirects the file as input

**Alternative (older method):**
```bash
# Read file into array manually
names=()
while IFS= read -r line; do
  names+=("$line")
done < "$input_file"
```

### Understanding the Empty Line Check

```bash
[[ -z "${name// }" ]] && continue
```

**Breaking it down:**

- `${name// }` - Removes all spaces from `$name`
- `-z` - Tests if the result is empty
- `&&` - If true (empty), then...
- `continue` - Skip to next iteration

**Example:**
```bash
name="   "  # Only spaces
[[ -z "${name// }" ]] && echo "Empty"
# Output: Empty
```

## C-Style `for` Loop

Bash also supports a syntax similar to C, Java, and JavaScript.

### Syntax

```bash
for (( INITIALIZATION; CONDITION; INCREMENT )); do
  COMMANDS
done
```

### Example

```bash
#!/usr/bin/env bash

for (( i=0; i<5; i++ )); do
  echo "i is $i"
done
```

**Output:**
```
i is 0
i is 1
i is 2
i is 3
i is 4
```

### How It Works

1. **Initialization:** `i=0` - Sets `i` to 0 before loop starts
2. **Condition:** `i<5` - Loop continues while this is true
3. **Increment:** `i++` - Adds 1 to `i` after each iteration

### Arithmetic Operators in C-Style Loops

| Operator | Meaning | Example |
|----------|---------|---------|
| `++` | Increment by 1 | `i++` |
| `--` | Decrement by 1 | `i--` |
| `+=` | Add to variable | `i+=5` |
| `-=` | Subtract from variable | `i-=2` |
| `*=` | Multiply variable | `i*=2` |
| `/=` | Divide variable | `i/=2` |

### Example: Count by Twos

```bash
#!/usr/bin/env bash

for (( i=0; i<=10; i+=2 )); do
  echo "$i"
done
```

**Output:**
```
0
2
4
6
8
10
```

### Example: Countdown

```bash
#!/usr/bin/env bash

for (( i=10; i>=1; i-- )); do
  echo "$i"
  sleep 1
done
echo "Blast off!"
```

### When to Use C-Style vs. Brace Expansion

**Use C-style when:**
- You need complex increment logic
- You need the counter variable for calculations
- Porting code from other languages

**Use brace expansion when:**
- Simple sequential numbers
- More readable for simple cases

```bash
# Simple: Use brace expansion
for i in {1..10}; do
  echo "$i"
done

# Complex: Use C-style
for (( i=0; i<100; i+=7 )); do
  echo "$i"
done
```

## The `while` Loop

### Purpose

The `while` loop runs **as long as a condition is true**. It keeps checking the condition before each iteration.

### Syntax

```bash
while CONDITION; do
  COMMANDS
done
```

### Example: Simple Counter

```bash
#!/usr/bin/env bash

count=1

while [[ $count -le 5 ]]; do
  echo "Count is $count"
  ((count++))
done
```

**Output:**
```
Count is 1
Count is 2
Count is 3
Count is 4
Count is 5
```

### How It Works

1. Check condition: Is `$count -le 5` (count ≤ 5)?
2. If true, run commands between `do` and `done`
3. After commands finish, go back to step 1
4. If false, exit the loop

### What My Professor Didn't Explain: Arithmetic

The `((count++))` syntax is **arithmetic expansion**:

```bash
# Increment
((count++))     # count = count + 1

# Decrement
((count--))     # count = count - 1

# Addition
((count += 5))  # count = count + 5

# Subtraction
((count -= 2))  # count = count - 2

# Multiplication
((result = 5 * 3))  # result = 15
```

**Important:** No `$` needed inside `(( ))` for variables.

### Example: Reading File Line by Line

One of the most common uses of `while`:

```bash
#!/usr/bin/env bash

while read -r line; do
  echo "Line: $line"
done < input.txt
```

**How it works:**

- `read -r line` - Reads one line from input into variable `line`
- `-r` - Treats backslashes literally (recommended)
- `< input.txt` - Redirects file as input to the loop
- Loop continues until `read` reaches end of file

### Example: Processing CSV Data

```bash
#!/usr/bin/env bash

# File: users.csv
# alice,25,Engineer
# bob,30,Designer
# carol,28,Manager

while IFS=',' read -r name age role; do
  echo "Name: $name"
  echo "Age: $age"
  echo "Role: $role"
  echo "---"
done < users.csv
```

**What's new:**

- `IFS=','` - Sets the Internal Field Separator to comma (splits line on commas)
- `read -r name age role` - Reads three fields into three variables

### Example: Waiting for a Condition

```bash
#!/usr/bin/env bash

echo "Waiting for server to start..."

while [[ ! -f /var/run/server.pid ]]; do
  echo "Still waiting..."
  sleep 2
done

echo "Server is running!"
```

### Example: Interactive Menu

```bash
#!/usr/bin/env bash

while true; do
  echo
  echo "Menu:"
  echo "1. Show date"
  echo "2. Show users"
  echo "3. Exit"
  echo
  
  read -p "Enter choice: " choice
  
  case "$choice" in
    1)
      date
      ;;
    2)
      who
      ;;
    3)
      echo "Goodbye!"
      break  # Exit the loop
      ;;
    *)
      echo "Invalid choice"
      ;;
  esac
done
```

## The `until` Loop

### Purpose

The `until` loop runs **until a condition becomes true** (opposite of `while`).

### Syntax

```bash
until CONDITION; do
  COMMANDS
done
```

### Conceptual Difference

- **`while`:** Run while condition is true (stop when false)
- **`until`:** Run until condition becomes true (stop when true)

### Example

```bash
#!/usr/bin/env bash

count=1

until [[ $count -gt 5 ]]; do
  echo "Count is $count"
  ((count++))
done
```

**Translation:** "Run until count is greater than 5"

This produces the same output as the equivalent `while` loop:

```bash
while [[ $count -le 5 ]]; do
  echo "Count is $count"
  ((count++))
done
```

### When to Use `until`

Use `until` when the logic reads more naturally:

**Better with `until`:**
```bash
# Keep trying until file exists
until [[ -f important.txt ]]; do
  echo "Waiting for important.txt..."
  sleep 1
done
```

**Same thing with `while` (less clear):**
```bash
while [[ ! -f important.txt ]]; do
  echo "Waiting for important.txt..."
  sleep 1
done
```

### Example: Retry Logic

```bash
#!/usr/bin/env bash

attempts=0
max_attempts=5

until ping -c1 google.com &>/dev/null; do
  ((attempts++))
  
  if [[ $attempts -ge $max_attempts ]]; then
    echo "Failed after $max_attempts attempts"
    exit 1
  fi
  
  echo "Attempt $attempts failed. Retrying..."
  sleep 2
done

echo "Connection successful!"
```

## Loop Control: `break` and `continue`

### The `break` Command

**Purpose:** Immediately exit the loop.

```bash
#!/usr/bin/env bash

for num in {1..10}; do
  if [[ $num -eq 5 ]]; then
    break  # Exit loop when num is 5
  fi
  echo "$num"
done

echo "Loop ended"
```

**Output:**
```
1
2
3
4
Loop ended
```

**Use cases:**

- Found what you're looking for (stop searching)
- Error condition (abort processing)
- User requested exit

### The `continue` Command

**Purpose:** Skip the rest of current iteration and move to the next one.

```bash
#!/usr/bin/env bash

for num in {1..5}; do
  if [[ $num -eq 3 ]]; then
    continue  # Skip 3
  fi
  echo "$num"
done
```

**Output:**
```
1
2
4
5
```

**Use cases:**

- Skip invalid/empty data
- Skip files that don't match criteria
- Skip errors and continue processing

### Example: Processing Only `.txt` Files

```bash
#!/usr/bin/env bash

for file in *; do
  # Skip if not a .txt file
  [[ ! "$file" = *.txt ]] && continue
  
  echo "Processing $file"
  # process the file...
done
```

### Example: Skip Empty Lines When Reading File

```bash
#!/usr/bin/env bash

while read -r line; do
  # Skip empty lines
  [[ -z "$line" ]] && continue
  
  echo "Processing: $line"
done < data.txt
```

### Example: Find First Match and Stop

```bash
#!/usr/bin/env bash

for file in *.log; do
  if grep -q "ERROR" "$file"; then
    echo "Found error in $file"
    break  # Stop searching, found what we need
  fi
done
```

## Choosing the Right Loop

### Use `for` when:

- You have a **known list** of items to process
- Processing files matching a pattern
- Iterating a specific number of times

```bash
# Process all .txt files
for file in *.txt; do
  process "$file"
done

# Count to 10
for i in {1..10}; do
  echo "$i"
done
```

### Use `while` when:

- You don't know how many iterations you need
- Reading files line by line
- Waiting for a condition
- Interactive menus

```bash
# Read file line by line
while read -r line; do
  process "$line"
done < file.txt

# Wait for condition
while [[ ! -f ready.flag ]]; do
  sleep 1
done
```

### Use `until` when:

- The exit condition reads more naturally
- Implementing retry logic
- Waiting for something to become available

```bash
# Retry until success
until command_succeeds; do
  echo "Retrying..."
  sleep 1
done
```

### Use C-style `for` when:

- You need complex arithmetic
- Porting code from other languages
- Non-standard increments

```bash
# Complex counting
for (( i=0; i<100; i+=7 )); do
  process "$i"
done
```

## Common Beginner Mistakes

### Mistake 1: Infinite Loops (Forgetting to Update Counter)

```bash
# WRONG: Infinite loop
count=1
while [[ $count -le 5 ]]; do
  echo "$count"
  # Forgot to increment count!
done
```

**Fix:**
```bash
count=1
while [[ $count -le 5 ]]; do
  echo "$count"
  ((count++))  # Don't forget this!
done
```

### Mistake 2: Not Quoting Array Expansion

```bash
names=("Alice Smith" "Bob Jones")

# WRONG: Breaks on spaces
for name in ${names[@]}; do
  echo "$name"
done

# CORRECT
for name in "${names[@]}"; do
  echo "$name"
done
```

### Mistake 3: Using `for` to Read Files (Problematic)

```bash
# WRONG: Doesn't handle spaces in lines
for line in $(cat file.txt); do
  echo "$line"
done

# CORRECT: Use while read
while read -r line; do
  echo "$line"
done < file.txt
```

### Mistake 4: Off-by-One Errors

```bash
# Runs 0-9 (10 times), not 1-10
for (( i=0; i<10; i++ )); do
  echo "$i"
done

# If you want 1-10, use:
for (( i=1; i<=10; i++ )); do
  echo "$i"
done
```

## Practice Exercises

### Exercise 1: Multiplication Table

Create a script that prints a multiplication table:

```bash
#!/usr/bin/env bash

number=$1

for (( i=1; i<=10; i++ )); do
  result=$((number * i))
  echo "$number x $i = $result"
done
```

### Exercise 2: File Backup

Create a script that backs up all `.conf` files:

```bash
#!/usr/bin/env bash

backup_dir="backup_$(date +%Y%m%d)"
mkdir -p "$backup_dir"

for file in *.conf; do
  [[ ! -f "$file" ]] && continue
  
  echo "Backing up $file"
  cp "$file" "$backup_dir/"
done

echo "Backup complete in $backup_dir"
```

### Exercise 3: Count Down Timer

```bash
#!/usr/bin/env bash

seconds=${1:-10}  # Default to 10 if no argument

until [[ $seconds -eq 0 ]]; do
  echo "$seconds seconds remaining"
  sleep 1
  ((seconds--))
done

echo "Time's up!"
```

---

# Summary and Next Steps

## What You've Learned

You now understand the fundamentals of Linux and Bash scripting:

1. **File Permissions** - How Linux controls access to files and directories
2. **The Shebang** - Making scripts executable and self-documenting
3. **Writing Scripts** - Creating your first Bash programs
4. **Variables** - Storing and reusing data in scripts
5. **Conditionals** - Making decisions based on conditions
6. **Loops** - Repeating commands efficiently

## Best Practices Checklist

✅ Always include a shebang in executable scripts  
✅ Make scripts executable with `chmod u+x script_name`  
✅ Quote variables: `"$variable"` not `$variable`  
✅ Use `[[` instead of `[` for conditionals in Bash  
✅ Add error handling (check arguments, file existence)  
✅ Use meaningful variable names  
✅ Add comments to explain non-obvious code  
✅ Test scripts with different inputs  
✅ Use `shellcheck` to catch common errors  

## Further Learning

### Recommended Topics to Explore Next

1. **Functions** - Organize code into reusable blocks
2. **Arrays** - Store and manipulate lists of data
3. **String manipulation** - Advanced text processing
4. **Regular expressions** - Pattern matching and text extraction
5. **Command-line argument parsing** - Professional option handling
6. **Error handling** - `set -e`, `trap`, exit codes
7. **Debugging** - `set -x`, debugging techniques

### Useful Tools

```bash
# Check your scripts for common errors
shellcheck script.sh

# Format/beautify Bash scripts
shfmt -w script.sh

# Bash manual
man bash

# Built-in help
help test
help if
help for
```

### Practice Projects

1. **System Information Reporter** - Display CPU, memory, disk usage
2. **File Organizer** - Sort files into directories by type/date
3. **Backup Script** - Automated backup with rotation
4. **Log Analyzer** - Parse and summarize log files
5. **Development Environment Setup** - Install and configure tools automatically
