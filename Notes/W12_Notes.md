# Linux Systems Administration: A Beginner's Study Guide

> **Who this guide is for:** Complete beginners to Linux. No prior experience is assumed. Every concept is explained from the ground up — not just *what* a command does, but *why* it exists and *when* you would actually use it.

---

## Table of Contents

1. [Introduction to Linux Logging](#1-introduction-to-linux-logging)
2. [systemd-journald: The Modern Logging Service](#2-systemd-journald-the-modern-logging-service)
3. [Viewing Logs with `journalctl`](#3-viewing-logs-with-journalctl)
4. [Introduction to systemd Timers](#4-introduction-to-systemd-timers)
5. [Writing systemd Timer Unit Files](#5-writing-systemd-timer-unit-files)
6. [Monotonic vs. Realtime Timers](#6-monotonic-vs-realtime-timers)

---

## 1. Introduction to Linux Logging

### What Is Logging?

**Logging** is the automatic process of recording events as they happen on a Linux system. Every time a service starts, a user logs in, an error occurs, or a scheduled task runs — Linux writes a note about it. These notes are called **log entries**, and their collected history is called a **log**.

Think of it this way: if your Linux system is a car, logs are the dashboard warning lights and the black box recorder combined. When something goes wrong, logs tell you exactly what happened, when it happened, and — often — why.

### Why Logs Are Important

Logs are not just for emergencies. System administrators and developers rely on them constantly for four key reasons:

- **Troubleshooting:** When a service fails to start or a script crashes, you don't have to guess what went wrong. The log will contain specific error messages pointing directly at the problem.
- **Security Auditing:** Logs record every login attempt (successful or failed), every use of `sudo` (the command that grants administrator access), and network connection activity. This makes it possible to detect if someone is trying to break into your system.
- **Performance Monitoring:** Logs can reveal that a process is timing out repeatedly, or that a disk is running out of space before it becomes a crisis.
- **Accountability:** In environments with multiple users, logs create a record of "who did what and when." This is essential for compliance in professional and regulated environments.

### A Brief History: From Text Files to `journald`

Historically, Linux systems used the **`syslog`** protocol for logging. This system stored logs as plain human-readable text files inside the `/var/log/` directory. For example, `/var/log/syslog` and `/var/log/auth.log` were common files you might read with a basic text editor.

> **What is `/var/log/`?** This is a standard directory in the Linux filesystem where log files live. `/var` stands for "variable data" — data that changes over time. The `/log/` subdirectory is specifically for log files.

Modern Linux distributions now primarily use **`systemd-journald`** (often just called "the journal") for logging. This system captures boot messages, kernel output, and the standard output of every service into a **binary (non-text) format**. You'll learn all about this in the next section.

---

## 2. systemd-journald: The Modern Logging Service

### What Is `systemd-journald`?

**`systemd-journald`** (also written as `systemd-journald.service`) is a background service — called a **daemon** — that runs constantly, collecting and storing log data from across your entire Linux system.

> **What is a daemon?** In Linux, a daemon is a program that runs silently in the background without direct user interaction. The name comes from Greek mythology. You can recognize daemons by the `d` at the end of their name — `journald`, `sshd`, `httpd`, etc.

`journald` is part of **systemd**, which is the init system and service manager used by most modern Linux distributions (including Debian, Ubuntu, Fedora, and Arch). Think of systemd as the master manager that starts all other programs when Linux boots.

### Where Does `journald` Collect Data From?

One of `journald`'s most powerful features is that it pulls log data from *multiple sources* automatically, giving you a single unified place to look:

| Source | What It Captures |
|---|---|
| **The Kernel** | Low-level hardware messages (similar to the old `dmesg` command) |
| **initrd** | Messages from the very earliest stage of boot, before the main filesystem is mounted |
| **System Services** | Anything managed by systemd — stdout and stderr from every service |
| **User Applications** | Apps that use standard C library logging calls or the native journal API |

> **What are `stdout` and `stderr`?** When a program runs, it has two "output channels." **stdout** (standard output) is where normal output goes — like the text a command prints when it succeeds. **stderr** (standard error) is where error messages go. Normally these just appear in your terminal. `journald` captures both of them automatically from every service it manages.

A key benefit here: even if a developer wrote a simple script with *no logging code at all*, `journald` will still capture anything the script prints to stdout/stderr. Nothing gets lost.

### Why Does `journald` Use a Binary Format?

Traditional syslog stored logs as plain text. `journald` uses a **structured binary format** instead. This is not to make things harder — it actually provides three significant advantages:

1. **Automatic Metadata:** Every single log entry automatically includes the timestamp, User ID (UID), Group ID (GID), Process ID (PID), and the name of the unit that generated it. With plain text logs, this information was often missing or inconsistent.
2. **Performance:** Binary indexing means that searching through gigabytes of logs is nearly instantaneous. With plain text files, searching was slow and resource-intensive.
3. **Integrity (Forward Secure Sealing):** The binary format supports a security feature that makes it very difficult for an attacker to modify or delete log entries to hide their tracks.

> **The tradeoff:** Because the format is binary, you can't just `cat` a journal file and read it. You *must* use the `journalctl` command (covered in Section 3) to read journal logs.

### Storage: Volatile vs. Persistent Logs

`journald` can store logs in two different ways, and understanding this distinction is important:

| Storage Type | Location on Disk | Behaviour |
|---|---|---|
| **Volatile** | `/run/log/journal/` | Stored in RAM — **lost when you reboot** |
| **Persistent** | `/var/log/journal/` | Written to disk — **survives reboots** |

> **What is RAM?** RAM (Random Access Memory) is your computer's short-term memory. It's fast but temporary — everything in RAM disappears when the power goes off or the system restarts.

**How does Linux decide which one to use?**

By default, `journald` uses **auto** mode: if the directory `/var/log/journal/` exists, it stores logs there (persistent). If that directory doesn't exist, it falls back to volatile storage in RAM. You can explicitly configure this behaviour (see below).

**Practical implication:** If you're troubleshooting a crash that required a reboot and your logs are volatile, you may find the evidence was wiped. Setting up persistent logging is strongly recommended on any system you manage.

### Configuring `journald`

The behaviour of `journald` is controlled by a configuration file.

**Location (Debian/Ubuntu):** `/etc/systemd/journald.conf`

> **What is `/etc/`?** This is the directory that stores system-wide configuration files on Linux. The name comes from an old Unix convention. If you want to change how a system service behaves, you almost always do it by editing a file in `/etc/`.

To view the current configuration:
```bash
cat /etc/systemd/journald.conf
```

You'll see many lines starting with `#`. These are **comments** — they're not active configuration, just documentation showing the default values. To change a setting, you remove the `#` and edit the value.

#### Key Configuration Directives

| Directive | What It Controls | Example |
|---|---|---|
| `Storage=` | Where logs are saved | `Storage=persistent` |
| `Compress=` | Whether to compress logs to save space | `Compress=yes` |
| `SystemMaxUse=` | Maximum total disk space the journal can use | `SystemMaxUse=500M` |
| `ForwardToSyslog=` | Whether to also send logs to a traditional syslog daemon like `rsyslog` | `ForwardToSyslog=no` |

#### Applying Configuration Changes

After editing the config file, the running `journald` service won't automatically pick up changes. You must **restart** the service:

```bash
sudo systemctl restart systemd-journald
```

> **What is `sudo`?** `sudo` (Super User DO) temporarily grants you administrator (root) privileges for a single command. Many system management tasks require this elevated access. You'll use it frequently. The system will ask for your password.

> **What is `systemctl`?** `systemctl` is the command used to control systemd services. `restart` stops the service and starts it again fresh.

---

## 3. Viewing Logs with `journalctl`

### What Is `journalctl`?

**`journalctl`** is the command-line tool you use to read, search, and filter the logs stored by `journald`. Since journal logs are stored in binary format, you can't just open them in a text editor — `journalctl` is your lens into the journal.

> **Most log-viewing commands require `sudo`** because the full system log contains security-sensitive information. If you run `journalctl` without `sudo`, you may only see logs from your own user account.

> **Tip — Forgot `sudo`?**
> If you run a command and get a "Permission denied" error because you forgot `sudo`, you don't have to retype the whole command. Just run:
> ```bash
> sudo !!
> ```
> `!!` is a shell shortcut that means "the previous command." This is a huge time-saver.

### Basic Usage

Running `journalctl` with no arguments shows the entire journal from oldest to newest. This is often too much — the output can be thousands of lines. You'll almost always pair it with a filter.

The output is displayed in a **pager** (similar to the `less` command), which lets you scroll up and down. Use the arrow keys or Page Up/Page Down to navigate. Press `q` to quit.

```bash
journalctl          # Show all logs (opens in pager — press q to exit)
```

#### Filtering by Boot Session

A **boot session** is a single period of system uptime, from power-on to shutdown. `journald` tracks each boot separately, so you can look at logs from a specific run of the system.

```bash
journalctl -b          # Logs from the current boot only
journalctl -b -1       # Logs from the previous boot (before last reboot)
journalctl -b -2       # Logs from two boots ago
journalctl --list-boots  # Show a numbered list of all recorded boots
```

**Why this matters:** If your system crashed and you rebooted, `journalctl -b -1` lets you look at the logs from *before* the crash — even though that session is over.

**Example output of `--list-boots`:**
```
-2 abc123def456 Mon 2025-11-04 09:00:00 UTC—Mon 2025-11-04 17:00:00 UTC
-1 def789abc012 Tue 2025-11-05 08:30:00 UTC—Tue 2025-11-05 23:00:00 UTC
 0 ghi345jkl678 Wed 2025-11-06 09:00:00 UTC—still running
```
The number on the left (`0`, `-1`, `-2`) is what you pass to `journalctl -b`.

#### Kernel Messages

```bash
journalctl -k          # Kernel messages only (equivalent to the old `dmesg` command)
```

Kernel messages are low-level hardware and driver messages. Useful for diagnosing hardware problems.

### Filtering by Time

You can ask `journalctl` to only show logs from a specific window of time. The `--since` and `--until` flags accept human-readable dates and times.

```bash
# Specific date and time range
journalctl --since "2025-11-11 10:00" --until "2025-11-11 12:00"

# Relative time expressions
journalctl --since yesterday
journalctl --since "1 hour ago"
journalctl --since 09:00 --until "1 hour ago"
```

> **Practical use case:** Your monitoring system flagged an anomaly at 2:15 PM. You run `journalctl --since "14:00" --until "14:30"` to see exactly what the system was doing in that window.

### Filtering by Priority (Severity Level)

Not all log messages are equally important. Linux logs use a standardized **priority** (also called severity) system. Lower numbers = more severe.

| Number | Level | Meaning |
|---|---|---|
| 0 | `emerg` | System is unusable |
| 1 | `alert` | Immediate action required |
| 2 | `crit` | Critical conditions |
| 3 | `err` | Error conditions |
| 4 | `warning` | Warning — something may be wrong |
| 5 | `notice` | Normal but significant event |
| 6 | `info` | Informational message |
| 7 | `debug` | Debug-level detail (very verbose) |

The `-p` flag filters by priority. Importantly, it shows messages **at that level and everything more severe** (i.e., lower numbers too).

```bash
journalctl -p err -b      # Show errors (level 3) and above: crit, alert, emerg
journalctl -p warning     # Show warnings and everything more severe
```

> **Real-world tip:** When you're investigating a problem, start with `journalctl -p err -b` to cut through the noise and see only serious issues from the current boot.

### Filtering by Unit, PID, or UID

In systemd, a **unit** is any resource that systemd manages — typically a service. Filtering by unit lets you see only the logs from a specific service.

```bash
# Show logs for the nginx web server from today
journalctl -u nginx --since today

# Show logs for multiple units at once
journalctl -u nginx -u php-fpm --since today

# Filter by Process ID (PID)
journalctl _PID=8880

# Filter by User ID (UID)
journalctl _UID=1001

# List all UIDs that have log entries (also works with _GID)
journalctl -F _UID
```

> **What is a PID?** Every running process on Linux is assigned a unique **Process ID** number. You can find the PID of a running process with the `ps` or `pgrep` commands.

> **What is a UID?** Every user account on Linux has a unique **User ID** number. The root (administrator) account is always UID 0. System services often run as special system users with UIDs below 1000.

**Why filter by UID instead of username?**
`journald` stores the UID directly, so filtering by `_UID` is more precise and reliable than filtering by username, especially for system accounts (like `www-data` for a web server) that may not have a traditional name in every context.

### Output Formatting

By default, `journalctl` shows a human-friendly view. You can change the output format for different purposes:

```bash
journalctl -o json-pretty    # Full JSON output with all fields, nicely indented
journalctl -o verbose        # Show every available metadata field
journalctl --no-pager        # Print output directly to terminal (no scroll interface)
```

**When would you use these?**
- `json-pretty`: When you need to parse log data with a script or send it to another tool.
- `verbose`: When you need to see all metadata (like SELinux context or kernel fields) for deep debugging.
- `--no-pager`: When writing a shell script that processes `journalctl` output — the pager would break automation.

**Example of `json-pretty` output:**
```json
{
  "__REALTIME_TIMESTAMP" : "1731312000000000",
  "_PID" : "1234",
  "_UID" : "0",
  "MESSAGE" : "Started nginx web server.",
  "_SYSTEMD_UNIT" : "nginx.service",
  "PRIORITY" : "6"
}
```

Notice how much richer this is compared to a plain text log line — the PID, UID, unit name, and priority are all automatically included.

### Live Log Monitoring

```bash
journalctl -f       # Follow logs live — new entries appear as they happen
```

This is the `journalctl` equivalent of `tail -f` (a classic command for watching log files). Use it when you're actively debugging — for example, trigger an action and watch the logs update in real time. Press `Ctrl+C` to stop following.

### Combining Filters

All of these filters can be combined for precise results:

```bash
# Show only errors from nginx since yesterday, in JSON format
journalctl -u nginx -p err --since yesterday -o json-pretty

# Follow live logs for two services at once
journalctl -f -u nginx -u mysql
```

---

## 4. Introduction to systemd Timers

### What Are Timers?

A **systemd timer** is a unit file that tells systemd to run another unit (almost always a service) automatically on a schedule. They are the modern Linux replacement for **cron jobs**.

> **What is a cron job?** Cron is the old, traditional Linux scheduling system. You define tasks in a `crontab` file using a compact but cryptic syntax. While cron is still widely used, systemd timers offer significant improvements.

### Why Use Timers Instead of Cron?

Here is a direct comparison:

| Feature | cron | systemd timers |
|---|---|---|
| **Precision** | Minute-level only | Down to seconds and milliseconds |
| **Dependencies** | None — runs blindly | Can wait for network, disk, other services |
| **Logging** | Via email or syslog (messy) | Fully integrated with `journalctl` |
| **Complexity** | One compact text line | Two separate unit files (more setup, more power) |
| **Missed runs** | Silently skipped | Can catch up with `Persistent=true` |

**The logging integration is a major practical win.** With cron, figuring out why a scheduled task failed often meant hunting through email or separate log files. With systemd timers, you just run `journalctl -u your-timer.timer` and see exactly what happened.

### How Timers Work: The Two-File System

A systemd timer always involves **two unit files** working as a pair:

1. **The `.timer` file** — defines the schedule ("run at 3am every day")
2. **The `.service` file** — defines what to actually do ("run this backup script")

By convention, if these files share the same base name, systemd automatically links them. For example:
- `backup.timer` will automatically trigger `backup.service`

You can override this with the `Unit=` directive inside `[Timer]` if you need a timer to start a service with a different name.

---

## 5. Writing systemd Timer Unit Files

### Anatomy of a Unit File

All systemd unit files share a common structure made up of **sections** marked with square brackets. The most relevant sections for timers are:

- **`[Unit]`** — Metadata: description, documentation, and conditions
- **`[Timer]`** — The schedule definition (replaces `[Service]` from service files)
- **`[Install]`** — How and when to enable this unit

### Where to Put Your Unit Files

Unit files live in specific directories. The location determines the priority and purpose:

| Directory | Purpose | Priority |
|---|---|---|
| `/usr/lib/systemd/system/` | Installed by the OS or package manager | Lower (can be overridden) |
| `/etc/systemd/system/` | Created by you, the administrator | **Higher** (takes precedence) |

**Rule of thumb:** Always put your own timer files in `/etc/systemd/system/`. Never edit files in `/usr/lib/systemd/system/` — they may be overwritten by system updates.

### A Real-World Example: The `/tmp` Cleanup Timer

The system ships with a timer that automatically cleans out old files from the `/tmp` directory. Studying it is a great way to understand the format.

> **What is `/tmp`?** This is a special directory for temporary files. Programs put files there that they only need briefly. Without cleanup, it would fill up forever.

**File:** `/usr/lib/systemd/system/systemd-tmpfiles-clean.timer`

```ini
[Unit]
Description=Daily Cleanup of Temporary Directories
Documentation=man:tmpfiles.d(5) man:systemd-tmpfiles(8)
ConditionPathExists=!/etc/initrd-release

[Timer]
OnBootSec=15min
OnUnitActiveSec=1d
```

**Breaking down every line:**

- **`Description=`**: A human-readable label for this timer, shown in `systemctl list-timers`. Always write a clear description.
- **`Documentation=`**: Links to relevant manual pages. Not required, but good practice.
- **`ConditionPathExists=!/etc/initrd-release`**: A safety guard. The `!` means "NOT." This says: "Only run this timer if the file `/etc/initrd-release` does *not* exist." That file is only present during early boot (initrd phase), so this line ensures the timer never runs too early in the boot process.
- **`OnBootSec=15min`**: Wait 15 minutes after the system finishes booting before running the first time. This is a courtesy — you don't want disk-heavy cleanup tasks slowing down your login experience right after boot.
- **`OnUnitActiveSec=1d`**: After each run, wait 24 hours before running again.

### Naming Convention and Automatic Linking

If your timer and service share the same base name, systemd links them automatically — no extra configuration needed:

```
/etc/systemd/system/backup.timer    ← schedule
/etc/systemd/system/backup.service  ← what to run
```

If you need to point a timer at a differently-named service:
```ini
[Timer]
Unit=my-different-service.service
OnCalendar=hourly
```

### Managing Timers with `systemctl`

Timers follow the exact same lifecycle as any other systemd unit:

```bash
# Enable a timer to start automatically at boot
sudo systemctl enable my-timer.timer

# Start a timer immediately (without waiting for a reboot)
sudo systemctl start my-timer.timer

# Enable AND start in one command
sudo systemctl enable --now my-timer.timer

# Check the status of a timer
sudo systemctl status my-timer.timer

# Disable a timer
sudo systemctl disable my-timer.timer

# List all timers and their schedules
sudo systemctl list-timers
```

**Understanding `systemctl list-timers` output:**

```
NEXT                          LEFT          LAST                          PASSED       UNIT                         ACTIVATES
Thu 2025-11-07 03:00:00 UTC   9h left       Wed 2025-11-06 03:00:00 UTC   14h ago      backup.timer                 backup.service
```

| Column | Meaning |
|---|---|
| **NEXT** | The exact date and time of the next scheduled run |
| **LEFT** | How much time remains until the next run |
| **LAST** | When the timer last triggered |
| **PASSED** | How long ago that was |
| **UNIT** | The name of the timer unit |
| **ACTIVATES** | The service unit this timer will trigger |

---

## 6. Monotonic vs. Realtime Timers

There are two fundamentally different ways to define *when* a timer fires. Understanding the difference is essential.

### Monotonic Timers

**Monotonic timers** trigger based on elapsed time since a specific *event* (like boot or last run), not based on a calendar date.

Think of it like a stopwatch: "Run 15 minutes after the system boots" or "Run every 24 hours after the last run."

**Directives used:**

| Directive | Meaning |
|---|---|
| `OnBootSec=15min` | Trigger 15 minutes after boot |
| `OnUnitActiveSec=1h` | Trigger 1 hour after the last time the unit ran |
| `OnActiveSec=30s` | Trigger 30 seconds after this timer itself starts |
| `OnStartupSec=5min` | Trigger 5 minutes after systemd starts |

**Key behaviour: Monotonic timers do not accumulate time while the machine is off.** If you have a timer set for every 5 hours and you power off after 2 hours, the clock effectively resets. The timer will trigger again based on the event (e.g., next boot), not the original schedule.

**Best used for:**
- Cleanup tasks (clear temp files every hour)
- Delayed startup tasks (start a service 2 minutes after boot to let the network come up first)
- Periodic tasks that don't need to happen at a specific clock time

**Example:**
```ini
[Unit]
Description=Run example task weekly, starting shortly after boot

[Timer]
# Don't run immediately on boot — wait 15 minutes to avoid slowing down startup
OnBootSec=15min

# After the first run, wait exactly one week before running again
OnUnitActiveSec=1w

[Install]
# This tells systemd when to activate this timer
# timers.target is the standard target for general-purpose timers
WantedBy=timers.target
```

> **What is `WantedBy=timers.target`?** This line in the `[Install]` section is what makes the timer start automatically when you enable it. `timers.target` is a systemd synchronization point that's reached during normal boot. By declaring `WantedBy=timers.target`, you're saying: "When the system reaches the timers-are-running stage, include me." Without this, `systemctl enable` won't know *when* to activate the timer.

---

### Realtime Timers

**Realtime timers** (also called "calendar timers") trigger at a specific, human-readable date and/or time — like an alarm clock.

They use a single directive: `OnCalendar=`

**Key behaviour with `Persistent=true`:** If the system is off when a realtime timer was supposed to fire, setting `Persistent=true` tells systemd to run the service immediately the next time the machine boots. Without this, the run is simply skipped.

**Best used for:**
- Backups (every night at 2am)
- Weekly reports (every Monday at 6am)
- Any task that must happen at a specific, human-meaningful time

**Example:**
```ini
[Unit]
Description=Run clear-cache.service daily at 3am

[Timer]
OnCalendar=*-*-* 03:00:00

# If the machine was off at 3am, run the task on next boot instead of skipping it
Persistent=true

[Install]
WantedBy=timers.target
```

### The `OnCalendar=` Syntax

The format for `OnCalendar=` expressions is:
```
DayOfWeek Year-Month-Day Hour:Minute:Second
```

A `*` wildcard means "every possible value" for that position.

```
* *-*-* *:*:*
│ │ │ │ │ │ │
│ │ │ │ │ │ └──── Seconds  (0 – 59)
│ │ │ │ │ └────── Minutes  (0 – 59)
│ │ │ │ └──────── Hours    (0 – 23)
│ │ │ └────────── Day      (1 – 31)
│ │ └──────────── Month    (1 – 12)
│ └────────────── Year
└──────────────── Day of Week (Sun – Sat)
```

#### Practical `OnCalendar=` Examples

```ini
# Every day at 4:00 AM
OnCalendar=*-*-* 04:00:00

# Monday through Friday at 5:00 AM (weekday mornings only)
OnCalendar=Mon..Fri 05:00:00

# First Saturday of every month at 6:00 PM
# Logic: it must be a Saturday AND the date must be between 1 and 7 (i.e., the first week)
OnCalendar=Sat *-*-1..7 18:00:00
```

#### Convenient Shorthand Keywords

You can use these words instead of writing out the full date expression:

| Shorthand | Equivalent Expression |
|---|---|
| `hourly` | `*-*-* *:00:00` |
| `daily` | `*-*-* 00:00:00` |
| `weekly` | `Mon *-*-* 00:00:00` |
| `monthly` | `*-*-01 00:00:00` |
| `yearly` / `annually` | `*-01-01 00:00:00` |

You can also append a timezone:
```ini
OnCalendar=daily UTC
OnCalendar=weekly Pacific/Auckland
```

> **Why does timezone matter?** Without specifying a timezone, `OnCalendar` uses the system's local time. If your server is set to UTC but you *mean* midnight in your local timezone, your task will fire at the wrong time. Being explicit prevents surprises.

#### Testing Your Time Expressions

Before deploying a timer, always verify that your `OnCalendar=` expression fires when you intend it to. The `systemd-analyze` tool can calculate the next trigger times for any expression:

```bash
# Test when "*-*-* 01:00:00" will next fire
systemd-analyze calendar "*-*-* 01:00:00"
```

**Example output:**
```
  Original form: *-*-* 01:00:00
Normalized form: *-*-* 01:00:00
    Next elapse: Thu 2025-11-07 01:00:00 UTC
       (in UTC): Thu 2025-11-07 01:00:00 UTC
       From now: 10h left
```

This confirms exactly when the timer will next trigger, which is much better than discovering a misconfigured schedule after a missed backup.

---

## Quick Reference Cheat Sheet

### `journalctl` Common Commands

```bash
journalctl -b                         # Logs from current boot
journalctl -b -1                      # Logs from previous boot
journalctl -p err -b                  # Only errors from current boot
journalctl -u nginx --since today     # nginx logs since today
journalctl --since "2025-11-11 10:00" --until "2025-11-11 11:00"
journalctl -f                         # Follow live
journalctl -o json-pretty             # JSON format
sudo journalctl -F _UID               # List all UIDs with log entries
```

### systemd Timer Directives Reference

| Directive | Type | Example |
|---|---|---|
| `OnBootSec=` | Monotonic | `OnBootSec=5min` |
| `OnUnitActiveSec=` | Monotonic | `OnUnitActiveSec=1h` |
| `OnCalendar=` | Realtime | `OnCalendar=daily` |
| `Persistent=` | Realtime modifier | `Persistent=true` |
| `Unit=` | Both | `Unit=other.service` |

### Minimal Timer Template

```ini
# /etc/systemd/system/my-task.timer
[Unit]
Description=My scheduled task

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/my-task.service
[Unit]
Description=My task service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/my-script.sh
```

```bash
# Enable and start
sudo systemctl enable --now my-task.timer

# Verify
sudo systemctl list-timers
journalctl -u my-task.timer
```

---

## Further Reading

- **Journalctl manual:** `man journalctl`
- **journald configuration:** `man journald.conf`
- **Timer units:** `man systemd.timer`
- **Time syntax reference:** `man systemd.time`
- **journald service:** `man systemd-journald.service`
- [Arch Wiki — systemd/Timers](https://wiki.archlinux.org/title/Systemd/Timers)
- [Digital Ocean — How To Use journalctl](https://www.digitalocean.com/community/tutorials/how-to-use-journalctl-to-view-and-manipulate-systemd-logs)
- [journald.conf Debian Manual](https://manpages.debian.org/testing/systemd/journald.conf.5.en.html)
- [systemd Journal File Format](https://systemd.io/JOURNAL_FILE_FORMAT/)
