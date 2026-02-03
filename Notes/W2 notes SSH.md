# SSH Getting Started: A Complete Beginner's Guide

---

## What is SSH?

**SSH (Secure Shell)** is a cryptographic network protocol that allows you to securely connect to and control a remote computer over an unsecured network (like the internet).

### Why Does SSH Exist?

Before SSH, administrators used protocols like **Telnet** and **rsh** (remote shell) to connect to remote machines. The problem? These protocols sent everything—including passwords—in **plain text**. Anyone intercepting the network traffic could read your credentials and commands.

SSH was created in 1995 to solve this problem. It encrypts **all** communication between your computer and the remote server, making it safe to:

- Log into remote servers
- Execute commands remotely
- Transfer files securely

### What We'll Use SSH For

In this course, we will use SSH to:

1. **Connect to a remote server** and run commands on that server as if we were sitting in front of it
2. **Transfer files** to and from a remote server (this includes working with Git and remote repositories like GitHub, GitLab, or sourcehut)
3. **Authenticate using SSH keys** instead of passwords—this is both more secure and more convenient

---

## Understanding the SSH Connection Process

When you connect to a remote server via SSH, several steps happen behind the scenes. Understanding this process helps you troubleshoot when things go wrong.

```
+-------------------------+      +---------------------------+
|  SSH Client (Your PC)   |      |  SSH Server (Remote)      |
+-------------------------+      +---------------------------+
         |                                   |
         |   1. TCP Connection               |
         |---------------------------------> | (Port 22 by default)
         |                                   |
         |   2. Protocol Negotiation         |
         |<--------------------------------> | (Agree on encryption methods)
         |                                   |
         |   3. Key Exchange                 |
         |<--------------------------------> | (Create shared session key)
         |                                   |
         |   4. Authentication               |
         |<--------------------------------> | (Password OR Public Key)
         |                                   |
         |   5. Secure SSH Session           |
         |<================================> | (All data is now encrypted)
         |                                   |
```

### Step-by-Step Breakdown

| Step | What Happens | Why It Matters |
|------|--------------|----------------|
| **1. TCP Connection** | Your computer opens a network connection to the server on port 22 | This is the "phone line" that SSH will use |
| **2. Protocol Negotiation** | Both sides agree on which encryption algorithms to use | Ensures compatibility between client and server |
| **3. Key Exchange** | A temporary shared secret (session key) is created | This key encrypts all subsequent communication |
| **4. Authentication** | You prove your identity (password or SSH key) | The server needs to know you're allowed in |
| **5. Secure Session** | You can now run commands, transfer files, etc. | Everything is encrypted end-to-end |

### What My Professor Didn't Explain

- **Port 22** is the default SSH port. Firewalls must allow traffic on this port for SSH to work.
- The **session key** created in step 3 is different from your SSH key pair. It's temporary and only used for this one connection.
- **Public key authentication** (step 4) is what we'll set up—it's more secure than passwords because your private key never leaves your machine.

---

## SSH Key Pairs: The Foundation of Secure Authentication

### What is an SSH Key Pair?

An SSH key pair consists of two mathematically related files:

