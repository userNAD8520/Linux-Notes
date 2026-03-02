# Linux-Notes
Yo wassup. These are my notes for Nathan's Linux Class. I know his notes are not the best so these notes provide you with a better understanding of concepts. I took the material from Nathan's notes on GitLab and enhanced it with more examples and more explainations. I'll be updating this as the weeks progress.

## WEEK 1
**Topics Covered:**
  - **Kernel vs. Distro: Technically**, Linux is just the kernel (hardware manager). A distro is the full OS package.
  - **The Kernel**: Created by Linus Torvalds; sits between the hardware and your apps.
  - **Distro Components**: Includes the kernel, Package Manager (installing apps), Shell (CLI), and Init System (booting).
  - **Forks**: Most distros are built on others (e.g., Mint is a fork of Ubuntu).
  - **Open Source**: Code is public, modifiable, and used everywhere from cars to the Mars Rover.
  - **CLI focus**: In this class, we use the text-based Command Line Interface rather than a GUI.
### [Week 1 Notes](./Notes/W1_Notes_Linux_History.pdf)
  --------------------------------------------------------------------
  - **SSH Basics**: Securely control servers and move files without passwords.
  - **Key Pairs**: Private key (stay on your PC) + Public key (upload to server).
  - **Key Gen**: Use ssh-keygen -t ed25519 for the modern, secure standard.
  - **SSH Config**: Create nicknames in ~/.ssh/config to avoid typing long IPs.
  - **Permissions**: You must chmod 600 your keys or SSH will block the connection.
  - **Known Hosts**: Stores server "fingerprints" to prevent Man-in-the-Middle attacks.
  - **Debug**: Add -vvv to your command to see exactly why a connection failed.
### [Week 1 Notes](./Notes/W1_notes_SSH.md)

## WEEK 2
**Topics Covered:**
  - **Bash Shell**: Passes commands to the OS. Use help for built-in info.
  - **Processes**: Running programs with unique PIDs. Check status with `ps -ax`.
  - **Exit Status (`$?`)**: `0` = Success, anything else = Error.
  - **Command Search**: Shell checks Aliases → Functions → Built-ins → $PATH.
  - **Navigation**: `pwd` (location), `cd` (move), `ls -al` (list all).
  - **File Ops**: `cp` (copy), mv (move/rename), `rm -rf` (delete folder - no undo).
  - **Directories**: `mkdir -p` creates nested folders instantly.
  - **Manuals**: Use `man <command>` to see the official docs; press `q` to exit.
### [Week 2 Notes](./Notes/W2_Notes.md)

## WEEK 3
**Topics Covered:**
  - **Root** (`/`): The start of everything. No "C: Drive" here; everything is a branch of this tree.
  - **Key Directories**:
    - `/etc`: System configuration files (mostly text).
    - `/home`: Where your personal files live (e.g., /home/username).
    - `/usr/bin`: Where the actual programs (like `ls` or `vim`) are stored.
    - `/var/log`: Where the system keeps track of what’s happening (logs).
  - **Pseudo Filesystems** (`/proc`, `/sys`): "Fake" files that live in RAM. They let you see kernel/CPU info by just using cat.
  - **Symlinks**: Shortcuts that point to another path. If you delete the original file, the link breaks.
  - **Pipes** (`|`): Passes the output of one command to the next (e.g., `ls | grep .txt`).
  - **Redirection**: * >: Save output to a file (overwrites).
    -   `>>`: Add output to the end of a file (appends).
    -    `2>`: Save only error messages to a file.
    -    `<`: Feed a file into a command as input.
  - **Vim (The Modes)**:
    - **Normal**: For moving and editing (default). Press `Esc` to get here.
    - **Insert**: For typing text. Press `i` to enter.
    - **Command**: For saving/quitting. Type : then `w` (save) or `q` (quit).
  - **Motions**: Use `h`, `j`, `k`, `l` (left, down, up, right) to stay on the home row.
  - **Composability**: Combine "verbs" and "nouns" (e.g., `dw` = delete word, `dd` = delete line).
 ### [Week 3 Notes](./Notes/W3_Notes.md)

## WEEK 4
**Topics Covered:**
  - **`*` (Asterisk)**: Matches zero or more characters (e.g., `*.csv`).
  - **`?` (Question Mark)**: Matches exactly one character (e.g., `file?.log`).
  - **`[]` (Square Brackets)**: Matches one character from a set (e.g., `[a-z].txt`).
  - **`**` (Globstar)**: Recursive matching through subdirectories (must enable with shopt `-s globstar`).
  - **`cut`**: Extracts columns/fields.
    - `cut -d',' -f2 file.csv` (Extracts 2nd column of a CSV).
  - **`sed` (Stream Editor)**: Search and replace text patterns.
      - `sed 's/old/new/g' file` (Replace "old" with "new" globally).
      - Use `-i` to save changes directly to the file ("in-place").
  - **`awk`**: Advanced data processing language.
      - Best for math on columns or complex reports (e.g., calculating averages).
  - **`grep`**: The "Find" for text patterns.
    - `grep "ERROR" log.txt` (Find all error lines).
### [Week 4 Notes](./Notes/W4_notes.md)

