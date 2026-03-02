# Linux Users, Groups & Permissions
### A Beginner-Friendly Study Guide

---

## Table of Contents

1. [Why Linux Has Users, Groups & Permissions](#chapter-1-why-linux-has-users-groups--permissions)
2. [Linux Users](#chapter-2-linux-users)
3. [Linux Groups](#chapter-3-linux-groups)
4. [User Management Commands](#chapter-4-user-management-commands)
5. [Group Management Commands](#chapter-5-group-management-commands)
6. [Permissions & Ownership Commands](#chapter-6-permissions--ownership-commands)
7. [How Linux Checks File Permissions](#chapter-7-how-linux-checks-file-permissions)
8. [Real-World Workflows](#chapter-8-real-world-workflows)
9. [Quick Reference](#quick-reference-commands-at-a-glance)

---

## Chapter 1: Why Linux Has Users, Groups & Permissions

Linux was built from the ground up as a **multi-user operating system**. This means many people — or programs — can use the same machine at the same time. Linux must be able to answer a critical question every time someone tries to do something:

> **"Are you allowed to do this?"**

To answer that question, Linux uses a straightforward system built on four pillars:

- **Users** — Who are you? Every action on Linux is performed by a user.
- **Groups** — What team do you belong to? Users can share access via groups.
- **Permissions** — What can be done to a file? Read, write, or execute.
- **Ownership** — Which user and group "own" a given file?

Together, these four concepts form a security boundary around every file, directory, and resource on the system. Understanding them is essential for managing any Linux machine.

---

### Reading the `ls -l` Output

The fastest way to see this system in action is with the `ls -l` command, which lists files in "long format":

```bash
$ ls -al
-rw-r--r--  1  mh9  devs  9  Apr 12 11:42  test
```

Here is what every column means:

| Column | Meaning |
| :--- | :--- |
| `-rw-r--r--` | File type + permissions (10 characters) |
| `1` | Number of hard links pointing to this file |
| `mh9` | The user (owner) of the file |
| `devs` | The group that owns the file |
| `9` | File size in bytes |
| `Apr 12 11:42` | Date and time of last modification |
| `test` | The filename |

---

### Breaking Down the Permission String

The 10-character string `-rw-r--r--` is split into four parts:

```
- rw- r-- r--
│ │   │   │
│ │   │   └── Others (o): r-- = read only
│ │   └─────── Group  (g): r-- = read only
│ └─────────── Owner  (u): rw- = read + write
└───────────── File type:  - = regular file
```

**Character 1 — File type:**
- `-` = regular file
- `d` = directory
- `l` = symbolic link

**Characters 2–4 — Owner permissions:** What the user who owns the file can do.

**Characters 5–7 — Group permissions:** What members of the owning group can do.

**Characters 8–10 — Other permissions:** What everyone else on the system can do.

---

### The Three Permission Types

| Permission | Symbol | Meaning |
| :--- | :--- | :--- |
| Read | `r` | View file contents, or list directory contents |
| Write | `w` | Modify file contents, or create/delete files in a directory |
| Execute | `x` | Run a file as a program, or enter (`cd` into) a directory |
| None | `-` | That permission is not granted |

> [!WARNING]
> **Execute on a directory is not the same as on a file.** Execute permission on a directory means you can `cd` into it and access its contents. Without execute on a directory, you cannot enter it — even if you have read permission.

---

## Chapter 2: Linux Users

Every action on a Linux system is performed by a user. Even background processes and services run as users — just not human ones. Linux tracks every user by a number called a **UID (User ID)**.

### Two Categories of Users

- **Human users (regular users)** — People who log in and use the system interactively. These are accounts for you, your colleagues, or students.
- **System users (service accounts)** — Accounts created to run specific services or processes (web server, database, etc.) in isolation. They usually cannot log in interactively.

---

### Understanding UIDs (User IDs)

Linux identifies users internally by a **UID number**, not by the username. When you type a username, Linux looks up its UID and uses that number to make all access decisions.

| UID Range | Purpose |
| :--- | :--- |
| `0` | **root** — the superuser. Has unrestricted access to everything. |
| `1 – 999` | System/service accounts. Used by background processes and daemons. |
| `1000+` | Regular human users. Most personal accounts start at UID 1000. |
| `65534` | `nobody` — a special minimal-privilege account used by some services. |

> [!TIP]
> When you create a user with `useradd`, Linux automatically assigns the next available UID in the 1000+ range. You can check a user's UID at any time with the `id` command.

---

### Where User Information Is Stored

Linux stores user information in two files — one readable by everyone, and one readable only by root. This split is a deliberate security decision.

#### The `/etc/passwd` File (Public User Data)

Every user account has an entry in `/etc/passwd`. Despite the name, it does **not** store actual passwords (those live in `/etc/shadow`).

```
mark:x:1001:1001:mark,,,:/home/mark:/bin/bash
[--] - [--] [--] [-----] [--------] [--------]
 1   2   3    4     5         6          7
```

| Field | Example | Meaning |
| :--- | :--- | :--- |
| 1 | `mark` | Username |
| 2 | `x` | Password placeholder — `x` means the real password is in `/etc/shadow` |
| 3 | `1001` | UID |
| 4 | `1001` | Primary GID |
| 5 | `mark,,,` | GECOS — optional full name and contact info. Rarely used today. |
| 6 | `/home/mark` | Home directory |
| 7 | `/bin/bash` | Login shell — the program that starts when the user logs in |

> [!TIP]
> System users that should never log in interactively have `/usr/sbin/nologin` or `/bin/false` as their shell. This prevents anyone from getting an interactive session as that user, even with the correct password.

---

#### The `/etc/shadow` File (Private Password Data)

Because `/etc/passwd` must be readable by all users, encrypted passwords are stored separately in `/etc/shadow`, which **only root can read**.

```
mark:$6$.n.:17736:0:99999:7:::
[--] [----] [---] - [---] -
 1     2      3   4   5   6
```

| Field | Example | Meaning |
| :--- | :--- | :--- |
| 1 | `mark` | Username |
| 2 | `$6$.n.…` | Encrypted password. `$6$` indicates SHA-512 hashing. |
| 3 | `17736` | Days since the Unix epoch (Jan 1, 1970) the password was last changed |
| 4 | `0` | Minimum days before the password can be changed |
| 5 | `99999` | Maximum days before the password must be changed. 99999 = effectively never. |
| 6 | `7` | Days before expiry that the user will be warned |
| 7–9 | (empty) | Inactivity period, account expiration date, and a reserved field |

---

## Chapter 3: Linux Groups

Groups solve a practical problem: how do you give multiple users access to the same resource without opening it up to everyone? The answer is to put those users in a group, and then grant that group access.

**Example:** A team of developers all need to read and write files in a shared project directory. You create a `developers` group, add all the developers to it, and set the directory's group owner to `developers` with write permissions. Done.

---

### Primary vs Secondary Groups

Every user belongs to **exactly one primary group** and may also belong to zero or more **secondary (supplementary) groups**.

#### Primary Group

- Assigned when the user is created.
- Stored in the 4th field of `/etc/passwd`.
- **Used as the default group when the user creates new files.** When you create a file, it is automatically owned by your primary group.
- On modern distributions (Debian, Fedora, Ubuntu), a **User Private Group (UPG)** with the same name as the user is created automatically. So user `naz` gets a primary group also called `naz` with a matching GID.

#### Secondary Groups

- Used to grant additional, optional access to shared resources.
- A user can belong to many secondary groups simultaneously.
- Common examples: `sudo` or `wheel` for admin rights, `docker` to run containers, `www-data` to access web server files.
- Stored in `/etc/group`.

> [!TIP]
> When you add a user to a new group with `usermod -aG`, the change does **not** take effect until the user logs out and logs back in. Their current session still uses the old group list.

---

### The `/etc/group` File

This file stores all group definitions. Each line describes one group:

```
developers:x:1002:alice,bob,charlie
```

| Field | Example | Meaning |
| :--- | :--- | :--- |
| 1 | `developers` | Group name |
| 2 | `x` | Password placeholder (group passwords are almost never used) |
| 3 | `1002` | GID |
| 4 | `alice,bob,charlie` | Comma-separated list of **secondary** members |

> [!WARNING]
> The `/etc/group` file only lists **secondary group members** in field 4. A user's **primary group is NOT listed here** — it's in `/etc/passwd`. This trips up many beginners when checking group memberships.

---

### GID Ranges

| GID Range | Purpose |
| :--- | :--- |
| `0` | `root` group — the superuser's group |
| `1 – 999` | System groups — used by services and background processes |
| `1000+` | Regular groups — created for human users |

---

## Chapter 4: User Management Commands

These commands create, modify, and remove user accounts. They all require root privileges — prepend `sudo` before each command unless you are already logged in as root.

---

### `useradd` — Create a New User

**What it does:** Creates a new user account entry in `/etc/passwd` and optionally creates a home directory, a primary group, and sets a default shell.

**Why it exists:** You need a formal, safe way to add users to the system. Manually editing `/etc/passwd` is error-prone and risky. `useradd` handles all the steps atomically.

**Syntax:**
```bash
useradd [options] username
```

**Examples:**

Create a basic user:
```bash
sudo useradd naz
```

Create a user with a home directory and Bash shell *(recommended for human users)*:
```bash
sudo useradd -m -s /bin/bash naz
```
- `-m` — Creates the home directory at `/home/naz`. Without this flag, the directory may **not** be created.
- `-s /bin/bash` — Sets the login shell to Bash. Without this, the user may get a restricted or unusable shell.

Create a system user (for a service, not a human):
```bash
sudo useradd -r web
```
- `-r` — Creates a system account with a UID in the 1–999 range, no home directory, and no expiry.

> [!WARNING]
> After creating a user with `useradd`, the account has **no password and is locked** by default. Always run `passwd username` immediately after creating a human user account, or they will not be able to log in.

---

### `usermod` — Modify an Existing User

**What it does:** Changes settings on an existing user account — group memberships, username, shell, and more.

**Why it exists:** User needs change over time. You might need to give a user `sudo` access, rename their account, or disable their ability to log in.

**Syntax:**
```bash
usermod [options] username
```

**Examples:**

Add a user to a secondary group *(the most common use case)*:
```bash
sudo usermod -aG developers naz
```
- `-a` — **Append.** Adds to the group without removing existing group memberships.
- `-G developers` — The group to add the user to.

> [!WARNING]
> **CRITICAL: Never use `-G` without `-a`.** Running `sudo usermod -G developers naz` (without `-a`) will **OVERWRITE all of naz's secondary groups**, leaving them in only the `developers` group. This can accidentally remove `sudo` access and cause serious problems.

Rename a user:
```bash
sudo usermod -l naz2 naz
```
- `-l naz2` — New username. Note: this does **not** rename the home directory. You must do that separately.

Disable interactive login (useful for service accounts):
```bash
sudo usermod -s /usr/sbin/nologin naz
```
- `-s /usr/sbin/nologin` — Changes the shell to a program that immediately refuses any login attempt.

---

### `userdel` — Delete a User

**What it does:** Removes a user account from the system.

**Examples:**

Delete the account only (home directory is kept):
```bash
sudo userdel naz
```

Forcefully delete the user and remove their home directory:
```bash
sudo userdel -fr naz
```
- `-f` — Force deletion even if the user is currently logged in.
- `-r` — Removes the user's home directory and mail spool.

> [!WARNING]
> `userdel -fr` **cannot be undone.** Before running it, back up anything important from the user's home directory. Files owned by a deleted user that exist outside their home directory become "orphaned" — they still exist but show a raw UID number instead of a name.

---

### `passwd` — Set or Change a Password

**What it does:** Sets or updates a user's password, storing the encrypted result in `/etc/shadow`.

**Examples:**

Set a password for a specific user (run as root):
```bash
sudo passwd naz
```

Change your own password:
```bash
passwd
```
Without a username, `passwd` changes the password of the currently logged-in user. You will be prompted to enter your current password first.

---

### `id` — Display User Identity

**What it does:** Shows a user's UID, primary GID, and all group memberships. The quickest way to verify who you are or confirm what groups a user belongs to.

**Examples:**

Show your own identity:
```bash
$ id
uid=1001(naz) gid=1001(naz) groups=1001(naz),27(sudo),1002(developers)
```
This tells you: username is `naz`, UID is `1001`, primary group is `naz` (GID 1001), and secondary groups are `sudo` and `developers`.

Show another user's identity:
```bash
id naz
```

Show only the UID:
```bash
id -u naz
```

---

### `su` — Switch User

**What it does:** Starts a new shell session as a different user. Requires that user's password (except when you are already root).

**Examples:**

Switch to another user:
```bash
su naz
```
You are prompted for `naz`'s password. Your current environment (variables, working directory) carries over.

Switch to another user with a **full login environment** *(almost always what you want)*:
```bash
su - naz
```
The `-` starts a full login shell, which loads the target user's environment variables, shell config, and sets the working directory to their home.

Switch to root:
```bash
su -
```

> [!TIP]
> `su -` (with the hyphen) gives a clean, proper session for the target user. `su` (without hyphen) inherits your current environment, which can cause confusing behaviour.

---

### `sudo` — Run a Command as Root

**What it does:** Executes a single command with elevated privileges (usually as root), without fully switching to that user. Every action taken with `sudo` is logged to the system journal.

**Why it's preferred over `su`:** `sudo` is safer and more auditable. It requires your own password, applies only to one command at a time, and leaves a traceable log of every privileged action.

**Examples:**

Run a single command as root:
```bash
sudo useradd naz
```

Run a command as a specific user:
```bash
sudo -u naz ls /home/naz
```

Start an interactive root shell:
```bash
sudo -i
```

#### When to Use `sudo`

Use `sudo` when a command requires administrative (root) privileges, such as:

- Installing or removing software
- Creating or modifying users and groups
- Changing system configuration files in `/etc`
- Modifying files owned by root or a system account
- Starting or stopping system services
- Any change to the system **outside** your home directory

#### When NOT to Use `sudo`

Do **not** use `sudo` for everyday tasks such as:

- Editing files in your own home directory
- Running normal programs
- Viewing files you already have permission to access

> [!WARNING]
> Never use `sudo` for routine tasks. Misusing it can accidentally change file ownership to root, bypass normal permission protections, or cause irreversible system changes. This violates the **Principle of Least Privilege** — always use the minimum permissions necessary for a task.

#### Alternatives to `sudo`

- `doas` — A simpler, more minimal alternative. Common on OpenBSD and some Linux distributions.
- `run0` — A newer tool introduced by systemd that uses session management to run commands as another user.

---

## Chapter 5: Group Management Commands

These commands create, modify, and delete groups, and manage which users belong to them. They all require root privileges.

---

### `groupadd` — Create a New Group

**What it does:** Creates a new group entry in `/etc/group` and assigns it a GID.

**Examples:**

Create a group with an auto-assigned GID:
```bash
sudo groupadd developers
```

Create a group with a specific GID:
```bash
sudo groupadd -g 1050 developers
```
- `-g 1050` — Manually specify the GID. Useful when GIDs must be consistent across multiple servers.

---

### `groupmod` — Modify an Existing Group

**What it does:** Renames a group or changes its GID.

**Examples:**

Rename a group:
```bash
sudo groupmod -n devteam developers
```
- `-n devteam` — New name. `developers` is replaced with `devteam`.

Change a group's GID:
```bash
sudo groupmod -g 1100 devteam
```

> [!WARNING]
> Changing a group's GID does **not** update existing files that used the old GID. Those files will show a raw number instead of a group name until their GID is updated with `chown` or `chgrp`.

---

### `groupdel` — Delete a Group

**What it does:** Removes a group from the system.

```bash
sudo groupdel developers
```

> [!WARNING]
> You **cannot** delete a group that is the primary group of any user. You must first change or delete that user. Files owned by the deleted group become orphaned — they retain the old GID number but it no longer resolves to a name.

---

### `gpasswd` — Manage Group Membership

**What it does:** Adds or removes users from a group. An alternative to `usermod -aG` for managing group membership.

**Examples:**

Add a user to a group:
```bash
sudo gpasswd -a naz developers
```
- `-a` — Add. Appends `naz` to the `developers` group.

Remove a user from a group:
```bash
sudo gpasswd -d naz developers
```
- `-d` — Delete/remove. Removes `naz` from the `developers` group.

> [!TIP]
> Both `gpasswd -a` and `usermod -aG` can add a user to a group. `gpasswd` is slightly simpler when you only need to add or remove group members; `usermod` is more versatile for broader account modifications.

---

## Chapter 6: Permissions & Ownership Commands

Knowing who owns a file is only half the story — you also need to control what each owner can do. These commands let you change permissions and ownership on files and directories.

---

### `chmod` — Change File Permissions

**What it does:** Changes the read/write/execute permissions on a file or directory for the owner (`u`), group (`g`), and others (`o`).

**Why it exists:** Files are created with default permissions, but you often need to restrict or expand access. `chmod` is the tool for making those changes.

**Syntax:**
```bash
chmod [who][operator][permission] filename   # symbolic mode
chmod [octal] filename                        # octal mode
```

---

#### Symbolic Mode

| Part | Options |
| :--- | :--- |
| **Who** | `u` (owner), `g` (group), `o` (others), `a` (all three) |
| **Operator** | `+` (add), `-` (remove), `=` (set exactly) |
| **Permission** | `r` (read), `w` (write), `x` (execute) |

Add execute permission for the owner:
```bash
chmod u+x script.sh
```
This makes `script.sh` runnable by its owner. Essential after writing a new shell script.

Remove write permission from group and others:
```bash
chmod go-w file.txt
```

---

#### Octal (Numeric) Mode

Each permission is assigned a number: **read = 4, write = 2, execute = 1**. Add the values for each category to get its octal digit.

| Octal | Permissions | Breakdown |
| :--- | :--- | :--- |
| `7` | `rwx` | 4+2+1 |
| `6` | `rw-` | 4+2 |
| `5` | `r-x` | 4+1 |
| `4` | `r--` | 4 |
| `0` | `---` | 0 |

Set exact permissions with octal:
```bash
chmod 750 script.sh
```
- `7` → owner: `rwx` (full access)
- `5` → group: `r-x` (read and execute, no write)
- `0` → others: `---` (no access at all)

Apply permissions recursively to a directory and all its contents:
```bash
chmod -R 750 project/
```
- `-R` — Recursive. Changes permissions on the directory **and** everything inside it.

> [!WARNING]
> Be careful with `chmod -R` on mixed-content directories. Recursive changes apply the same permission bits to everything, which may not be appropriate — for example, you might want directories to be executable (enterable) but not data files.

---

### `chown` — Change File Owner (and Group)

**What it does:** Changes which user owns a file. Can also change the group owner at the same time.

**Why it exists:** When files are created, they are automatically owned by the user and group that created them. You may need to transfer ownership after moving files or when setting up a service.

**Examples:**

Change only the user owner:
```bash
sudo chown naz file.txt
```

Change both owner and group:
```bash
sudo chown naz:developers file.txt
```
Syntax is `user:group`.

Recursively change ownership on a directory:
```bash
sudo chown -R naz:developers project/
```
- `-R` — Changes ownership of the directory and every file and subdirectory inside it.

---

### `chgrp` — Change Group Ownership Only

**What it does:** Changes the group that owns a file, without touching the user owner.

Change a file's group:
```bash
sudo chgrp developers file.txt
```

Recursively change group ownership:
```bash
sudo chgrp -R developers project/
```

> [!TIP]
> `chgrp group filename` is essentially a shortcut for `chown :group filename`. Use it when you only want to change the group.

---

### `umask` — Set Default Permissions for New Files

**What it does:** Controls the default permissions applied to newly created files and directories. It defines which permissions are **subtracted** (masked) from the system defaults.

**Why it exists:** When any program creates a new file, Linux starts with a default value (666 for files, 777 for directories) and subtracts the `umask` to determine the final permissions. `umask` is your way of customising that behaviour.

#### How umask Works

```
Final permissions = Starting default − umask
```

- **Files start at 666** (rw-rw-rw-)
- **Directories start at 777** (rwxrwxrwx)

**Example — common default of `umask 022`:**
- Files: `666 - 022 = 644` → `rw-r--r--` (owner reads/writes; group and others read only)
- Directories: `777 - 022 = 755` → `rwxr-xr-x` (owner has full access; group and others can read and enter)

#### umask Reference Table

| umask Digit | File Permissions | Directory Permissions |
| :--- | :--- | :--- |
| `0` | `rw-` | `rwx` |
| `1` | `rw-` | `rw-` |
| `2` | `r--` | `r-x` |
| `3` | `r--` | `r--` |
| `4` | `-w-` | `-wx` |
| `5` | `-w-` | `-w-` |
| `6` | `--x` | `--x` |
| `7` | `---` | `---` |

**Examples:**

View the current umask:
```bash
$ umask
0022
```

Set umask using symbolic notation:
```bash
umask u=rw,g=r,o=
```

Set umask using octal notation:
```bash
umask 026
```
Result: Files are created as `640` (rw-r-----); directories as `751` (rwxr-x--x).

> [!WARNING]
> `umask` changes are **temporary** — they apply only to the current shell session. To make a setting permanent, add the `umask` command to your shell profile file (`~/.bash_profile` or `~/.profile`). Without this, the setting resets after every logout or reboot.

---

## Chapter 7: How Linux Checks File Permissions

When you try to access a file, Linux doesn't simply match your username. It follows a specific **3-step decision process** using UIDs and GIDs:

1. **Are you the owner?** Linux checks if your UID matches the file's owner UID. If yes, it applies the **owner (u)** permission bits and stops checking.
2. **Are you in the group?** If you are not the owner, Linux checks if any of your group memberships (primary or secondary) match the file's group GID. If yes, it applies the **group (g)** permission bits and stops checking.
3. **Everyone else.** If neither match, the **other (o)** permission bits are applied.

> [!WARNING]
> **Important subtlety:** If you ARE the owner of a file, Linux **only** checks owner permissions — even if the group permissions are more permissive. For example, if a file has `--- rw- ---` (no access for owner, read/write for group), and you are the owner, you will be **denied** even if you are also in the group. Linux stops at the first matching category.

---

### A Complete Worked Example

```bash
-rw-r-----  1  alice  developers  1024  Mar 01  shared.txt
```

- Owner (`alice`): `rw-` — can read and write
- Group (`developers`): `r--` — can only read
- Others: `---` — no access at all

| User | Access Result | Reason |
| :--- | :--- | :--- |
| `alice` | Read + Write | She is the owner; owner permissions (`rw-`) apply. |
| `bob` (in developers) | Read only | Group permissions (`r--`) apply. |
| `carol` (not in developers) | No access | Other permissions (`---`) apply. |
| `root` | Full access | root bypasses all permission checks entirely. |

---

## Chapter 8: Real-World Workflows

Knowing the individual commands is important, but in practice you'll use them in combination. Here are the most common end-to-end workflows.

---

### Workflow 1: Setting Up a New Human User

```bash
# 1. Create the user with a home directory and bash shell
sudo useradd -m -s /bin/bash alice

# 2. Set a password immediately (account is locked until this is done)
sudo passwd alice

# 3. Add to relevant groups (e.g., sudo for admin access)
sudo usermod -aG sudo alice

# 4. Verify the result
id alice
```

---

### Workflow 2: Setting Up a Shared Project Directory

```bash
# 1. Create a group for the team
sudo groupadd developers

# 2. Add team members
sudo gpasswd -a alice developers
sudo gpasswd -a bob developers

# 3. Create the shared directory
sudo mkdir /srv/project

# 4. Set group ownership
sudo chown root:developers /srv/project

# 5. Set permissions: owner=rwx, group=rwx, others=---
sudo chmod 770 /srv/project
```

---

### Workflow 3: Deploying a System Service User

```bash
# 1. Create a system user with no login shell
sudo useradd -r -s /usr/sbin/nologin webapp

# 2. Create application directories owned by the service user
sudo mkdir /var/www/myapp
sudo chown -R webapp:webapp /var/www/myapp

# 3. Lock down permissions so only the service user has access
sudo chmod 750 /var/www/myapp
```

---

### Workflow 4: Safely Removing a User Who Has Left

```bash
# 1. Disable the account first (non-destructive and reversible)
sudo usermod -s /usr/sbin/nologin alice

# 2. Back up the home directory before deletion
sudo tar czf /backup/alice-home.tar.gz /home/alice

# 3. Delete the user and home directory when ready
sudo userdel -r alice
```

---

## Quick Reference: Commands at a Glance

### User Management

| Command | What It Does |
| :--- | :--- |
| `sudo useradd -m -s /bin/bash user` | Create human user with home dir + bash |
| `sudo useradd -r user` | Create a system user |
| `sudo usermod -aG group user` | Add user to group (**always use `-a`!**) |
| `sudo usermod -l newname user` | Rename a user |
| `sudo usermod -s /sbin/nologin user` | Disable login for a user |
| `sudo userdel -r user` | Delete user and home directory |
| `sudo passwd user` | Set or change a user's password |
| `id user` | Show UID, GID, and group memberships |
| `su - user` | Switch to user with full login environment |
| `sudo command` | Run a single command as root |

---

### Group Management

| Command | What It Does |
| :--- | :--- |
| `sudo groupadd developers` | Create a new group |
| `sudo groupadd -g 1050 developers` | Create group with specific GID |
| `sudo groupmod -n newname oldname` | Rename a group |
| `sudo groupdel developers` | Delete a group |
| `sudo gpasswd -a user group` | Add user to a group |
| `sudo gpasswd -d user group` | Remove user from a group |

---

### Permissions & Ownership

| Command | What It Does |
| :--- | :--- |
| `chmod u+x file` | Add execute permission for owner |
| `chmod 750 file` | Set exact permissions (`rwxr-x---`) |
| `chmod -R 750 dir/` | Recursive permission change |
| `chown user file` | Change file owner |
| `chown user:group file` | Change owner and group |
| `chown -R user:group dir/` | Recursive ownership change |
| `chgrp group file` | Change group owner only |
| `umask` | Show current default permission mask |
| `umask 022` | Set default permissions (removes write for group/others) |
