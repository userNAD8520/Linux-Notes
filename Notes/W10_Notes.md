
> **Who this is for:** Linux beginners with little or no prior experience.
> This guide rewrites and expands course notes into a clear, structured reference you can study and return to.

---

## Table of Contents

1. [What is an Init System?](#1-what-is-an-init-system)
2. [The Linux Boot Process (Step by Step)](#2-the-linux-boot-process-step-by-step)
3. [What is systemd?](#3-what-is-systemd)
4. [systemd Units and Unit Files](#4-systemd-units-and-unit-files)
5. [Service Units in Depth](#5-service-units-in-depth)
6. [systemd Targets](#6-systemd-targets)
7. [How Dependencies Work in systemd](#7-how-dependencies-work-in-systemd)
8. [Managing Services with `systemctl`](#8-managing-services-with-systemctl)
9. [Quick Reference Table](#9-quick-reference-table)

---

## 1. What is an Init System?

### The Problem it Solves

When you power on a Linux computer, the hardware boots, the kernel loads — but then what? The kernel alone does not know *which programs to start*, *in what order*, or *how to keep them running*. Something needs to coordinate all of that. That coordinator is called the **init system**.

### Definition

An **init system** (short for *initialization system*) is the **first program started by the Linux kernel** after it finishes booting. It becomes the **ancestor of every other process** on the system and stays running for as long as the machine is on.

Its responsibilities include:

- **Starting system services** — programs that run in the background (called *daemons*), such as a web server, an SSH server, or a network manager.
- **Managing service lifecycles** — starting, stopping, and restarting those services.
- **Handling dependencies** — ensuring that if Service B requires Service A, Service A starts first.
- **Defining system states** — deciding what set of services should be active (e.g., graphical desktop vs. text-only server mode).
- **Shutting down cleanly** — stopping services safely when you reboot or power off.

### Why PID 1 Matters

Every running program on Linux is assigned a **Process ID (PID)** — a unique number the kernel uses to track it. The init system always gets **PID 1**, which is special:

- It is the *first* userspace process.
- All other processes are descendants of PID 1.
- If PID 1 crashes or is killed, the entire system goes down.

You can verify this yourself by running:

```bash
ps -p 1
```

Expected output on a systemd-based system:

```
  PID TTY          TIME CMD
    1 ?        00:00:05 systemd
```

Or alternatively:

```bash
cat /proc/1/comm
```

```
systemd
```

> **What my professor didn't explain:** `/proc` is a special virtual filesystem that the Linux kernel provides so you can read information about running processes. It is not a real folder on disk — it exists only in memory. `/proc/1/comm` is a tiny file that just contains the name of whatever process has PID 1.

### Other Init Systems (For Context)

Most modern Linux distributions use **systemd**, but alternatives exist. They tend to be used by distributions that prioritize simplicity or need to run on very low-powered hardware:

| Init System | Notable Users |
|---|---|
| **systemd** | Ubuntu, Fedora, Debian, Arch, RHEL, and most mainstream distros |
| **OpenRC** | Gentoo, Alpine Linux |
| **runit** | Void Linux |
| **dinit** | Artix Linux (optional) |
| **SysV (System V)** | Very old systems; largely replaced |

> **Note on SysV:** The "V" is the Roman numeral for 5 (as in "System 5"), not the letter V. You may still see references to it in documentation about older Linux systems.

---

## 2. The Linux Boot Process (Step by Step)

Understanding *how* Linux boots helps you understand *where* systemd fits in. Here is the full sequence, from the moment you press the power button to the moment you see a login prompt.

### Stage 1 — Firmware (BIOS / UEFI)

**What happens:**
The computer's firmware runs immediately when powered on. It performs a **POST (Power-On Self Test)** to verify that essential hardware (CPU, RAM, storage) is working. Then it selects a boot device (e.g., your SSD or HDD) and hands control over to the bootloader stored there.

- **BIOS** is the older standard, found on machines made before roughly 2012.
- **UEFI** is the modern replacement, faster and with more features.

As a Linux user, you rarely interact with firmware directly — but you might enter the firmware menu (often by pressing `F2`, `F12`, or `DEL` at startup) to change the boot device order.

---

### Stage 2 — Bootloader (GRUB)

**What happens:**
The firmware loads a small program called a **bootloader** from the boot device. The most common Linux bootloader is **GRUB** (GNU Grand Unified Bootloader).

The bootloader:

1. Displays a menu (if configured) so you can select which OS or kernel version to boot.
2. Loads the **Linux kernel** (a file typically named `vmlinuz`) into RAM.
3. Loads the **initramfs** (Initial RAM Filesystem) — a temporary, minimal filesystem used during early boot.
4. Passes **kernel parameters** (configuration options) to the kernel.

> **What my professor didn't explain:** `vmlinuz` is the compressed Linux kernel binary. The `z` at the end stands for *zlib* compression. Think of it as a zip file containing the entire core of the operating system.

---

### Stage 3 — Kernel Initialization

**What happens:**
The kernel unpacks itself from its compressed form, then:

1. Initializes the CPU, memory, and hardware drivers.
2. Mounts the **initramfs** as a temporary root filesystem (`/`).
3. Uses the initramfs to perform early hardware detection (e.g., finding the real disk).
4. Mounts the *real* root filesystem from your disk.
5. Launches **PID 1** — the init system.

> **What my professor didn't explain:** The initramfs is necessary because the kernel might not have the right drivers built in to directly access your storage device. The initramfs contains just enough tools to find and mount the real root filesystem, then hands over control.

---

### Stage 4 — systemd (PID 1) Takes Over

Once the kernel starts systemd:

1. systemd reads its **unit files** (configuration files describing services and system states).
2. It determines the **default target** — the operating mode the system should boot into.
3. It resolves all **dependencies** between services and builds a startup graph.
4. It starts all required services **in parallel where possible** (a key performance advantage over older init systems).

---

### Stage 5 — Reaching the Default Target

The system boots into one of two common states:

| Target | Mode | Typical Use |
|---|---|---|
| `multi-user.target` | Command-line only | Servers, headless systems |
| `graphical.target` | Full desktop GUI | Desktops and laptops |

You can check which target your system defaults to:

```bash
systemctl get-default
```

Example output on a desktop:

```
graphical.target
```

---

### Stage 6 — User Login

After the target is reached:

- If graphical: a **display manager** (such as GDM or SDDM) starts and shows a login screen.
- If text-only: a login prompt appears on the terminal.
- After login, user-specific services start (e.g., your desktop environment, notification daemons, etc.).

---

## 3. What is systemd?

### The Short Answer

**systemd** is the init system used by most modern Linux distributions. It is the program that runs as PID 1 and is responsible for starting and managing the rest of the system.

### The Longer Answer

systemd is actually a large *suite* of tools — not just one program. It includes utilities for logging (`journald`), managing network time (`timesyncd`), hostname configuration, and more. However, when people say "systemd," they almost always mean the **init and service management** component. That is the focus of this guide.

From the official systemd documentation:

> *systemd is a suite of basic building blocks for a Linux system. It provides a system and service manager that runs as PID 1 and starts the rest of the system.*

### Why Was systemd Created?

The older init system (SysV) started services *sequentially* — one at a time, in a fixed order. On modern hardware with SSDs and multi-core CPUs, this was unnecessarily slow. systemd was designed to:

- Start services **in parallel** wherever possible, dramatically speeding up boot times.
- Use **dependency declarations** so services start in the right order automatically.
- **Monitor and restart** services that crash.
- Provide **structured logging** (via `journald`) for all services.
- Manage more than just services — also mounts, devices, timers, sockets, and more.

---

## 4. systemd Units and Unit Files

### What is a Unit?

In systemd, the fundamental managed object is called a **unit**. A unit represents any resource or component that systemd knows how to manage. This could be:

- A background service (e.g., the SSH server)
- A filesystem mount point
- A hardware device
- A scheduled timer
- A group of other units

Every unit is described by a **unit file** — a plain text configuration file that tells systemd everything it needs to know about that unit: how to start it, when to start it, what it depends on, and how to stop it.

Think of unit files as *recipes* that systemd follows to bring services to life.

### Where are Unit Files Stored?

Unit files can be in several places. The two most important are:

| Location | Purpose |
|---|---|
| `/usr/lib/systemd/system/` | **System-provided** unit files. Installed by your package manager (e.g., `apt`, `dnf`). You should not edit these directly. |
| `/etc/systemd/system/` | **Administrator overrides and custom units.** Files here take priority over `/usr/lib/systemd/system/`. This is where you put your own units or customizations. |

> **Why two locations?** This separation protects you from accidentally breaking a system-provided unit file during a package update. If you want to customize how nginx starts, you create a file in `/etc/systemd/system/` rather than editing the original. Your version takes precedence.

You can browse what unit files are installed on your system:

```bash
ls /usr/lib/systemd/system/
```

### Unit Types

Units are categorized by type, shown by their **file extension**:

| Unit Type | Extension | Purpose |
|---|---|---|
| **Service** | `.service` | Manages a daemon or background process |
| **Target** | `.target` | Groups units into logical system states |
| **Mount** | `.mount` | Controls filesystem mounts |
| **Socket** | `.socket` | Enables socket-based service activation |
| **Timer** | `.timer` | Schedules recurring jobs (similar to `cron`) |

> **For this course:** Focus on **service units** (`.service`) and **timer units** (`.timer`). The others are listed for completeness but will not be on the exam.

---

## 5. Service Units in Depth

### What is a Service Unit?

A **service unit** is the most common type of unit. It tells systemd how to run and manage a **daemon** — a background process that provides some functionality (e.g., a web server, SSH server, or database).

### Anatomy of a Service Unit File

Service unit files are divided into **sections**, each marked with a header in square brackets. Within each section are **directives** — key-value pairs that configure behavior.

Here is a complete example of a simple service unit file:

```ini
[Unit]
Description=Example Web Application
After=network.target

[Service]
ExecStart=/usr/local/bin/example-app
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Let's break down each section and directive:

---

#### The `[Unit]` Section — Metadata and Dependencies

This section describes *what* the unit is and *how it relates to other units*.

| Directive | What it Does |
|---|---|
| `Description=` | A human-readable name for the service. Shown in `systemctl status` output. |
| `After=` | Specifies ordering — this service should start *after* the listed units. Does not force those units to start. |
| `Before=` | The opposite of `After=` — this service must start *before* the listed units. |
| `Wants=` | A *soft* dependency — systemd will *try* to start the listed units, but if they fail, this service still starts. |
| `Requires=` | A *hard* dependency — if the listed unit fails to start, this service will not start either. |
| `Documentation=` | Links to man pages or URLs for documentation. |

---

#### The `[Service]` Section — How the Service Runs

This section tells systemd exactly how to run the program.

| Directive | What it Does |
|---|---|
| `ExecStart=` | The full command to run when starting the service. Must be an absolute path (e.g., `/usr/sbin/nginx`, not just `nginx`). |
| `ExecStop=` | Command to run when stopping the service (optional — systemd can send signals automatically). |
| `ExecReload=` | Command to reload the service's configuration without a full restart. |
| `Restart=` | When to automatically restart the service. Common values: `on-failure` (restart only on crash), `always` (always restart). |
| `RestartSec=` | How many seconds to wait before restarting. |
| `Type=` | How the process signals that it is ready. Common values: `simple` (default), `notify`, `forking`. |
| `Environment=` | Set environment variables for the process. |
| `EnvironmentFile=` | Load environment variables from a file. |
| `KillMode=` | How to kill the service on stop. `process` kills only the main process. |

---

#### The `[Install]` Section — Integration with System Startup

This section defines how the unit connects to the boot process. It is only consulted when you run `systemctl enable`.

| Directive | What it Does |
|---|---|
| `WantedBy=` | When this service is *enabled*, it will be pulled in by the specified target at boot. The most common value is `multi-user.target`. |
| `RequiredBy=` | Stronger version of `WantedBy=` — the target *requires* this service. |
| `Alias=` | Alternative names for the unit. |

> **Important:** The `[Install]` section does **nothing** on its own. It only takes effect when you run `systemctl enable`. Enabling creates a symbolic link that causes the target to automatically pull in your service at boot.

---

### Real-World Example: The SSH Service Unit

Here is the actual `sshd.service` file used on Fedora:

```ini
[Unit]
Description=OpenSSH server daemon
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target sshd-keygen.target
Wants=sshd-keygen.target
Wants=ssh-host-keys-migration.service

[Service]
Type=notify
EnvironmentFile=-/etc/sysconfig/sshd
ExecStart=/usr/sbin/sshd -D $OPTIONS
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

**What this means in plain English:**

- Run `/usr/sbin/sshd -D` to start the SSH server.
- Start only *after* the network and SSH key generation are ready.
- If the SSH daemon crashes, wait 42 seconds and then automatically restart it.
- When enabled, this service should be pulled in when the system reaches `multi-user.target`.

> **What my professor didn't explain:** The `-` in `EnvironmentFile=-/etc/sysconfig/sshd` is a special prefix meaning "if this file doesn't exist, don't error out — just skip it." Without the `-`, systemd would refuse to start the service if the file was missing.

---

## 6. systemd Targets

### What is a Target?

A **target** is a special type of systemd unit that represents a **system state** — a specific collection of services and resources that should be active at a given time. Targets are systemd's replacement for the older concept of *runlevels*.

Think of a target as a **milestone** in the system's startup process. When systemd "reaches" a target, it means all the services that belong to that target have been started.

### Common Built-in Targets

| Target | Purpose |
|---|---|
| `default.target` | The target the system boots into by default (usually an alias for `graphical.target` or `multi-user.target`). |
| `multi-user.target` | A fully functional non-graphical system — networking, SSH, and other server services are running. Most servers boot to this. |
| `graphical.target` | Everything in `multi-user.target`, plus a graphical display manager and desktop environment. |
| `network.target` | Indicates the network stack is available. Many services depend on this. |
| `rescue.target` | Single-user mode — minimal services, used for system recovery. Root password required. |
| `emergency.target` | Even more minimal than rescue — mounts only the root filesystem read-only. Used for severe problems. |
| `shutdown.target` | Triggered during system shutdown. |

### How Targets Relate to Each Other

Targets form a hierarchy. For example:

```
graphical.target
    └─ depends on multi-user.target
            └─ depends on network.target
                    └─ depends on basic.target
```

This means if you boot into `graphical.target`, you automatically get everything from `multi-user.target` first — networking, SSH, and all server services come up before the desktop.

---

## 7. How Dependencies Work in systemd

This is one of the most important (and most misunderstood) parts of systemd. There are two completely different kinds of dependencies:

1. **Runtime dependencies** — control ordering and activation *while the system is running*
2. **Installation-time dependencies** — control what happens when you `enable` a service

### Runtime Dependencies (in the `[Unit]` section)

#### `After=` and `Before=` — Ordering

```ini
After=network.target sshd-keygen.target
```

This tells systemd: **"Don't start this service until those units are active."**

Crucially, `After=` does **not** cause those units to start. It only controls *order*. If the listed units aren't starting for some other reason, this service will still start (just after them if they are starting).

Think of `After=` like saying "I want to board the plane *after* security screening" — you're not *causing* security screening to happen, you're just declaring you should come after it.

#### `Wants=` — Soft Dependency

```ini
Wants=sshd-keygen.target
```

`Wants=` says: **"When this service starts, also try to start these units."** If the wanted unit fails to start, this service continues anyway. It is a *best-effort* relationship.

#### `Requires=` — Hard Dependency

`Requires=` says: **"This service cannot function without these units."** If a required unit fails, this service will also fail or be stopped.

| Directive | Effect if dependency fails |
|---|---|
| `Wants=` | Service continues anyway |
| `Requires=` | Service is stopped/fails too |

> **Best practice:** Prefer `Wants=` unless the service genuinely cannot function without the dependency. `Wants=` makes the system more resilient to partial failures.

---

### Installation-Time Dependencies (in the `[Install]` section)

#### `WantedBy=` — The Enable Mechanism

```ini
[Install]
WantedBy=multi-user.target
```

This directive is only used by `systemctl enable`. When you run:

```bash
sudo systemctl enable sshd.service
```

systemd creates a **symbolic link** (a shortcut) in:

```
/etc/systemd/system/multi-user.target.wants/sshd.service
```

This symbolic link effectively adds `sshd.service` to the list of services that `multi-user.target` wants to start. From that point on, every time the system boots into `multi-user.target`, `sshd.service` will be started automatically.

When you run `systemctl disable`, that symbolic link is removed, and the service no longer starts at boot.

> **What my professor didn't explain:** The "enable" mechanism being based on symbolic links is why you can look inside `/etc/systemd/system/*.target.wants/` to see exactly which services are enabled for boot. Try running:
> ```bash
> ls /etc/systemd/system/multi-user.target.wants/
> ```

---

### Putting it All Together — A Full Boot Chain Example

Here's what happens when a server boots to `multi-user.target` with SSH enabled:

```
default.target (alias)
      ↓
multi-user.target
      ↓  (via symlink from systemctl enable)
sshd.service
      ↓  (via Wants=)
sshd-keygen.target    ssh-host-keys-migration.service
```

With ordering applied:

```
network.target  →  sshd-keygen.target  →  sshd.service
```

**The result:** SSH is only started after the network is ready and after SSH host keys exist — exactly the correct order.

---

## 8. Managing Services with `systemctl`

### What is `systemctl`?

**`systemctl`** (system control) is the command-line tool you use to interact with systemd. It is your primary interface for managing services and the state of the system.

Everything from starting a web server to checking why a service crashed goes through `systemctl`.

### A Note About Permissions

Most `systemctl` commands that *change* system state require **root privileges**, meaning you need to prefix them with `sudo`.

Commands that only *read* state (like `status`) can often be run without `sudo`, but some information may be hidden.

### A Note About the `.service` Extension

Because service units are the most common type, systemd lets you omit the `.service` extension. These two commands are identical:

```bash
sudo systemctl start nginx.service
sudo systemctl start nginx
```

For other unit types (like timers), you must include the extension:

```bash
sudo systemctl start backup.timer
```

---

### Core Commands

#### Starting and Stopping Services

**Start a service immediately** (does not persist across reboots):

```bash
sudo systemctl start nginx.service
```

What happens: systemd reads the unit file for nginx, resolves dependencies, and runs the `ExecStart=` command. The service is now running, but only until the next reboot.

---

**Stop a service immediately:**

```bash
sudo systemctl stop nginx.service
```

What happens: systemd sends a stop signal to the running process. If `ExecStop=` is defined in the unit file, that command is run. Otherwise, systemd sends `SIGTERM` (politely asks the process to quit), followed by `SIGKILL` (forcefully terminates) if it doesn't stop in time.

---

**Restart a service** (stop then start):

```bash
sudo systemctl restart nginx.service
```

Useful after making configuration changes that require a full restart.

---

**Reload a service's configuration** (without stopping the process):

```bash
sudo systemctl reload nginx.service
```

Some services can reload their configuration while continuing to run. This causes less downtime than a restart. Not all services support this — it depends on whether `ExecReload=` is defined in the unit file.

> **Gotcha:** If a service doesn't support reload, this command may fail silently or do nothing. When in doubt, use `restart`.

---

#### Enabling and Disabling Services at Boot

**Enable a service** (start automatically at boot, but don't start it right now):

```bash
sudo systemctl enable nginx.service
```

What happens: Creates a symbolic link in the appropriate `.wants/` directory so the service is pulled in during boot. The service does **not** start immediately.

---

**Enable and start a service immediately:**

```bash
sudo systemctl enable --now nginx.service
```

This is the most common command you'll use when setting up a new service — it both starts it immediately and ensures it starts on every future boot.

---

**Disable a service** (remove it from automatic startup):

```bash
sudo systemctl disable nginx.service
```

What happens: Removes the symbolic link created by `enable`. The service **continues running** if it is currently active — disable only affects future boots.

> **Common mistake:** Students often think `disable` stops a running service. It does not. To both stop and disable: `sudo systemctl disable --now nginx.service`

---

#### Checking Service Status

**Check the current status of a service:**

```bash
systemctl status nginx.service
```

Example output:

```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: disabled)
     Active: active (running) since Mon 2025-03-03 14:22:01 UTC; 2h 15min ago
       Docs: man:nginx(8)
   Main PID: 1234 (nginx)
      Tasks: 3 (limit: 4915)
     Memory: 5.3M
        CPU: 234ms
     CGroup: /system.slice/nginx.service
             ├─1234 "nginx: master process /usr/sbin/nginx -g daemon off;"
             └─1235 "nginx: worker process"

Mar 03 14:22:01 server systemd[1]: Started A high performance web server.
```

This output tells you:

| Field | Meaning |
|---|---|
| `Loaded` | Where the unit file is, and whether it's enabled for boot |
| `Active` | Whether the service is currently running and for how long |
| `Main PID` | The process ID of the running service |
| `Memory` | How much RAM the service is using |
| `CGroup` | The control group — shows all processes belonging to this service |
| Log lines at the bottom | The most recent log messages from the service |

> **Why is this useful?** When a service fails to start, `systemctl status` is the first place you look. The log messages at the bottom often tell you exactly what went wrong.

---

**Check if a service is enabled:**

```bash
systemctl is-enabled nginx.service
```

Output will be `enabled`, `disabled`, `static`, or `masked`.

---

#### Listing Units

**List all currently active units:**

```bash
systemctl list-units
```

**List only active service units:**

```bash
systemctl list-units --type=service
```

**List all installed unit files** (including disabled ones):

```bash
systemctl list-unit-files --type=service
```

**List units that have failed:**

```bash
systemctl --failed
```

> **Pro tip:** `systemctl --failed` is one of the first commands to run when something seems wrong with your system. It immediately shows you which services have crashed.

---

#### Masking a Service

**Mask a service** (make it impossible to start, even manually):

```bash
sudo systemctl mask nginx.service
```

Masking creates a symbolic link pointing the unit file to `/dev/null`, completely preventing it from being started by anything — even if another service lists it as a dependency.

**Unmask a service:**

```bash
sudo systemctl unmask nginx.service
```

> **When would you mask a service?** When you want to completely prevent a service from running. For example, if your system has two conflicting services installed and you want to ensure only one can ever run.

> **Difference between `disable` and `mask`:**
> - `disable` — service won't start at boot, but can still be started manually with `systemctl start`
> - `mask` — service cannot be started at all, by anything

---

#### Reloading systemd Itself

After creating or modifying a unit file, you must tell systemd to re-read its configuration:

```bash
sudo systemctl daemon-reload
```

> **Important:** If you create or edit a unit file and then try to start it without running `daemon-reload` first, systemd will use the old (cached) version and your changes will be ignored. Always run `daemon-reload` after editing unit files.

---

### Getting Help

View the manual page associated with a unit:

```bash
systemctl help nginx.service
```

This opens the documentation linked in the unit file's `Documentation=` directive.

---

## 9. Quick Reference Table

| Action | Command | Notes |
|---|---|---|
| **Start** a service now | `sudo systemctl start unit` | Does not persist after reboot |
| **Stop** a service now | `sudo systemctl stop unit` | Does not affect boot config |
| **Restart** a service | `sudo systemctl restart unit` | Full stop and start |
| **Reload** config (no restart) | `sudo systemctl reload unit` | Only works if service supports it |
| **Enable** at boot | `sudo systemctl enable unit` | Does not start it immediately |
| **Enable + start** now | `sudo systemctl enable --now unit` | Most common setup command |
| **Disable** at boot | `sudo systemctl disable unit` | Does not stop it if running |
| **Disable + stop** now | `sudo systemctl disable --now unit` | |
| **Mask** (prevent entirely) | `sudo systemctl mask unit` | Stronger than disable |
| **Unmask** | `sudo systemctl unmask unit` | |
| **Check status** | `systemctl status unit` | Shows logs, PID, uptime |
| **Check if enabled** | `systemctl is-enabled unit` | |
| **List active units** | `systemctl list-units` | |
| **List active services** | `systemctl list-units --type=service` | |
| **List all unit files** | `systemctl list-unit-files` | |
| **List failed units** | `systemctl --failed` | First stop for troubleshooting |
| **Reload systemd config** | `sudo systemctl daemon-reload` | Run after editing unit files |
| **Check default target** | `systemctl get-default` | |
| **Show system status** | `systemctl status` | Overall system health |
| **View help for a unit** | `systemctl help unit` | |

---

## Key Concepts Summary

| Term | Definition |
|---|---|
| **Init system** | The first userspace process (PID 1); coordinates all services |
| **systemd** | The init system used by most modern Linux distributions |
| **Unit** | Any resource systemd manages (service, mount, timer, etc.) |
| **Unit file** | A text configuration file describing a unit |
| **Service unit** | A unit that manages a background process (daemon) |
| **Target** | A unit that groups other units into a system state |
| **`Wants=`** | Soft runtime dependency — try to start, but don't fail if missing |
| **`Requires=`** | Hard runtime dependency — fail if the dependency fails |
| **`After=`** | Ordering directive — start after these units |
| **`WantedBy=`** | Installation directive — which target should pull this service in |
| **`systemctl enable`** | Creates a symlink so a service starts at boot |
| **`systemctl disable`** | Removes the symlink; service no longer starts at boot |
| **`systemctl mask`** | Prevents a service from being started by anything |
| **`daemon-reload`** | Tells systemd to re-read all unit files after changes |
| **PID 1** | The process ID assigned to the init system; the root of all processes |

---

*References:*
- [systemd project site](https://systemd.io)
- [Arch Wiki — systemd](https://wiki.archlinux.org/title/Systemd)
- [Arch Wiki — Boot Process](https://wiki.archlinux.org/title/Arch_boot_process)
- [DigitalOcean — Understanding systemd Units and Unit Files](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files)