## WEEK 5
**Topics Covered:**
  - **Package Manager**: Automates install/update/removal and handles dependencies.
  - **dpkg**: Low-level, local `.deb` files only; no dependency resolution.
  - **APT**: High-level, talks to repositories and solves dependencies.
  - **Key Commands**:
      - `apt update`: Refresh package lists.
      - `apt upgrade`: Update all installed software.
      - `apt install <pkg>`: Install new software.
      - `apt purge <pkg>`: Remove software + config files.
      - `apt autoremove`: Delete unused dependency "leftovers."
  - `~/.bashrc`: For interactive shells (Aliases, Prompts, Functions).
  - `~/.profile`: For login shells (PATH, Environment Variables).
  - `Aliases`: Text shortcuts. Example: `alias gs='git status'`.
  - `Functions`: Shortcuts that handle arguments. Example: `mkd() { mkdir -p "$1" && cd "$1"; }`.
  - **PATH**: List of directories the shell searches for commands.
      - Add to it: `PATH="$HOME/bin:$PATH"`.
  - **Escape Sequences**: `\u` (user), `\h` (host), `\w` (path), `\$` (`$` or `#`).
  - **Colors**: Uses ANSI codes. Example: `\[\e[31m\]` (Red).
  - **Reset**: Must end with `\[\e[0m\]` to avoid "bleeding" color into commands.
  - **Apply Changes**: Run source `~/.bashrc`.
### [Week 5 Notes](./Notes/W5_Notes.md)

## WEEK 6
**Topics Covered:**
- Permissions & Shebang
  - `#!/usr/bin/env bash`
  - `chmod u+x <file>`
- Variables
  - `var="value"` (No spaces!)
  - `"$var"` (Always use quotes to expand)
  - `$1, $2, ...` (Positional arguments)
  - `$#` (Number of arguments)
- Conditionals
  - `[[ -f $file ]]`: True if file exists.
  - `[[ $status -eq 0 ]]`: True if successful.
- Loops
  - `for item in $list; do ... done`
  - `while read -r line; do ... done < file.txt`
### [Week 6 Notes](./Notes/W6_Notes.md)
 
## WEEK 7
**Topics Covered:**
* **Expansions**
    * `{1..5}` or `{a,b}` (Brace expansion)
    * `"$var"` (Parameter expansion - **always** quote!)
    * `$(command)` (Command substitution)
    * `$(( 1 + 1 ))` (Arithmetic - integers only)

* **Debugging**
    * `set -x` (Print commands as they run)
    * `set -e` (Exit immediately on error)
    * `set -euo pipefail` (Strict "safe mode")
    * `shellcheck script.sh` (Static analysis tool)

* **Parsing Options (getopts)**
    * `while getopts "a:b" opt` (Parse flags loop)
    * `"a:"` (Colons mean the flag needs an argument)
    * `"$OPTARG"` (Holds the flag's argument value)
    * `shift $((OPTIND - 1))` (Removes flags after processing)

* **Heredocs**
    * `cmd << EOF` (Multi-line input block)
    * `<<-` (Strips leading tabs for indentation)
    * `<< 'EOF'` (Literal input - no variable expansion)

* **User Input**
    * `read -p "Prompt" var` (Ask for input inline)
    * `read -s` (Silent input for passwords)
    * `while read -r line` (Safely read file line-by-line)

* **Functions**
    * `name() { ... }` (Function definition)
    * `local var="val"` (Scope variable to function)
    * `$1` (First argument passed *to the function*)
    * `return 1` (Sets exit code only; use `echo` for data)
### [Week 7 Notes](./Notes/W7_Notes.md)

## WEEK 9

Here's a bullet point summary of all the notes:

**Core Concepts**
- Linux is a multi-user OS — every action is performed by a user, and access is controlled through users, groups, permissions, and ownership
- Every file has one owner (user), one group owner, and permission bits for owner / group / others
- Permissions are read (`r`), write (`w`), and execute (`x`) — execute on a directory means you can `cd` into it, not "run" it

**Users**
- Two types: human users (interactive login) and system users (run services, no login)
- Every user has a UID — Linux uses the number internally, not the username
- UID 0 = root, UIDs 1–999 = system accounts, UIDs 1000+ = regular users
- User data is split across `/etc/passwd` (public: username, UID, home dir, shell) and `/etc/shadow` (root-only: encrypted password and expiry info)

**Groups**
- Groups let multiple users share access to files without opening them up to everyone
- Every user has one primary group (used when creating new files) and can have many secondary groups
- Group data is stored in `/etc/group` — field 4 lists secondary members only; primary group is in `/etc/passwd`
- GIDs follow the same ranges as UIDs (0 = root, 1–999 = system, 1000+ = regular)

**User Management Commands**
- `useradd -m -s /bin/bash user` — create a human user with a home directory and bash shell
- `usermod -aG group user` — add a user to a group; **always use `-a`** or you'll overwrite all their groups
- `userdel -r user` — delete user and home directory; irreversible
- `passwd user` — set or change a password (accounts are locked until this is done)
- `id user` — show UID, GID, and all group memberships
- `su - user` — switch to another user with a full login environment
- `sudo command` — run one command as root; preferred over `su` because it's logged and scoped to one command

**Group Management Commands**
- `groupadd developers` — create a group
- `groupmod -n newname oldname` — rename a group
- `groupdel developers` — delete a group (can't delete a user's primary group)
- `gpasswd -a user group` / `gpasswd -d user group` — add or remove a user from a group

**Permissions & Ownership Commands**
- `chmod u+x file` — symbolic mode; who (`u/g/o/a`) + operator (`+/-/=`) + permission (`r/w/x`)
- `chmod 750 file` — octal mode; each digit = sum of r(4)+w(2)+x(1) for owner/group/others
- `chmod -R 750 dir/` — apply recursively
- `chown user:group file` — change owner and group; use `-R` for recursive
- `chgrp group file` — change group only
- `umask 022` — sets default permissions for new files by subtracting from 666 (files) or 777 (dirs); changes are temporary unless saved to `~/.bash_profile`

**How Permission Checking Works**
- Linux checks in order: owner → group → others, and **stops at the first match**
- If you're the owner, only owner permissions apply — even if group permissions are more permissive
- root bypasses all permission checks entirely
### [Week 9 Notes](./Notes/W9_Notes.md)
