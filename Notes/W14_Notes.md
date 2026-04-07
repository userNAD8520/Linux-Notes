# Linux Study Guide: Cloud Storage, Network File Systems, and Cloud-Init

> **Course Context:** These notes cover cloud storage options, Network File Storage (NFS), and cloud-init — the tools and concepts you need to provision and manage servers in the cloud. The examples use DigitalOcean, but every concept applies to AWS, Azure, Google Cloud, and other providers.

---

## Table of Contents

1. [Cloud Storage — The Big Picture](#1-cloud-storage--the-big-picture)
2. [Object Storage (DigitalOcean Spaces)](#2-object-storage-digitalocean-spaces)
3. [Network File Storage (NFS)](#3-network-file-storage-nfs)
4. [Block Storage (Volumes)](#4-block-storage-volumes)
5. [Connecting to Network File Storage — Hands-On](#5-connecting-to-network-file-storage--hands-on)
6. [Mounting Filesystems in Linux](#6-mounting-filesystems-in-linux)
7. [Persistent Mounts with `/etc/fstab`](#7-persistent-mounts-with-etcfstab)
8. [Persistent Mounts with systemd `.mount` Unit Files](#8-persistent-mounts-with-systemd-mount-unit-files)
9. [Cloud-Init — Automated Server Setup](#9-cloud-init--automated-server-setup)
10. [YAML — The Configuration Language](#10-yaml--the-configuration-language)
11. [Writing a Cloud-Init Configuration File](#11-writing-a-cloud-init-configuration-file)
12. [Cloud-Init on DigitalOcean](#12-cloud-init-on-digitalocean)
13. [Key Takeaways and Study Tips](#13-key-takeaways-and-study-tips)

---

## 1. Cloud Storage — The Big Picture

### Why Does Cloud Storage Matter?

When you run a server in the cloud, you are renting a **virtual machine (VM)** — a computer that exists as software on someone else's hardware. That VM needs a place to store data: application code, databases, user uploads, logs, backups, and more.

Cloud providers like DigitalOcean, AWS, and Azure all offer **multiple types of storage**, each designed for different use cases. Choosing the right one is a fundamental skill for any systems administrator or DevOps engineer.

### The Three Main Categories

Think of cloud storage as falling into three buckets:

| Storage Type | Analogy | Access Pattern | Shared? | Speed |
|---|---|---|---|---|
| **Object Storage** | A filing cabinet with labeled folders | Via API (HTTP requests) | Yes (via URLs) | Moderate |
| **Network File Storage** | A shared network drive at the office | Mounted like a local folder | Yes (multiple VMs) | Moderate |
| **Block Storage** | An external hard drive plugged into your PC | Mounted like a local disk | No (single VM) | Fast |

### What My Professor Didn't Explain

These three types exist because **no single storage solution is optimal for every workload.** A database needs fast, low-latency disk access (block storage). A team of web servers all need to read the same uploaded images (network file storage). A backup of your entire database needs to be stored cheaply for years (object storage).

Understanding *when* to use each type — not just *what* they are — is the real skill.

> **Key Principle:** Different cloud providers use different product names, but the underlying concepts and use cases are identical. Learn the concept once, and it transfers everywhere.

---

## 2. Object Storage (DigitalOcean Spaces)

### What Is It?

**Object Storage** is a type of cloud storage designed for storing large, unstructured files — things like images, videos, PDFs, backups, and static website assets. DigitalOcean calls their object storage product **Spaces**. On AWS, the equivalent is **S3 (Simple Storage Service)**.

### Why Does It Exist?

Traditional filesystems (like the one on your laptop) organize data into directories and files with permissions, ownership, and a hierarchy. This works great when you need to *edit* files frequently. But when you just need to *store and retrieve* large files — especially millions of them — a traditional filesystem becomes slow and expensive to scale.

Object storage solves this by treating each file as an **object**: a blob of data with a unique identifier and some metadata (like its name, size, and creation date). You don't browse it with `ls` and `cd` — you access objects through an **API** (a set of HTTP endpoints).

### How Does It Work?

- You upload a file (an "object") to a "bucket" (a named container).
- Each object gets a unique URL.
- You retrieve it by making an HTTP request (like a web browser loading a page).
- The storage system handles replication, durability, and scaling behind the scenes.

### When To Use Object Storage

- **Backups** — Large compressed archives (`.tar.gz`, `.sql.gz`)
- **Static assets** — CSS, JavaScript, images for websites
- **Log storage** — Archiving logs that you rarely need to read
- **CDN-backed content delivery** — Serving files to users worldwide through a Content Delivery Network
- **User uploads** — Profile pictures, documents, media files
- **Static website hosting** — Hosting an entire static site directly from a bucket

### When NOT To Use Object Storage

- When your application needs to read and write files as if they were on a local disk (use block storage)
- When multiple servers need to share a traditional filesystem (use network file storage)
- When you need fast, low-latency disk I/O for a database (use block storage)

### What My Professor Didn't Explain

Object storage uses the **Amazon S3 API** — and this is worth understanding. Amazon invented the S3 API, and it became so dominant that nearly every other cloud provider (DigitalOcean, Google Cloud, MinIO, Wasabi) built their object storage to be **S3-compatible**. This means tools like `aws s3 cp`, `s3cmd`, or libraries like `boto3` (Python) work across providers with just a configuration change.

**Reference:**
- [DigitalOcean Spaces Product Page](https://www.digitalocean.com/products/spaces)

---

## 3. Network File Storage (NFS)

### What Is It?

**Network File Storage** provides a shared filesystem that multiple virtual machines can access simultaneously over a network. It uses the **NFS protocol** (Network File System), a standard that has been part of Unix/Linux systems since the 1980s.

When you "mount" an NFS share on your server, it appears as a regular directory — you can `cd` into it, `ls` its contents, create files, set permissions — just like any local folder. But behind the scenes, every read and write travels over the network to a remote storage server.

### Why Does It Exist?

Imagine you have three web servers behind a load balancer, all serving the same website. A user uploads a photo to Server A. Without shared storage, Servers B and C have no idea that file exists. The next time that user's request lands on Server B, the image is missing.

NFS solves this: all three servers mount the same shared directory, so a file written by one server is immediately visible to the others.

### When To Use NFS

- **Shared uploads directory** — Multiple web servers accessing the same user-uploaded files
- **CMS media storage** — WordPress, Drupal, or other content management systems storing images and documents
- **CI/CD build artifacts** — Shared output from automated build pipelines
- **Internal file server** — A simple team-accessible file storage
- **AI/ML data pipelines** — Supplying training data to multiple compute nodes (often using the "High Performance" tier)

### When NOT To Use NFS

- For a database (too slow for random I/O — use block storage)
- For long-term archival of large files (use object storage — it's cheaper and more durable)
- When only one server needs access (block storage is simpler and faster)

### What My Professor Didn't Explain

**NFS is a protocol, not a product.** DigitalOcean's "Network File Storage" is their managed NFS service, but NFS itself is an open standard. You can set up your own NFS server on any Linux machine with the `nfs-kernel-server` package. The cloud-managed version just removes the burden of maintaining that server yourself.

**Performance:** NFS adds network latency to every file operation. Reading a file from NFS is always slower than reading it from a local disk. For most web applications, this is perfectly acceptable. For databases or applications that do thousands of tiny reads/writes per second, it is not.

**Reference:**
- [DigitalOcean NFS Product Page](https://www.digitalocean.com/products/storage/network-file-storage)

---

## 4. Block Storage (Volumes)

### What Is It?

**Block Storage** volumes act like virtual hard drives that you attach to a single VM. DigitalOcean calls these **Volumes**. On AWS, the equivalent is **EBS (Elastic Block Store)**.

Once attached, a volume appears to the operating system as a raw disk device (like `/dev/sda`). You format it with a filesystem (like `ext4`), mount it to a directory, and use it exactly like a local hard drive — because from the OS's perspective, it *is* a local hard drive.

### Why Does It Exist?

There is an important design principle in systems administration: **separate your data from your application logic.**

If your web server's code, configuration, *and* database all live on the VM's built-in disk, then replacing that VM (for an upgrade, a crash, or a redeployment) means losing everything. But if your database lives on a separate block storage volume, you can destroy the VM, create a new one, reattach the volume, and your data is untouched.

### When To Use Block Storage

- **Database storage** — PostgreSQL, MySQL, MongoDB (requires fast disk I/O)
- **Application data** — Anything that requires low-latency, high-throughput reads and writes
- **Log storage** — Logs for a single service that fill up quickly
- **Stateful services** — Any service that stores data locally and must survive VM replacements

### Real-World Analogy

This is exactly how many people already use their personal computers:
- The **operating system** can be reinstalled or replaced
- **Important data** is stored separately (an external hard drive, a USB drive, cloud sync)

Block storage is the cloud version of plugging in an external hard drive.

### What My Professor Didn't Explain

**Block storage is "single-attach."** Unlike NFS, a block storage volume can only be connected to **one VM at a time**. If you need multiple VMs to access the same data, block storage is the wrong choice — use NFS instead.

**You should still maintain independent backups.** Block storage is more durable than a VM's built-in disk, but it is not a backup strategy. If you accidentally delete a file on a volume, it's gone. Use snapshots, `rsync`, or object storage backups as a safety net.

**Reference:**
- [DigitalOcean Volumes Product Page](https://www.digitalocean.com/products/block-storage)

---

## 5. Connecting to Network File Storage — Hands-On

### Before You Begin: Regions Matter

Cloud resources are hosted in specific **regions** — physical data centers around the world (e.g., `NYC2` for New York, `SFO3` for San Francisco). A critical rule:

> **Resources that need to communicate over a private network must be in the same region.**

This means your NFS share and the Droplets (VMs) that connect to it **must be in the same region**. If your NFS is in `NYC2` and your Droplet is in `SFO3`, they cannot see each other.

This is true across all cloud providers, not just DigitalOcean. When planning infrastructure, always verify that the services you need are available in your chosen region.

### What My Professor Didn't Explain

**Why same-region?** Cloud providers connect machines within the same data center using a high-speed **private network** (DigitalOcean calls it a **VPC — Virtual Private Cloud**). Traffic on this network is fast, free, and not exposed to the public internet. Cross-region traffic would have to travel over the public internet — it would be slower, less secure, and cost money.

The **Mount Source** you see in the DigitalOcean NFS dashboard (e.g., `10.100.0.3:/exports/data`) is a private IP address on this VPC. It is not reachable from the internet — only from Droplets in the same region and VPC.

### Creating NFS in DigitalOcean — Key Settings

When creating Network File Storage in DigitalOcean:

1. **Region:** Choose the region where your Droplets already exist (e.g., `NYC2`)
2. **Performance tier:** "Standard" is fine for testing and most workloads. Use "High Performance" for I/O-intensive applications like AI/ML training data
3. **Size:** Minimum is 50 GiB for the standard tier — sufficient for testing. You can resize later
4. **Name:** Use a descriptive name in production (e.g., `nfs-chatty-testing`). The default name is acceptable for learning

### Step 1: SSH Into Your Droplet

All NFS connection steps happen **on the Droplet**, not in the DigitalOcean dashboard. Connect via SSH first:

```bash
ssh your-user@your-droplet-ip
```

### Step 2: Install the NFS Client Package

Your Debian Droplet does not include NFS support by default. You need the `nfs-common` package, which provides the tools to mount remote NFS shares.

```bash
sudo apt update
sudo apt install nfs-common
```

#### Command Breakdown

| Part | Purpose |
|---|---|
| `sudo` | Run as root — installing packages requires administrator privileges |
| `apt update` | Refresh the local package index so `apt` knows what's available |
| `apt install nfs-common` | Download and install the NFS client utilities |

**Why two commands?** `apt update` does not install anything — it only downloads the latest list of available packages from the Debian repositories. Without it, `apt install` might try to install an outdated version or fail to find the package at all. Always `update` before `install`.

### Step 3: Create a Mount Point

A **mount point** is simply an empty directory where the remote filesystem will appear. By Linux convention, external and temporary filesystems go in `/mnt`:

```bash
sudo mkdir -p /mnt/share
```

#### Command Breakdown

| Part | Purpose |
|---|---|
| `sudo` | Creating directories in `/mnt` requires root privileges |
| `mkdir` | "Make directory" — creates a new folder |
| `-p` | "Parents" — creates parent directories if they don't exist, and doesn't error if the directory already exists |
| `/mnt/share` | The path where the NFS share will be accessible |

After mounting, you will access your shared files by navigating to `/mnt/share` — it will look and feel like a local directory.

### Step 4: Mount the NFS Share (Manual/Temporary)

```bash
sudo mount -t nfs -o nconnect=8,rw 10.100.0.3:/exports/data /mnt/share
```

Replace `10.100.0.3:/exports/data` with your actual **Mount Source** from the DigitalOcean NFS dashboard.

#### Command Breakdown

| Part | Purpose |
|---|---|
| `sudo` | Mounting filesystems requires root privileges |
| `mount` | The Linux command for attaching a filesystem to a directory |
| `-t nfs` | **Type** — tells `mount` this is an NFS filesystem (not ext4, not FAT, etc.) |
| `-o` | **Options** — a comma-separated list of mount options (see below) |
| `nconnect=8` | Opens **8 parallel TCP connections** to the NFS server. This dramatically improves performance for workloads with many simultaneous reads and writes. Requires NFSv4.1 or newer |
| `rw` | **Read-write** — allows both reading and writing files. This is actually the default, but being explicit is good practice |
| `10.100.0.3:/exports/data` | The **Mount Source** — the private IP and path of the remote NFS share |
| `/mnt/share` | The **local mount point** — where the remote filesystem will appear on your machine |

### Verifying the Mount

After mounting, confirm it worked:

```bash
# Check that the NFS is mounted
df -h /mnt/share
```

Sample output:
```
Filesystem                Size  Used Avail Use% Mounted on
10.100.0.3:/exports/data   50G  1.2G   49G   3% /mnt/share
```

You can also test by creating a file:

```bash
# Create a test file
echo "Hello from Droplet A" | sudo tee /mnt/share/test.txt

# Read it back
cat /mnt/share/test.txt
```

If you have a second Droplet connected to the same NFS, you will see `test.txt` there too — that's the power of shared storage.

### Warning: This Mount Is Temporary

The `mount` command creates an **ephemeral mount** — it only lasts until the next reboot. If you restart your Droplet, the NFS share will not be there. You would have to run the `mount` command again.

For a persistent mount that survives reboots, you need one of the following two methods.

---

## 6. Mounting Filesystems in Linux

### What Does "Mounting" Mean?

In Linux, **every storage device** must be "mounted" before you can use it. Mounting is the act of attaching a filesystem (whether it's a hard drive, a USB stick, a network share, or a virtual disk) to a specific directory in the filesystem tree.

Unlike Windows, which assigns drive letters (`C:`, `D:`, `E:`), Linux has a single unified directory tree starting at `/` (the **root**). Every device is mounted *somewhere* within this tree.

For example:
- Your main hard drive is mounted at `/` (the root)
- A USB stick might be mounted at `/media/usb`
- An NFS share might be mounted at `/mnt/share`
- A boot partition might be mounted at `/boot/efi`

Once mounted, you interact with the contents using normal commands (`ls`, `cd`, `cp`, `cat`) — Linux abstracts away the fact that the data might live on a different device or even a different machine.

### What My Professor Didn't Explain

**Why does Linux require mounting?** In Linux, the kernel doesn't automatically know how to read every type of storage. Mounting tells the kernel: "This device uses the ext4 filesystem, and I want its contents to appear at this directory." The kernel then loads the appropriate filesystem driver and makes the contents accessible.

This is why `mount` needs the `-t` flag — it tells the kernel *which* filesystem driver to use. Common types:
- `ext4` — The default Linux filesystem
- `nfs` — Network File System
- `vfat` — FAT32, commonly used on USB sticks and EFI boot partitions
- `xfs` — A high-performance filesystem used in enterprise Linux (default on Red Hat/CentOS)

---

## 7. Persistent Mounts with `/etc/fstab`

### What Is `/etc/fstab`?

**`/etc/fstab`** stands for **"file systems table."** It is a plain-text configuration file that tells Linux which filesystems to mount automatically at boot time, and how to mount them.

Every Linux system has this file. Without it, you would have to manually mount every disk, every partition, and every network share every time you restarted your computer.

### Why Does It Exist?

When Linux boots, it needs to know:
- Where is the root filesystem (`/`)?
- Where is the boot partition (`/boot/efi`)?
- Are there any network shares to connect to?
- What options should each filesystem use?

`/etc/fstab` answers all of these questions in one file, and the system reads it automatically during startup.

### What Does It Look Like?

Here is what `/etc/fstab` looks like on a typical DigitalOcean Debian Droplet:

```
PARTUUID=0d819da1-7f79-4915-ae69-7ca9d101a78f  /          ext4  rw,discard,errors=remount-ro,x-systemd.growfs  0  1
PARTUUID=62694ba1-1d59-48ca-b1bc-f3a0c31bbf2c  /boot/efi  vfat  defaults,umask=077                             0  2
```

### Understanding the Format

Each line in `/etc/fstab` has **six fields**, separated by spaces or tabs:

```
<source>   <mount_point>   <type>   <options>   <dump>   <pass>
```

| Field | What It Means | Example |
|---|---|---|
| **`<source>`** | The device or remote share to mount. Can be a UUID, PARTUUID, device path (`/dev/sda1`), or NFS path (`10.0.0.3:/share`) | `PARTUUID=0d819da1...` |
| **`<mount_point>`** | The directory where this filesystem will appear | `/` or `/boot/efi` |
| **`<type>`** | The filesystem type | `ext4`, `vfat`, `nfs` |
| **`<options>`** | Comma-separated mount options (controls behavior) | `rw,discard,errors=remount-ro` |
| **`<dump>`** | Used by an old backup utility called `dump`. Almost always `0` (disabled) in modern systems | `0` |
| **`<pass>`** | Filesystem check order at boot using `fsck`. `0` = don't check, `1` = check first (usually `/`), `2` = check after root | `1` or `2` |

### Adding an NFS Mount to `/etc/fstab`

To make your NFS share mount automatically at boot, open `/etc/fstab` in a text editor (like Vim) and add a new line:

```bash
sudo vim /etc/fstab
```

Add this line at the bottom (replace the IP and path with your actual Mount Source):

```
10.100.0.3:/exports/data  /mnt/share  nfs  _netdev,nofail,x-systemd.automount,x-systemd.idle-timeout=600,nconnect=8,vers=4.1  0  0
```

### Understanding the NFS Mount Options

These options are critical for a reliable NFS mount. Here is what each one does and why it matters:

| Option | What It Does | Why It Matters |
|---|---|---|
| **`_netdev`** | Marks this as a **network filesystem** | Tells the system to wait for networking to be fully up before attempting the mount. Without this, the system tries to mount the NFS share before the network is ready, and it fails |
| **`nofail`** | If the mount fails, **boot continues anyway** | Without this, a failed NFS mount (e.g., the NFS server is down) drops your system into **emergency mode** — the server becomes unusable until you fix the mount. `nofail` prevents this |
| **`x-systemd.automount`** | Tells **systemd** to mount the share **on first access**, not at boot | The NFS share isn't actually mounted until you (or a program) first tries to access `/mnt/share`. This makes boot faster and avoids problems if the NFS server takes a moment to become available |
| **`x-systemd.idle-timeout=600`** | Automatically **unmounts** the share after 600 seconds (10 minutes) of inactivity | Works with `automount`. Frees resources and avoids stale mounts if the NFS server disappears. The share remounts automatically the next time you access it |
| **`nconnect=8`** | Opens 8 parallel TCP connections | Improves NFS performance (same as the manual mount) |
| **`vers=4.1`** | Explicitly requests **NFS version 4.1** | Ensures compatibility with features like `nconnect`. Without this, the client might negotiate an older version |

### After Editing `/etc/fstab`

You don't need to reboot to test your new entry. Use:

```bash
# Re-read /etc/fstab and mount everything that isn't already mounted
sudo mount -a
```

Then verify:

```bash
df -h /mnt/share
```

### Warnings and Gotchas

- **A typo in `/etc/fstab` can prevent your server from booting.** Always double-check your entry. If you add a mount without `nofail` and the source is unreachable, the system will hang at boot or drop into emergency mode.
- **Use `nofail` on every non-essential mount.** Only the root filesystem (`/`) and boot partition should ever be allowed to block booting.
- **Test before rebooting.** Run `sudo mount -a` after editing. If it produces errors, fix them before rebooting.

---

## 8. Persistent Mounts with systemd `.mount` Unit Files

### What Is a `.mount` Unit File?

**systemd** is the init system (the first process that runs at boot) on most modern Linux distributions. It manages services, timers, mounts, and much more using **unit files** — small configuration files that describe what to run and how.

A `.mount` unit file is a systemd-specific way to define a mount point. It is an alternative to `/etc/fstab` that some administrators prefer for its readability and integration with other systemd features.

### Why Use a `.mount` File Instead of `/etc/fstab`?

| | `/etc/fstab` | systemd `.mount` file |
|---|---|---|
| **Format** | Dense, space-separated columns | INI-style sections, clearly labeled |
| **Readability** | Harder to read for complex entries | Easier to read and understand |
| **Recommended for** | Personal systems, simple setups | Managed infrastructure, automation |
| **Integration** | Works with systemd (translated internally) | Native systemd — can depend on other services |

The systemd `systemd.mount` man page recommends using `/etc/fstab` for human-managed systems (like your personal desktop) and unit files for systems managed by automation tools (like cloud-init, Ansible, or Puppet).

### Critical Rule: The Filename Must Match the Path

> **Warning:** The `.mount` filename **must** correspond exactly to the mount point path. Take the path, remove the leading `/`, and replace all remaining `/` characters with `-` (dashes).

Examples:
| Mount Point | Filename |
|---|---|
| `/mnt/share` | `mnt-share.mount` |
| `/mnt/data/backups` | `mnt-data-backups.mount` |
| `/srv/nfs/exports` | `srv-nfs-exports.mount` |

If the filename doesn't match the path, systemd will silently ignore it or produce confusing errors.

### Example `.mount` File

Create the file at `/etc/systemd/system/mnt-share.mount`:

```ini
[Unit]
Description=Mount NFS Share /mnt/share
After=network-online.target
Wants=network-online.target

[Mount]
What=10.100.0.3:/13308319/0f8f2e71-fcf9-41c9-b4e7-f8632c6f8f83
Where=/mnt/share
Type=nfs
Options=nconnect=8,ro
TimeoutSec=30

[Install]
WantedBy=multi-user.target
```

### Understanding Each Section

**`[Unit]` Section — Metadata and Dependencies**

| Directive | Purpose |
|---|---|
| `Description=` | A human-readable description of what this mount does |
| `After=network-online.target` | Don't attempt this mount until networking is fully up |
| `Wants=network-online.target` | Actively request that the network target is activated (a "soft dependency") |

**Why both `After` and `Wants`?** `After=` only controls *ordering* — "if network-online happens, do this after it." `Wants=` actually *requests* that network-online is started. Together, they ensure the network is both requested and fully ready before the mount is attempted.

**`[Mount]` Section — The Mount Configuration**

| Directive | Purpose |
|---|---|
| `What=` | The source — the remote NFS path (your Mount Source from DigitalOcean) |
| `Where=` | The local directory to mount to (must match the filename) |
| `Type=` | The filesystem type (`nfs`) |
| `Options=` | Mount options, comma-separated |
| `TimeoutSec=` | How long to wait before giving up (30 seconds) |

**Note the `ro` option:** This example uses `ro` (read-only) instead of `rw` (read-write). This is intentional — there are real scenarios where you want some servers to have read-only access. For example:
- **Server A** (the data team) mounts with `rw` and can upload new data
- **Server B** (the analytics service) mounts with `ro` and can only read data, preventing accidental modifications

**`[Install]` Section — When to Activate**

| Directive | Purpose |
|---|---|
| `WantedBy=multi-user.target` | Tells systemd to activate this mount when the system reaches "multi-user" mode (normal operation, no graphical desktop). This is the standard target for servers |

### Activating the Mount Unit

After creating the file, you need to tell systemd about it and start it:

```bash
# 1. Reload systemd so it discovers the new unit file
sudo systemctl daemon-reload

# 2. Start the mount immediately
sudo systemctl start mnt-share.mount

# 3. Enable it to start automatically at every boot
sudo systemctl enable mnt-share.mount
```

#### Command Breakdown

| Command | What It Does |
|---|---|
| `systemctl daemon-reload` | systemd caches its unit files in memory. After creating or editing any unit file, you must run this so systemd re-reads the configuration. **This does not start or restart anything** — it just refreshes the catalog |
| `systemctl start mnt-share.mount` | Activates the mount right now (equivalent to running the `mount` command) |
| `systemctl enable mnt-share.mount` | Creates a symbolic link that tells systemd to automatically start this mount at every future boot |

### Checking the Mount Status

```bash
# See if the mount is active
sudo systemctl status mnt-share.mount
```

Sample output:
```
● mnt-share.mount - Mount NFS Share /mnt/share
     Loaded: loaded (/etc/systemd/system/mnt-share.mount; enabled; preset: enabled)
     Active: active (mounted) since Tue 2026-04-07 10:30:00 UTC; 5min ago
      Where: /mnt/share
       What: 10.100.0.3:/exports/data
```

### When To Use Which Method — Summary

| Scenario | Recommended Method |
|---|---|
| Personal desktop or laptop | `/etc/fstab` |
| Simple server with one or two mounts | `/etc/fstab` |
| Servers managed by automation (cloud-init, Ansible) | `.mount` unit files |
| Complex dependencies (mount after a specific service starts) | `.mount` unit files |
| Quick temporary testing | `mount` command (no persistence) |

---

## 9. Cloud-Init — Automated Server Setup

### What Is Cloud-Init?

**Cloud-init** is the industry-standard tool for the **early initialization** of cloud virtual machines. It is the bridge between a generic operating system image and a fully configured, ready-to-use server.

When you create a VM in the cloud, the provider gives you a **base image** — a vanilla installation of Debian, Ubuntu, Rocky Linux, etc. It has no users configured, no software installed, no SSH keys, no custom settings. It's like a blank canvas.

Cloud-init takes a configuration file you write and automatically applies it during the very first boot. It can:
- Create user accounts
- Install software packages
- Configure SSH keys
- Write files to disk
- Set the hostname and timezone
- Run arbitrary shell commands
- And much more

### Why Does Cloud-Init Exist?

Imagine you need to deploy 50 identical web servers. Without cloud-init, you would have to:

1. Create each VM
2. SSH into each one
3. Manually run `apt update && apt install nginx git curl`
4. Manually create a user account
5. Manually configure SSH keys
6. Manually write configuration files
7. ...repeat 49 more times

This is slow, error-prone, and produces **"snowflake servers"** — machines that were supposed to be identical but have subtle differences because of human error.

Cloud-init eliminates this problem. You write a configuration file once, and every server provisioned with that file is configured **exactly** the same way, every time.

### How Cloud-Init Works (The Big Picture)

Cloud-init operates on a **declarative model**. Instead of writing step-by-step instructions ("first install nginx, then create a user, then..."), you describe the **desired end state** ("I want nginx installed, a user named tron, and these SSH keys"). Cloud-init figures out how to make that happen.

#### The Boot Sequence

When a VM starts for the first time, cloud-init runs through four stages:

| Stage | What Happens |
|---|---|
| **1. Local** | cloud-init detects the cloud platform and retrieves basic configuration (before networking is up) |
| **2. Network** | Networking is configured, cloud-init reaches out to the metadata service for your configuration |
| **3. Config** | Your configuration directives are applied — packages installed, users created, files written |
| **4. Final** | Final commands are executed (the `runcmd` section), and cloud-init signals completion |

#### Where Does Cloud-Init Get Your Configuration?

When the VM boots, cloud-init looks for a **datasource** — a source of configuration data provided by the cloud platform. On DigitalOcean, this is a metadata service accessible at a special internal IP address. The datasource provides two things:

- **Metadata** — Information about the VM itself (hostname, region, IP addresses)
- **User-data** — Your custom configuration (the cloud-init YAML file you write)

### Who Makes Cloud-Init?

Cloud-init is an **open-source project** primarily maintained by **Canonical** (the company behind Ubuntu). However, because it is so essential to cloud computing, all major cloud providers and Linux distributions contribute to it — including Red Hat, Amazon, VMware, and Microsoft.

It is pre-installed on virtually every cloud image you will encounter: Debian, Ubuntu, Rocky Linux, Fedora, Amazon Linux, and more.

### Why Should You Learn Cloud-Init?

- **Infrastructure as Code (IaC):** Cloud-init is often the "last mile" of tools like Terraform. Terraform creates the VM; cloud-init configures it.
- **Consistency:** Every server built from the same cloud-init config is identical — no snowflakes.
- **Transferable skills:** Whether you work with AWS, Azure, Google Cloud, OpenStack, or Proxmox, cloud-init is the standard.
- **Efficiency:** Automate the tedious setup (SSH keys, timezone, updates, basic packages) so you can focus on architecture and problem-solving.

---

## 10. YAML — The Configuration Language

### What Is YAML?

**YAML** (YAML Ain't Markup Language) is a **data serialization format** — a way to represent structured data as plain text. It is similar to JSON but designed to be more human-readable.

Cloud-init configuration files are written in YAML. You will also encounter YAML in Docker Compose, Kubernetes, Ansible, GitHub Actions, and many other DevOps tools.

### YAML Basics You Need To Know

**Indentation matters.** YAML uses spaces (never tabs) to define structure. Indentation indicates nesting — like Python.

```yaml
# A simple key-value pair
name: tron

# A list (array)
packages:
  - nginx
  - git
  - curl

# A nested object (dictionary)
user:
  name: tron
  shell: /bin/bash
  groups: sudo
```

### Key YAML Rules

- **Use spaces, not tabs.** Tabs will cause parsing errors. Use 2 spaces per indent level (this is the convention).
- **Colons separate keys and values:** `key: value` (note the space after the colon).
- **Dashes denote list items:** `- item` (note the space after the dash).
- **The pipe `|` preserves line breaks** in multi-line strings (like writing an HTML file).
- **Comments start with `#`.**

### Common YAML Mistakes

| Mistake | What Happens |
|---|---|
| Using tabs instead of spaces | YAML parser throws an error |
| Missing space after `:` | `key:value` is invalid — must be `key: value` |
| Inconsistent indentation | Child items appear at wrong nesting level or cause errors |
| Forgetting the `#cloud-config` header | Cloud-init ignores the entire file |

---

## 11. Writing a Cloud-Init Configuration File

### The Configuration File

Create a file called `cloud-config.yaml` on your local machine. This is the file you will paste into DigitalOcean's "Initialization scripts" field when creating a Droplet.

### Complete Annotated Example

```yaml
#cloud-config

# ──────────────────────────────────────────────
# 1. PACKAGE MANAGEMENT
# ──────────────────────────────────────────────
# Update the package index (like running "apt update")
package_update: true

# Upgrade all installed packages to their latest versions (like "apt upgrade -y")
package_upgrade: true

# ──────────────────────────────────────────────
# 2. INSTALL PACKAGES
# ──────────────────────────────────────────────
# List packages to install. Cloud-init automatically uses the correct
# package manager for the distro (apt on Debian/Ubuntu, dnf on Fedora, etc.)
packages:
  - nginx
  - git
  - curl

# ──────────────────────────────────────────────
# 3. CREATE USERS
# ──────────────────────────────────────────────
users:
  - name: tron
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... your-key-comment

# ──────────────────────────────────────────────
# 4. WRITE FILES
# ──────────────────────────────────────────────
# Create or overwrite files on the system
write_files:
  - owner: www-data:www-data
    path: /var/www/html/index.html
    content: |
      <html>
        <body>
          <h1>Server Provisioned via Cloud-Init</h1>
          <p>Status: Operational</p>
        </body>
      </html>

# ──────────────────────────────────────────────
# 5. RUN COMMANDS (last resort)
# ──────────────────────────────────────────────
runcmd:
  - [ systemctl, enable, nginx ]
  - [ systemctl, start, nginx ]
  - echo "Provisioning complete at $(date)" >> /var/log/provision.log
```

### Section-by-Section Breakdown

#### `package_update` and `package_upgrade`

```yaml
package_update: true
package_upgrade: true
```

- **`package_update: true`** — Runs the equivalent of `sudo apt update` before installing anything. This refreshes the local package index so the system knows about the latest available versions.
- **`package_upgrade: true`** — Runs the equivalent of `sudo apt upgrade -y` to update all pre-installed packages to their latest versions.

**Why both?** The base image might be weeks or months old. Packages (including security patches) may have been updated since the image was built. Running both ensures your server starts fully patched.

#### `packages`

```yaml
packages:
  - nginx
  - git
  - curl
```

This installs the listed packages using the distribution's native package manager. Cloud-init detects the distro automatically:
- On **Debian/Ubuntu**, it uses `apt`
- On **Fedora/Rocky**, it uses `dnf`
- On **Arch**, it uses `pacman`

**Important caveat:** The package name must exist in the distro's repository. Most common packages (like `nginx`, `git`, `curl`) have the same name everywhere, but some do not. For example, the Apache web server is called `apache2` on Debian but `httpd` on Red Hat-based systems.

#### `users`

```yaml
users:
  - name: tron
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... your-key-comment
```

This creates a new user account with the following configuration:

| Directive | Purpose |
|---|---|
| `name: tron` | The username for the new account |
| `groups: sudo` | Add this user to the `sudo` group, granting them administrative privileges |
| `shell: /bin/bash` | Set `bash` as the user's default shell (what runs when they log in) |
| `sudo: ['ALL=(ALL) NOPASSWD:ALL']` | Allow this user to run any command with `sudo` without being prompted for a password |
| `ssh_authorized_keys:` | A list of public SSH keys that can log in as this user. Replace the placeholder with your actual public key |

**What My Professor Didn't Explain:** The `NOPASSWD:ALL` directive means this user can run `sudo rm -rf /` without being asked for a password. This is convenient for automation but is a security trade-off. In production environments, you might want to remove `NOPASSWD` or restrict `sudo` to specific commands.

#### `write_files`

```yaml
write_files:
  - owner: www-data:www-data
    path: /var/www/html/index.html
    content: |
      <html>
        <body>
          <h1>Server Provisioned via Cloud-Init</h1>
          <p>Status: Operational</p>
        </body>
      </html>
```

This creates (or overwrites) a file at the specified path:

| Directive | Purpose |
|---|---|
| `owner: www-data:www-data` | Set the file's owner and group to `www-data` (the default user that Nginx runs as on Debian) |
| `path:` | The absolute path where the file will be created |
| `content: \|` | The `\|` (pipe) tells YAML to preserve line breaks in the following indented block. The HTML is written exactly as shown |

**Cross-distribution caveat:** This writes the file to `/var/www/html/`, which is Nginx's default document root on Debian/Ubuntu. On Fedora or Arch, Nginx might serve files from a different directory (e.g., `/usr/share/nginx/html/`). If you need to support multiple distros, also include a `write_files` entry that overwrites the Nginx configuration file itself.

#### `runcmd`

```yaml
runcmd:
  - [ systemctl, enable, nginx ]
  - [ systemctl, start, nginx ]
  - echo "Provisioning complete at $(date)" >> /var/log/provision.log
```

This runs arbitrary shell commands at the very end of the cloud-init process. Two syntax styles are shown:

- **List syntax:** `[ systemctl, enable, nginx ]` — Each element is passed as a separate argument. Safer because the shell does not interpret special characters.
- **String syntax:** `echo "Provisioning complete..."` — Passed to `/bin/sh` as a single string. Supports shell features like `$()` and `>>` but is more error-prone.

### Warning: Avoid `runcmd` When Possible

Commands in `runcmd` are often **not idempotent**. "Idempotent" means running the same action multiple times produces the same result. For example:
- `package_update: true` is idempotent — running it twice is harmless.
- `echo "text" >> /var/log/file` is **not** idempotent — running it twice appends the text twice.

**Best practice:** If cloud-init has a dedicated module for a task, use it instead of `runcmd`. For example, use the `packages:` directive to install packages — don't put `apt install` in `runcmd`.

---

## 12. Cloud-Init on DigitalOcean

### How To Use It

Cloud-init is **already installed and running** on DigitalOcean Droplets (Debian, Ubuntu, Rocky Linux, etc.). You do not need to install anything.

To use your cloud-init configuration:

1. Begin creating a new Droplet as normal (choose region, image, size, SSH keys)
2. Near the bottom of the creation page, click the blue **"Advanced Options"** menu
3. Check **"Add Initialization scripts"**
4. A text field appears — paste your entire `cloud-config.yaml` content into this field
5. Continue creating the Droplet as usual

When the Droplet boots for the first time, cloud-init reads your configuration from DigitalOcean's metadata service and applies it automatically.

### How DigitalOcean Delivers Your Configuration

This is the same mechanism used for SSH keys. When you create a Droplet and select an SSH key, DigitalOcean passes that key to the VM through its metadata service. Cloud-init then picks it up and places it in the appropriate user's `~/.ssh/authorized_keys` file.

Your cloud-init YAML goes through the exact same pipeline — you paste it in the dashboard, DigitalOcean delivers it via metadata, and cloud-init processes it on first boot.

### Verifying Cloud-Init Ran Successfully

After your Droplet is created and you've SSH'd in, check the cloud-init logs:

```bash
# View cloud-init status
cloud-init status

# View detailed cloud-init logs
sudo cat /var/log/cloud-init-output.log
```

If something went wrong, the log will tell you exactly which module failed and why.

**Reference:**
- [cloud-init Official Documentation](https://docs.cloud-init.io/en/latest/index.html)
- [YAML Specification](https://yaml.org)

---

## 13. Key Takeaways and Study Tips

### Storage — Know the Three Types

| Question | Object Storage | Network File Storage | Block Storage |
|---|---|---|---|
| What is it like? | A locker with labeled bins | A shared office drive | An external hard drive |
| Accessed via? | API (HTTP) | Mounted directory (NFS) | Mounted block device |
| Shared across VMs? | Yes (via URLs) | Yes (multiple mounts) | No (single VM only) |
| Best for? | Backups, media, static files | Shared files across servers | Databases, fast I/O |

### Mounting — Three Methods

| Method | Persists after reboot? | Best for |
|---|---|---|
| `mount` command | No | Quick testing |
| `/etc/fstab` entry | Yes | Personal systems, simple setups |
| systemd `.mount` file | Yes | Managed/automated infrastructure |

### Cloud-Init — The Key Modules

| Module | What It Does | Idempotent? |
|---|---|---|
| `package_update` | Refreshes package index | Yes |
| `package_upgrade` | Upgrades all packages | Yes |
| `packages` | Installs listed packages | Yes |
| `users` | Creates user accounts | Yes |
| `write_files` | Creates/overwrites files | Yes |
| `runcmd` | Runs shell commands | Depends on the command |

### Study Focus (Per the Professor's Advice)

You do not need to memorize every mount option or cloud-init directive. Focus on:

- **What are the components involved?** (NFS server, NFS client, mount points, fstab, systemd, cloud-init, YAML)
- **How do they fit together?** (A cloud-init config installs `nfs-common`, creates a mount unit file, and reloads systemd — automating everything we did manually)
- **When would you choose one approach over another?** (fstab vs. mount file, object storage vs. NFS vs. block storage)

---

> **References:**
> - [DigitalOcean Spaces](https://www.digitalocean.com/products/spaces)
> - [DigitalOcean NFS](https://www.digitalocean.com/products/storage/network-file-storage)
> - [DigitalOcean Block Storage](https://www.digitalocean.com/products/block-storage)
> - [DigitalOcean NFS Docs](https://docs.digitalocean.com/products/nfs/)
> - [DigitalOcean VPC](https://www.digitalocean.com/products/vpc)
> - [cloud-init Official Documentation](https://docs.cloud-init.io/en/latest/index.html)
> - [YAML Specification](https://yaml.org)
