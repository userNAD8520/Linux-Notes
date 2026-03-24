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


### [Week 9 Notes](./Notes/W9_Notes.md)






---


## WEEK 10

**Topics Covered:**

- **Init System**: The first process started by the kernel (PID 1). Coordinates all services, manages boot order, and keeps the system running.
- **systemd**: The modern Linux init system. Starts services in parallel, handles dependencies automatically, and runs as PID 1.
- **Units**: The fundamental objects systemd manages. Types include `.service`, `.target`, `.timer`, `.mount`, and `.socket`.
- **Unit Files**: Plain text config files that tell systemd how to run a resource. System files live in `/usr/lib/systemd/system/`, custom overrides go in `/etc/systemd/system/`.
- **Service Units**: The most common unit type. Three sections: `[Unit]` (metadata), `[Service]` (how to run), `[Install]` (boot integration).
- **Dependencies**: `Wants=` is a soft dependency (won't fail if missing), `Requires=` is hard (fails if missing), `After=` controls start order only.
- **Targets**: Logical system states that group units together. Common ones: `multi-user.target` (server/CLI) and `graphical.target` (desktop).
- **Boot Chain**: `default.target` → `multi-user.target` → individual services pulled in via `WantedBy=` symlinks.
- **Starting & Stopping**: Use `systemctl start unit` and `systemctl stop unit` to control services immediately — changes do not persist across reboots.
- **Enable / Disable**: `systemctl enable unit` creates a symlink so the service starts at boot. `systemctl disable unit` removes it. Neither starts nor stops a running service.
- **Enable + Now**: `systemctl enable --now unit` starts the service immediately *and* enables it at boot — the most common setup command.
- **Masking**: `systemctl mask unit` prevents a service from being started by anything. Stronger than disable. Undo with `systemctl unmask unit`.
- **Status**: `systemctl status unit` shows whether a service is running, its PID, memory usage, and recent logs — first stop for troubleshooting.
- **Daemon Reload**: Run `systemctl daemon-reload` after editing any unit file, or systemd will ignore your changes.
- **Exit Status (`$?`)**: After any `systemctl` command, `0` = success, anything else = error.


**How Permission Checking Works**
- Linux checks in order: owner → group → others, and **stops at the first match**
- If you're the owner, only owner permissions apply — even if group permissions are more permissive
- root bypasses all permission checks entirely
### [Week 10 Notes](./Notes/W10_Notes.md)


# Week 12

**Topics Covered:**

- **Linux Logging**: The continuous process of recording OS, service, and application events. Used for troubleshooting errors, security auditing (login attempts, `sudo` usage), performance monitoring, and accountability in multi-user systems.
- **syslog (Legacy)**: The old logging protocol. Stored plain-text files in `/var/log/` (e.g., `/var/log/syslog`). Still present on many systems but largely superseded by `journald`.
- **systemd-journald**: The modern logging daemon (`journald`). Collects logs from the kernel, initrd, system services (stdout/stderr), and user applications into a single centralized journal stored in a structured binary format.
- **Binary Format**: Unlike plain text logs, the journal automatically attaches metadata to every entry — UID, GID, PID, unit name, and timestamp. Enables fast indexed searching and supports Forward Secure Sealing (tamper resistance).
- **Journal Storage**: Two modes — **Volatile** (`/run/log/journal/`, RAM only, lost on reboot) and **Persistent** (`/var/log/journal/`, survives reboots). Default is `auto`: persistent if `/var/log/journal/` exists, volatile otherwise.
- **journald.conf**: Configuration file at `/etc/systemd/journald.conf`. Key directives: `Storage=` (volatile/persistent/auto/none), `Compress=` (default yes), `SystemMaxUse=` (disk space cap), `ForwardToSyslog=` (bridge to legacy syslog). Apply changes with `sudo systemctl restart systemd-journald`.
- **journalctl**: The primary tool for reading the binary journal. Supports filtering by boot session (`-b`), time (`--since`, `--until`), priority (`-p`), unit (`-u`), PID (`_PID=`), and UID (`_UID=`). Output formats include `json-pretty` and `verbose`. Live follow with `-f`.
- **Priority Levels**: Eight severity levels — `0 emerg`, `1 alert`, `2 crit`, `3 err`, `4 warning`, `5 notice`, `6 info`, `7 debug`. Filtering with `-p err` shows that level **and everything more severe**.
- **systemd Timers**: Unit files (`.timer`) that trigger a paired `.service` file on a schedule. The modern replacement for cron jobs. Fully integrated with `journalctl` for logging. More precise (millisecond resolution) and supports dependencies (e.g., wait for network).
- **cron vs. systemd Timers**: cron uses a single compact text line, minute-level precision, no dependency handling, and fragmented logging. Timers require two unit files but offer second-level precision, dependency awareness, `Persistent=` for missed runs, and native journal integration.
- **Timer Unit Files**: Two sections distinguish timers from services — the `.timer` extension and the `[Timer]` section (replaces `[Service]`). If a `.timer` and `.service` share the same base name, systemd links them automatically. Override with `Unit=` in `[Timer]`.
- **Unit File Locations**: `/usr/lib/systemd/system/` for OS/package-installed units (lower priority). `/etc/systemd/system/` for administrator-created units (higher priority — always use this for your own timers).
- **Monotonic Timers**: Trigger after a time span relative to an event. Directives: `OnBootSec=` (after boot), `OnUnitActiveSec=` (after last run). Do **not** accumulate time while the machine is off. Best for cleanup tasks and delayed startup jobs.
- **Realtime Timers**: Trigger at a specific calendar date/time using `OnCalendar=`. Add `Persistent=true` so a missed run (machine was off) fires immediately on next boot. Best for backups, reports, and any human-scheduled task.
- **OnCalendar= Syntax**: Format is `DayOfWeek Year-Month-Day Hour:Minute:Second`. Wildcards (`*`) match any value. Shorthands available: `daily`, `weekly`, `monthly`, `hourly`, `yearly`. Timezone can be appended (e.g., `OnCalendar=daily UTC`).
- **Testing Timers**: Use `systemd-analyze calendar "expression"` to preview exactly when a schedule will next fire before deploying it.
- **Timer systemctl Commands**: `systemctl enable --now my.timer` is the standard setup command (enables at boot + starts immediately). `systemctl list-timers` shows NEXT, LEFT, LAST, PASSED, UNIT, and ACTIVATES for all active timers.
- **WantedBy=timers.target**: Required in `[Install]` for timers. Tells systemd to activate the timer when the system reaches the stage where scheduled tasks are processed. Without it, `systemctl enable` has no effect.
- **Daemon Reload**: Run `sudo systemctl daemon-reload` after creating or editing any unit file. systemd reads unit files at startup — without a reload, it will not see your changes.

---

**How journalctl Filtering Works**

- journalctl always filters by **intersection** — every flag you add narrows results further
- `-p err` means "priority 3 and below (more severe)" — it does **not** mean only exact `err` messages
- `_UID=` and `_PID=` use the journal's stored metadata fields — more reliable than filtering by username
- `--no-pager` disables the interactive scroll view — required when using `journalctl` inside scripts

---

### [Week 12 Notes](./Notes/W12_Notes.md)
