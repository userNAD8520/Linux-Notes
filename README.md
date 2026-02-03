# Linux-Notes
Yo wassup. These are my notes for Nathan's Linux Class. I know his notes are not the best so these notes provide you with a better understanding of concepts. I took the material from Nathan's notes on GitLab and enhanced it with more examples and more explainations

## Week 1
**Topics Covered:
- **Kernel vs. Distro: Technically**, Linux is just the kernel (hardware manager). A distro is the full OS package.
- **The Kernel**: Created by Linus Torvalds; sits between the hardware and your apps.
- **Distro Components**: Includes the kernel, Package Manager (installing apps), Shell (CLI), and Init System (booting).
- **Forks**: Most distros are built on others (e.g., Mint is a fork of Ubuntu).
- **Open Source**: Code is public, modifiable, and used everywhere from cars to the Mars Rover.
- **CLI focus**: In this class, we use the text-based Command Line Interface rather than a GUI.
- [Notes](./Notes/W1_Notes_Linux_History.pdf)
--------------------------------------------------------------------
- **SSH Basics**: Securely control servers and move files without passwords.
- **Key Pairs**: Private key (stay on your PC) + Public key (upload to server).
- **Key Gen**: Use ssh-keygen -t ed25519 for the modern, secure standard.
- **SSH Config**: Create nicknames in ~/.ssh/config to avoid typing long IPs.
- **Permissions**: You must chmod 600 your keys or SSH will block the connection.
- **Known Hosts**: Stores server "fingerprints" to prevent Man-in-the-Middle attacks.
- **Debug**: Add -vvv to your command to see exactly why a connection failed.
- [Notes](./Notes/W1_notes_SSH.md)

## Week 2
**Topics Covered:**
- **Bash Shell**: Passes commands to the OS. Use help for built-in info.
- **Processes**: Running programs with unique PIDs. Check status with `ps -ax`.
- **Exit Status (`$?`)**: `0` = Success, anything else = Error.
- **Command Search**: Shell checks Aliases → Functions → Built-ins → $PATH.
- **Navigation**: `pwd` (location), `cd` (move), `ls -al` (list all).
- **File Ops**: `cp` (copy), mv (move/rename), `rm -rf` (delete folder - no undo).
- **Directories**: `mkdir -p` creates nested folders instantly.
- **Manuals**: Use `man <command>` to see the official docs; press `q` to exit.
- [Notes](./Notes/W2_Notes.md)

## Week 3
**Topics Covered:**
Topics Covered:
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


- [Notes](./Notes/W3_Notes.md)

## Week 4