| Key Type | File Extension | Where It Lives | Who Can See It |
|----------|---------------|----------------|----------------|
| **Private Key** | None (e.g., `ocean`) | Your local machine only | **Only you—NEVER share this** |
| **Public Key** | `.pub` (e.g., `ocean.pub`) | Copied to remote servers | Anyone (it's safe to share) |

### How Key-Based Authentication Works

1. You generate a key pair on your local machine
2. You copy your **public key** to the remote server
3. When you connect, the server challenges you with a puzzle only your **private key** can solve
4. Your SSH client solves the puzzle without ever sending the private key
5. The server grants access

**Analogy:** Think of it like a padlock (public key) and its key (private key). You can give padlocks to anyone, but only you have the key that opens them.

### Why Use Keys Instead of Passwords?

| Passwords | SSH Keys |
|-----------|----------|
| Can be guessed or brute-forced | Virtually impossible to crack |
| Must be typed every time | Can be used automatically |
| Sent to the server (encrypted, but still) | Private key never leaves your machine |
| Same password often reused | Each key pair is unique |

---

## Commands

### `ssh-keygen` — Generate SSH Key Pairs

#### Purpose

The `ssh-keygen` command creates new SSH key pairs. You'll typically create separate key pairs for different purposes:

- One for your Linux VMs (like DigitalOcean droplets)
- One for code hosting services (like sourcehut, GitHub, or GitLab)

#### When Would You Use This?

- Setting up a new computer that needs SSH access
- Creating a dedicated key for a new server or service
- Rotating keys for security purposes

#### Syntax

```bash
ssh-keygen -t <type> -f <filepath> -C "<comment>"
```

#### Options Explained

| Option | Meaning | Example |
|--------|---------|---------|
| `-t` | **Type** of key algorithm | `-t ed25519` (recommended, modern and secure) |
| `-f` | **File** path and name for the key | `-f ~/.ssh/ocean` |
| `-C` | **Comment** to identify the key | `-C "work laptop"` |

#### Practical Example

```bash
ssh-keygen -t ed25519 -f ~/.ssh/ocean -C "my digitalocean droplet key"
```

**What happens when you run this:**

1. SSH asks if you want to set a **passphrase** (optional extra security)
2. Two files are created:
   - `~/.ssh/ocean` — your **private key** (keep this secret!)
   - `~/.ssh/ocean.pub` — your **public key** (copy this to servers)

**Sample output:**

```
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/user/.ssh/ocean
Your public key has been saved in /home/user/.ssh/ocean.pub
The key fingerprint is:
SHA256:abc123... my digitalocean droplet key
```

#### What My Professor Didn't Explain

- **`~/.ssh/`** is a hidden directory in your home folder. The `~` means "home directory" and the `.` prefix makes it hidden.
- **ed25519** is an elliptic curve algorithm—it's faster and more secure than older RSA keys, with much shorter key lengths.
- The **passphrase** encrypts your private key file. If someone steals the file, they still can't use it without the passphrase. However, you'll need to type it each time you use the key (unless you use an SSH agent).

#### ⚠️ Warnings and Best Practices

- **NEVER share your private key** (the file without `.pub`)
- **DO share your public key** (the `.pub` file)—that's the whole point
- Use **descriptive comments** so you remember what each key is for
- Consider using a **passphrase** for keys that access sensitive systems

---

### `ssh` — Connect to Remote Servers

#### Purpose

The `ssh` command establishes a secure connection to a remote server, giving you a command-line shell on that machine.

#### When Would You Use This?

- Managing a web server
- Deploying code to production
- Accessing a cloud VM (like a DigitalOcean droplet)
- Running commands on a remote machine

#### Syntax

```bash
ssh -i <identity_file> <user>@<host>
```

#### Components Explained

| Component | Meaning | Example |
|-----------|---------|---------|
| `-i` | **Identity file** (your private key) | `-i ~/.ssh/ocean` |
| `user` | Username on the **remote** server | `root`, `bob`, `ubuntu` |
| `host` | Server address (IP or domain name) | `192.168.1.100`, `example.com` |

#### Practical Example

```bash
ssh -i ~/.ssh/ocean root@143.198.123.45
```

**What this does:**

1. Uses the private key at `~/.ssh/ocean` to authenticate
2. Connects as the user `root`
3. To the server at IP address `143.198.123.45`

**After connecting, you'll see something like:**

```
Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-52-generic x86_64)

Last login: Mon Jan 15 10:30:22 2024 from 98.76.54.32
root@my-server:~# 
```

You're now running commands **on the remote server**, not your local machine!

#### What My Professor Didn't Explain

- The **`@`** symbol separates the username from the hostname—this is standard Unix convention
- If you don't specify `-i`, SSH will try keys in `~/.ssh/` automatically (like `id_ed25519`, `id_rsa`)
- The **first time** you connect to a server, SSH will ask you to verify the server's fingerprint—this prevents man-in-the-middle attacks
- To **exit** an SSH session, type `exit` or press `Ctrl+D`

---

## Important Files in `~/.ssh/`

Your `~/.ssh/` directory contains several important files. Understanding them helps you manage multiple servers efficiently.

### Overview of `~/.ssh/` Contents

```
~/.ssh/
├── config          # Your SSH shortcuts and settings
├── known_hosts     # Fingerprints of servers you've connected to
├── ocean           # Private key (example)
├── ocean.pub       # Public key (example)
├── hut             # Another private key (example)
└── hut.pub         # Another public key (example)
```

---

### `~/.ssh/config` — Your SSH Shortcut File

#### Purpose

The config file lets you create **aliases** for SSH connections. Instead of typing long commands with multiple options, you define them once and use a short name.

#### Why This File Exists

Without a config file, connecting to a server requires remembering:
- The IP address or hostname
- Which username to use
- Which key file to use
- Any special options

The config file stores all of this so you can simply type `ssh myserver`.

#### Example Config File

```
# DigitalOcean Droplet
Host alpine
  HostName 143.198.123.45
  User bob
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/ocean
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null

# sourcehut (code hosting)
Host *sr.ht
  IdentityFile ~/.ssh/hut
  PreferredAuthentications publickey
```

#### Directives Explained

| Directive | Purpose | Example Value |
|-----------|---------|---------------|
| `Host` | **Alias name** you'll type after `ssh` | `alpine`, `myserver`, `*sr.ht` |
| `HostName` | Actual IP address or domain | `143.198.123.45` |
| `User` | Username to log in as | `bob`, `root`, `ubuntu` |
| `IdentityFile` | Path to your private key | `~/.ssh/ocean` |
| `PreferredAuthentications` | How to authenticate | `publickey` (use keys, not password) |
| `StrictHostKeyChecking` | Verify server fingerprint? | `no` (skip the prompt) |
| `UserKnownHostsFile` | Where to store fingerprints | `/dev/null` (don't store them) |

#### Using the Config File

**With config file:**
```bash
ssh alpine
```

**Without config file (equivalent command):**
```bash
ssh -i ~/.ssh/ocean -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null bob@143.198.123.45
```

The config file saves you from typing all of that every time!

#### Understanding Wildcards

The `*` in `Host *sr.ht` is a **wildcard**. It matches any hostname ending in `sr.ht`, such as:
- `git.sr.ht`
- `lists.sr.ht`
- `builds.sr.ht`

This is useful for services with multiple subdomains.

#### What My Professor Didn't Explain

- **`StrictHostKeyChecking no`** and **`UserKnownHostsFile /dev/null`** are useful when you frequently create and destroy servers (like in cloud environments). Each new server has a different fingerprint, and these options prevent SSH from complaining.
- **⚠️ Warning:** These options reduce security! Only use them for disposable development servers, not production systems.
- The config file is read **top to bottom**. Put specific hosts before wildcards.
- **`/dev/null`** is a special Linux "black hole" file—anything written to it disappears. Using it as `UserKnownHostsFile` means fingerprints aren't saved.

---

### `~/.ssh/known_hosts` — Server Fingerprint Database

#### Purpose

This file stores the **public key fingerprints** of every server you've connected to. SSH uses it to verify you're connecting to the real server, not an imposter.

#### Why This File Exists

Imagine someone intercepts your connection and pretends to be your server (a "man-in-the-middle" attack). They could steal your data or credentials. The `known_hosts` file prevents this by remembering what each server's fingerprint should be.

#### Example Entry

```
example.com,192.168.1.10 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBa...
```

This line says: "The server at `example.com` (also known as `192.168.1.10`) has this specific public key."

#### What Happens When You Connect

**First connection to a new server:**
```
The authenticity of host 'example.com (192.168.1.10)' can't be established.
ED25519 key fingerprint is SHA256:abc123...
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'example.com' (ED25519) to the list of known hosts.
```

**Subsequent connections:** SSH silently verifies the fingerprint matches.

**If the fingerprint changes (potential attack!):**
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
```

#### What My Professor Didn't Explain

- A **legitimate** reason for fingerprint changes: the server was reinstalled or its SSH keys were regenerated
- To fix the warning (after verifying it's safe), remove the old entry:
  ```bash
  ssh-keygen -R example.com
  ```
- This is why `StrictHostKeyChecking no` is risky—it skips this security check entirely

---

## SSH Troubleshooting Guide

When SSH connections fail, the error messages can be cryptic. Here's how to diagnose and fix common problems.

### 🔴 Error: "Permission denied (publickey)"

This is the most common SSH error. The server is rejecting your authentication.

#### Diagnostic Checklist

| Check | How to Verify | Fix |
|-------|---------------|-----|
| **Wrong private key** | Is `-i` pointing to the correct key? | Update your command or config file |
| **Public key not on server** | Is your `.pub` key in the server's `~/.ssh/authorized_keys`? | Add it using `ssh-copy-id` or manually |
| **Wrong permissions** | Are your key files too open? | See permission fixes below |
| **Wrong username** | Are you using the correct `user@`? | Check with server admin |

#### Fixing Permissions

SSH refuses to use keys with incorrect permissions (this is a security feature).

**On your local machine:**
```bash
# Fix private key permissions (read/write for owner only)
chmod 600 ~/.ssh/ocean

# Fix .ssh directory permissions (read/write/execute for owner only)
chmod 700 ~/.ssh/
```

**On the remote server:**
```bash
chmod 700 ~/.ssh/
chmod 600 ~/.ssh/authorized_keys
```

#### What the Permission Numbers Mean

| Number | Meaning | Who Can Access |
|--------|---------|----------------|
| `700` | rwx------ | Only the owner can read, write, and enter the directory |
| `600` | rw------- | Only the owner can read and write the file |

---

### 🔴 Error: "Connection timed out"

Your computer couldn't reach the server at all.

#### Diagnostic Checklist

| Check | How to Verify | Fix |
|-------|---------------|-----|
| **Wrong IP/hostname** | Can you ping the server? | Verify the address |
| **Server is down** | Check your cloud provider's dashboard | Start/restart the server |
| **Firewall blocking port 22** | Check server firewall rules | Allow SSH traffic |
| **Network issues** | Are you on a restricted network? | Try a different network |

**Test connectivity:**
```bash
ping 143.198.123.45
```

---

### 🔧 General Troubleshooting: Verbose Mode

When you can't figure out what's wrong, **verbose mode** shows you exactly what SSH is doing.

```bash
# One -v for basic info
ssh -v user@host

# Two -vv for more detail
ssh -vv user@host

# Three -vvv for maximum detail
ssh -vvv user@host
```

**Example verbose output (showing key being tried):**
```
debug1: Authentications that can continue: publickey
debug1: Next authentication method: publickey
debug1: Offering public key: /home/user/.ssh/ocean ED25519
debug1: Server accepts key: /home/user/.ssh/ocean ED25519
debug1: Authentication succeeded (publickey).
```

---

## Practical Walkthrough: Setting Up SSH for Your Droplet

Let's put everything together with a step-by-step guide.

### Step 1: Generate Your SSH Key Pair

**Linux and macOS:**
```bash
ssh-keygen -t ed25519 -f ~/.ssh/ocean -C "my droplet key"
```

**Windows (PowerShell):**

First, create the `.ssh` directory if it doesn't exist:
```powershell
mkdir .ssh
```

Then generate the key (note: use full path, not `~`):
```powershell
ssh-keygen -t ed25519 -f C:\Users\YourUsername\.ssh\ocean -C "my droplet key"
```

> **Windows Note:** Tilde (`~`) expansion doesn't always work in PowerShell. Use the full path instead.

---

### Step 2: Copy Your Public Key

You need to copy the contents of your **public key** (the `.pub` file) to add it to your server or cloud provider.

**macOS:**
```bash
pbcopy < ~/.ssh/ocean.pub
```

**Linux (with Wayland and wl-clipboard):**
```bash
wl-copy < ~/.ssh/ocean.pub
```

**Linux (with X11):**
```bash
xclip -selection clipboard < ~/.ssh/ocean.pub
```

**Windows (PowerShell):**
```powershell
Get-Content C:\Users\YourUsername\.ssh\ocean.pub | Set-Clipboard
```

**Or just display it and copy manually:**
```bash
cat ~/.ssh/ocean.pub
```

---

### Step 3: Add the Public Key to Your Server

When creating a DigitalOcean droplet, paste your public key in the "SSH Keys" section.

For existing servers, add your public key to `~/.ssh/authorized_keys` on the server.

---

### Step 4: Connect to Your Server

**Direct connection:**
```bash
ssh -i ~/.ssh/ocean root@YOUR_DROPLET_IP
```

---

### Step 5: Create an SSH Config Entry (Recommended)

Edit (or create) `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

Add this configuration:

```
# DigitalOcean Droplet
Host debian
  HostName YOUR_DROPLET_IP
  User root
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/ocean
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null

# sourcehut
Host *sr.ht
  IdentityFile ~/.ssh/hut
  PreferredAuthentications publickey
```

Replace `YOUR_DROPLET_IP` with your actual IP address.

---

### Step 6: Connect Using Your Alias

Now you can simply type:

```bash
ssh debian
```

And you're in! 🎉

---

## Quick Reference Card

### Generate a New Key
```bash
ssh-keygen -t ed25519 -f ~/.ssh/keyname -C "description"
```

### Connect to a Server
```bash
ssh -i ~/.ssh/keyname user@hostname
```

### Fix Key Permissions
```bash
chmod 700 ~/.ssh/
chmod 600 ~/.ssh/keyname
```

### View Public Key
```bash
cat ~/.ssh/keyname.pub
```

### Debug Connection Issues
```bash
ssh -vvv user@hostname
```

### Remove Old Host Fingerprint
```bash
ssh-keygen -R hostname
```

