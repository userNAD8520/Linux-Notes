# Linux Study Guide: SFTP, Web Servers, and Cloud Infrastructure

> **Course Context:** These notes cover transferring files with `sftp`, core web infrastructure terminology (web servers, reverse proxies, load balancers, firewalls, APIs), and a full hands-on walkthrough of deploying a web server with Caddy on DigitalOcean — including setting up a load balancer and a cloud firewall.

---

## Table of Contents

1. [SFTP — Secure File Transfer](#1-sftp--secure-file-transfer)
2. [Connecting to a Server with `sftp`](#2-connecting-to-a-server-with-sftp)
3. [Navigating in an `sftp` Session](#3-navigating-in-an-sftp-session)
4. [Transferring Files and Directories](#4-transferring-files-and-directories)
5. [Web Servers — What They Are and Why They Exist](#5-web-servers--what-they-are-and-why-they-exist)
6. [Nginx — The Industry Workhorse](#6-nginx--the-industry-workhorse)
7. [Caddy — The Modern Alternative](#7-caddy--the-modern-alternative)
8. [Reverse Proxy Servers](#8-reverse-proxy-servers)
9. [APIs — Application Programming Interfaces](#9-apis--application-programming-interfaces)
10. [Load Balancers](#10-load-balancers)
11. [Firewalls](#11-firewalls)
12. [`curl` — Testing from the Command Line](#12-curl--testing-from-the-command-line)
13. [Hands-On: Deploying a Web Server on DigitalOcean](#13-hands-on-deploying-a-web-server-on-digitalocean)
14. [Key Takeaways and Study Tips](#14-key-takeaways-and-study-tips)

---

## 1. SFTP — Secure File Transfer

### What Is `sftp`?

**`sftp`** (SSH File Transfer Protocol) is a command-line tool for transferring files and directories between your local machine and a remote server. It works over an **encrypted SSH connection**, meaning everything you transfer — filenames, file contents, passwords — is protected from eavesdropping.

> From the man page: `sftp` is a file transfer program, similar to `ftp`, which performs all operations over an encrypted SSH transport. It may also use many features of SSH, such as public key authentication and compression.

### Why Does `sftp` Exist?

So far in the course, you may have been copying and pasting file contents between your terminal and your server. This works for a 10-line script, but it falls apart quickly when:

- The file is large (a binary, an image, a compiled program)
- You need to transfer many files at once
- You need to transfer an entire directory tree
- You want to avoid copy-paste errors (wrong line breaks, missing characters, encoding issues)

`sftp` solves all of these problems. It gives you an interactive session where you can browse directories on both your local machine and your server, then transfer files in either direction with simple commands.

### Why `sftp` and Not Regular `ftp`?

The original **FTP** (File Transfer Protocol) sends everything — including your username and password — in **plain text** over the network. Anyone on the same network can intercept and read it. This is a serious security risk.

**`sftp`** is not just "FTP with encryption bolted on." It is a completely different protocol that happens to provide similar functionality. It runs inside an SSH session, which means:

- All data is encrypted end-to-end
- Authentication uses SSH keys (no passwords sent over the network)
- It works through the same port as SSH (port 22) — no extra firewall rules needed

### What My Professor Didn't Explain

**`sftp` is not the same as FTPS.** There are three different file transfer protocols that beginners often confuse:

| Protocol | How It Works | Secure? |
|---|---|---|
| **FTP** | Original protocol, plain text | No — do not use |
| **FTPS** | FTP wrapped in TLS/SSL encryption | Yes, but complex to configure |
| **SFTP** | Completely different protocol running over SSH | Yes — the simplest and most common choice |

When someone says "use FTP," they almost always mean `sftp` in a modern context.

**Reference:**
- [OpenBSD sftp man page](https://man.openbsd.org/sftp.1)

---

## 2. Connecting to a Server with `sftp`

### Prerequisites

`sftp` uses SSH under the hood. If you can already SSH into your server, you can use `sftp` with the exact same configuration — same SSH keys, same `~/.ssh/config` file, same passphrase.

### Connecting

If you've been connecting to your server with:

```bash
ssh debian
```

Then connecting with `sftp` is as simple as replacing `ssh` with `sftp`:

```bash
sftp debian
```

Here, `debian` is the **host alias** you defined in your `~/.ssh/config` file (set up in Week 1). If you set up a passphrase on your SSH key, you will be prompted for it — this is the same passphrase you enter for `ssh`, because `sftp` is using the same SSH connection.

### What You'll See

After a successful connection, your prompt changes:

```
sftp>
```

This is the **sftp interactive prompt**. You are now inside an sftp session — you can run sftp-specific commands (not regular shell commands like `apt` or `systemctl`). Think of it as a special mini-shell designed only for file navigation and transfer.

### Disconnecting

To close your sftp session, use any of:

```
sftp> bye
```

```
sftp> exit
```

```
sftp> quit
```

All three do the same thing — close the connection and return you to your local shell.

### Getting Help Inside `sftp`

While connected, type `help` or `?` to see all available commands:

```
sftp> help
```

This displays a full list of commands with brief descriptions. You do not need to memorize them — the help is always one keystroke away.

---

## 3. Navigating in an `sftp` Session

### The Two-Context Model

Here is the most important concept to understand about `sftp`: **you are operating in two places at once.**

- The **remote** side — your server (the machine you connected to)
- The **local** side — your own computer (where you ran the `sftp` command)

Every navigation and listing command has two versions:

| Action | Remote (Server) Command | Local (Your Machine) Command |
|---|---|---|
| Print working directory | `pwd` | `lpwd` |
| List files | `ls` | `lls` |
| Change directory | `cd path` | `lcd path` |
| Create directory | `mkdir path` | `lmkdir path` |

The **`l` prefix** stands for **"local."** Commands without the `l` prefix operate on the remote server.

### Starting Directories

When you connect:
- **Remote:** You start in your **home directory** on the server (e.g., `/home/debian` or `/root`)
- **Local:** You start in whatever directory you were in when you ran the `sftp` command on your local machine

### Example Session — Exploring Both Sides

```
sftp> pwd
Remote working directory: /home/debian

sftp> lpwd
Local working directory: /home/user/Documents

sftp> ls
scripts/  configs/  README.md

sftp> lls
notes.txt  project/  backup.tar.gz

sftp> cd scripts
sftp> pwd
Remote working directory: /home/debian/scripts

sftp> lcd project
sftp> lpwd
Local working directory: /home/user/Documents/project
```

### What My Professor Didn't Explain

**Why the "l" prefix?** This design pattern comes from the original `ftp` client and has been preserved in `sftp`. It can be confusing at first because in a normal SSH session, every command runs on the remote server. In `sftp`, you need to be conscious of *which machine* you're affecting. If you forget the `l`, you'll be listing or changing directories on the server instead of your local machine.

**Running local shell commands:** If you need to run a regular shell command on your local machine during an sftp session, prefix it with `!`:

```
sftp> !ls -la ~/Downloads
```

Or type just `!` to drop into a local shell entirely:

```
sftp> !
user@laptop:~$ # You're now in a local shell
user@laptop:~$ exit
sftp> # Back in sftp
```

---

## 4. Transferring Files and Directories

### Uploading: `put` (Local to Server)

The **`put`** command transfers files **from your local machine to the server**. Think of it as "putting" a file onto the server.

#### Upload a Single File

```
sftp> put file.txt
```

This uploads `file.txt` from your **current local directory** to your **current remote directory**.

#### Upload a File Using a Full Path

```
sftp> put /home/user/Documents/config.yaml
```

This uploads the file from the specified local path to whatever remote directory you're currently in.

#### Upload an Entire Directory

```
sftp> put -R myproject
```

The **`-R`** flag means **"recursive"** — it copies the directory and everything inside it (all files, subdirectories, and their contents).

### Downloading: `get` (Server to Local)

The **`get`** command transfers files **from the server to your local machine**. Think of it as "getting" a file from the server.

#### Download a Single File

```
sftp> get server-log.txt
```

This downloads `server-log.txt` from your **current remote directory** to your **current local directory**.

#### Download a File Using a Full Path

```
sftp> get /home/debian/.local/bin/script.sh
```

This downloads the file from the specified remote path to your current local directory.

#### Download an Entire Directory

```
sftp> get -R configs/
```

Again, `-R` means recursive — it downloads the entire directory tree.

### Quick Reference

| What You Want | Command | Direction |
|---|---|---|
| Upload a file | `put filename` | Local → Server |
| Upload a directory | `put -R dirname` | Local → Server |
| Download a file | `get filename` | Server → Local |
| Download a directory | `get -R dirname` | Server → Local |
| Resume a failed upload | `reput filename` | Local → Server |
| Resume a failed download | `reget filename` | Server → Local |

### Practical Example: Full Workflow

Suppose you have a script called `deploy.sh` on your local machine in `~/scripts/` and you want to upload it to your server's `/usr/local/bin/` directory:

```
# Connect to the server
$ sftp debian

# Verify where you are on both sides
sftp> lpwd
Local working directory: /home/user

sftp> pwd
Remote working directory: /home/debian

# Navigate to the local directory containing the file
sftp> lcd scripts
sftp> lls
deploy.sh  cleanup.sh  backup.sh

# Navigate to the destination on the server
sftp> cd /usr/local/bin

# Upload the file
sftp> put deploy.sh
Uploading deploy.sh to /usr/local/bin/deploy.sh
deploy.sh                                  100%  1024     1.0KB/s   00:00

# Verify it arrived
sftp> ls deploy.sh
-rw-r--r--    1 debian   debian       1024 Apr  7 12:00 deploy.sh

# Done — disconnect
sftp> bye
```

### Warnings and Gotchas

- **Permissions after upload:** Files uploaded via `sftp` are owned by the user you connected as. If you need them owned by `root` or another user, you'll need to SSH in separately and use `chown`/`chmod`.
- **Overwriting:** `put` and `get` will overwrite existing files without warning. There is no confirmation prompt.
- **`-R` for directories is mandatory.** If you try `put mydirectory` without `-R`, it will fail — sftp does not know to recurse into subdirectories unless you tell it to.
- **Binary files work fine.** Unlike some old FTP clients that had "text mode" and "binary mode," sftp handles all file types correctly without any special settings.

**Reference:**
- [OpenBSD sftp man page](https://man.openbsd.org/sftp.1)

---

## 5. Web Servers — What They Are and Why They Exist

### What Is a Web Server?

A **web server** is software that listens for incoming network requests (typically HTTP or HTTPS) and responds by sending back documents — usually HTML pages, images, CSS files, JavaScript, or API responses.

In its simplest form, the cycle works like this:

1. A **client** (your web browser, a `curl` command, a mobile app) sends a request: "Give me the document at `/index.html`"
2. The **web server** receives the request
3. It finds the requested file on disk (or generates a response dynamically)
4. It sends the file back to the client
5. If something goes wrong (file not found, server error), it sends back an **error message** with an HTTP status code (like `404 Not Found` or `500 Internal Server Error`)

### Why Does a Web Server Need to Run as a Service?

A web server runs as a **background service** (a daemon) on a Linux machine. It starts at boot and runs continuously, waiting for requests 24/7. This is why you manage it with `systemctl`:

```bash
sudo systemctl start nginx     # Start the server
sudo systemctl enable nginx    # Start automatically at boot
sudo systemctl status nginx    # Check if it's running
```

If the web server process crashes or stops, no one can access your website until it's restarted.

### Key Configuration Concepts

Every web server has a configuration file that controls its behavior. The most important directives are:

| Directive | What It Controls | Example |
|---|---|---|
| **Listen** | Which port to accept connections on | Port `80` (HTTP) or `443` (HTTPS) |
| **Server name** | Which domain names this server responds to | `example.org`, `www.example.org` |
| **Document root** | The directory on disk where your website files live | `/web/html`, `/var/www/html` |
| **Index** | The default file served when someone visits `/` | `index.html` |

### Example: Nginx Configuration

Here is a minimal Nginx configuration to illustrate these concepts:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name _;
    root /web/html;

    location / {
        try_files $uri $uri/ =404;
        index index.html;
    }
}
```

#### Line-by-Line Breakdown

| Line | What It Does |
|---|---|
| `listen 80;` | Listen for HTTP connections on port 80 (IPv4) |
| `listen [::]:80;` | Listen for HTTP connections on port 80 (IPv6). The `[::]` is IPv6 notation for "all addresses" |
| `server_name _;` | The underscore `_` is a catch-all — respond to requests regardless of the domain name used. In production, you'd put your actual domain here (e.g., `server_name example.org www.example.org;`) |
| `root /web/html;` | Serve files from the `/web/html` directory on this Linux machine |
| `location / { ... }` | Rules for the root URL path (`/`) and everything under it |
| `try_files $uri $uri/ =404;` | When a request comes in, first look for a file matching the URL, then a directory, then return a 404 error |
| `index index.html;` | If someone visits `/` (the root), serve the file `index.html` |

### What My Professor Didn't Explain

**Ports 80 and 443:** When you type `http://example.com` in a browser, the browser silently adds `:80` to the end — `http://example.com:80`. For `https://`, it uses `:443`. These are the "well-known" ports for web traffic. A web server *must* listen on these ports for browsers to find it without extra configuration.

**HTTP vs. HTTPS:** HTTP sends everything in plain text — anyone between you and the server can read the traffic. HTTPS encrypts it with TLS. In production, you should always use HTTPS (port 443). The exercises use HTTP (port 80) for simplicity, since HTTPS requires a domain name and a TLS certificate.

**`server_name _;`** — The underscore is not a special Nginx keyword. It's a deliberate non-match: since no real domain name is `_`, it functions as a "default server" that handles any request not matched by another server block. This is a common pattern for simple or test configurations.

**Reference:**
- [Mozilla Developer Network — What is a web server?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_web_server)

---

## 6. Nginx — The Industry Workhorse

### What Is Nginx?

**Nginx** (pronounced "engine-x") is one of the most widely used web servers in the world. It was originally written by Igor Sysoev in 2004 to solve the "C10K problem" — how to handle 10,000 simultaneous connections on a single server.

Nginx is far more than just a web server. It can function as:

- An **HTTP web server** — Serving static files (HTML, CSS, images)
- A **reverse proxy** — Forwarding requests to backend application servers
- A **load balancer** — Distributing traffic across multiple servers
- A **content cache** — Storing frequently accessed responses to reduce backend load
- A **TCP/UDP proxy** — Forwarding non-HTTP traffic

### Why Is Nginx So Popular?

Nginx was designed from the ground up for **performance and concurrency**. It uses an event-driven, non-blocking architecture — meaning it can handle thousands of simultaneous connections with very little memory. This makes it ideal for high-traffic websites.

Major companies using Nginx include Netflix, Cloudflare, GitHub, and WordPress.com.

### What My Professor Didn't Explain

**Nginx vs. Apache:** Before Nginx, **Apache HTTP Server** was the dominant web server. Apache uses a "process-per-connection" model — each request gets its own process or thread. This works fine at small scale but consumes a lot of memory under heavy load. Nginx's event-driven model handles the same load with a fraction of the resources. Today, both are widely used, but Nginx tends to be preferred for high-traffic and reverse proxy scenarios.

**Reference:**
- [Nginx Official Site](https://nginx.org/en/)

---

## 7. Caddy — The Modern Alternative

### What Is Caddy?

**Caddy** is a modern web server written in Go. It is designed to be simpler to configure than Nginx while providing features that would require extra setup on Nginx — most notably, **automatic HTTPS**.

When you tell Caddy to serve a domain name, it automatically:
1. Obtains a free TLS certificate from Let's Encrypt
2. Configures HTTPS
3. Redirects HTTP traffic to HTTPS
4. Renews the certificate before it expires

With Nginx, each of these steps requires manual configuration or additional tools (like `certbot`).

### When To Use Caddy vs. Nginx

| Scenario | Recommended |
|---|---|
| Simple static site or reverse proxy | **Caddy** — easier config, automatic HTTPS |
| High-traffic production site (millions of requests) | **Nginx** — battle-tested, extremely tunable |
| Learning web server concepts | **Either** — Caddy for simplicity, Nginx for understanding the ecosystem |
| Existing infrastructure already using Nginx | **Nginx** — don't switch without a reason |

### What My Professor Didn't Explain

**Why is Caddy used in this course?** Caddy's configuration is dramatically simpler than Nginx's. A Caddyfile that would take 3 lines might require 20 lines in an Nginx config. For a course focused on understanding web infrastructure (not mastering one specific server's configuration language), Caddy lets you focus on the concepts.

**Caddy is a single binary.** Unlike Nginx, which you install via a package manager (`apt install nginx`), Caddy is distributed as a single compiled binary file. You download it, place it in `/usr/local/bin/`, and it's ready to run. This is why the course instructions have you copy the `caddy` file to `/usr/local/bin/` rather than installing it with `apt`.

**Reference:**
- [Caddy Official Site](https://caddyserver.com)

---

## 8. Reverse Proxy Servers

### What Is a Reverse Proxy?

A **reverse proxy** is a server that sits *in front of* one or more backend servers and forwards client requests to them. The client never communicates directly with the backend — it only talks to the reverse proxy.

### Why Does It Exist?

Imagine you have a web application with two components:
- A **frontend** — Static HTML/CSS/JavaScript files
- A **backend API** — A program that processes data and returns JSON

Without a reverse proxy, you'd have to expose both services directly to the internet on different ports (e.g., the frontend on port 80, the API on port 8080). Users would need to know both addresses.

A reverse proxy solves this by presenting a **single, unified entry point**:

```
Client request: GET /           → Reverse proxy serves the static HTML
Client request: POST /api/data  → Reverse proxy forwards to the backend API
```

The client sees one server. Behind the scenes, the reverse proxy routes requests to the appropriate backend based on the URL path.

### Reverse Proxy Benefits

- **Unified entry point** — One IP/domain for everything, regardless of how many backends exist
- **Security** — Backend servers are hidden from the internet; only the proxy is exposed
- **HTTPS termination** — The proxy handles encryption/decryption; backends communicate in plain HTTP internally (faster)
- **Load balancing** — A reverse proxy can distribute requests across multiple backend copies

### How Caddy Acts as a Reverse Proxy

In the course setup, the Caddyfile configures Caddy to:
1. Serve the static HTML site at the root path (`/`)
2. Forward any request to `/api` to a backend service running on `localhost`

This means Caddy is simultaneously a **web server** (serving static files) and a **reverse proxy** (forwarding API requests to a backend).

### What My Professor Didn't Explain

**"Reverse" vs. "Forward" proxy:** A **forward proxy** (what most people just call a "proxy") sits in front of *clients* and forwards their requests outward — like a VPN or corporate proxy. A **reverse proxy** sits in front of *servers* and forwards incoming requests inward. The "reverse" refers to the direction of the proxy relative to the client.

```
Forward Proxy:   Client → [Proxy] → Internet → Server
Reverse Proxy:   Client → Internet → [Proxy] → Backend Server
```

**`localhost`** in the course context: The Caddyfile forwards API requests to `localhost` because the API service is running on the **same machine** as Caddy. In production, you'd typically run the API on a separate server and point the reverse proxy to that server's internal IP address.

**Reference:**
- [Cloudflare — What is a reverse proxy?](https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/)

---

## 9. APIs — Application Programming Interfaces

### What Is an API?

An **API** (Application Programming Interface) is a set of rules and endpoints that allow two pieces of software to communicate with each other. In the context of web development, a **web API** is a service that accepts HTTP requests and returns structured data (usually JSON).

### A Concrete Example

The weather app on your phone does not generate weather data itself. It sends a request to a weather service's API:

```
GET https://api.weather.com/forecast?city=vancouver
```

The API responds with data:

```json
{
  "city": "Vancouver",
  "temperature": "12°C",
  "condition": "Partly cloudy"
}
```

The app then displays this data in a user-friendly format. The API is the bridge between the data source and the application that presents it.

### How APIs Work in This Course

The "API" you deploy in this exercise is a simple test service that:
1. **Echoes back** whatever data you send it (the request body)
2. **Logs** each request for debugging

It is not a "real" API in the sense that it doesn't process or store meaningful data — it exists purely for testing that the reverse proxy, load balancer, and firewall are working correctly.

You test it by sending an HTTP POST request with `curl`:

```bash
curl -X POST http://YOUR_SERVER_IP/api -d "hi server"
```

If everything is configured correctly, the API echoes back `hi server`.

### What My Professor Didn't Explain

**Why POST and not GET?** HTTP has several "methods" (also called "verbs") that indicate what kind of action you want:

| Method | Purpose | Has a Body? |
|---|---|---|
| `GET` | Retrieve data | No |
| `POST` | Send/create data | Yes |
| `PUT` | Update/replace data | Yes |
| `DELETE` | Remove data | Usually no |

The `-d "hi server"` flag in the `curl` command sends data in the **request body**. Since you're sending data *to* the server, `POST` is the correct method. A `GET` request doesn't have a body.

**Reference:**
- [AWS — What is an API?](https://aws.amazon.com/what-is/api/)

---

## 10. Load Balancers

### What Is a Load Balancer?

A **load balancer** is a service that distributes incoming network traffic across multiple servers. Instead of one server handling all requests, the load balancer spreads the work so that no single server gets overwhelmed.

### Why Does It Exist?

A single server has finite capacity — it can only handle so many requests per second before it slows down or crashes. Load balancers solve this in two ways:

1. **Scaling:** Add more servers behind the load balancer to handle more traffic
2. **Reliability:** If one server goes down, the load balancer automatically stops sending traffic to it and routes requests to the remaining healthy servers

### How It Works

```
                         ┌──────────────┐
                         │ Load Balancer│  ← Client sends request here
                         │ (Public IP)  │
                         └──────┬───────┘
                                │
                    ┌───────────┼───────────┐
                    │           │           │
              ┌─────▼─────┐ ┌──▼──────┐ ┌──▼──────┐
              │ Server A  │ │ Server B│ │ Server C│
              │ (web-01)  │ │ (web-02)│ │ (web-03)│
              └───────────┘ └─────────┘ └─────────┘
```

The client only knows the load balancer's IP address. It has no idea that multiple servers exist behind it. The load balancer picks a server (using algorithms like round-robin, least connections, or weighted distribution) and forwards the request.

### Load Balancers on DigitalOcean

DigitalOcean provides a **managed load balancer** — you don't install or configure load balancing software yourself. You create it in the dashboard, tell it which Droplets to balance across, and it handles everything.

A key feature is **tag-based targeting.** Instead of listing specific Droplet IPs, you assign a tag (like `web`) to your Droplets, and tell the load balancer: "balance across all Droplets with the `web` tag." This means:
- Adding a new server: Just create a Droplet with the `web` tag — it's automatically included
- Removing a server: Delete it — it's automatically excluded
- No load balancer reconfiguration needed

### Health Checks

DigitalOcean's load balancer automatically performs **health checks** — it periodically sends test requests to each server. If a server stops responding, the load balancer marks it as "unhealthy" and stops sending traffic to it. When the server recovers, traffic is automatically restored.

### What My Professor Didn't Explain

**Why not just get a bigger server?** This is called **vertical scaling** (more CPU, more RAM on one machine), and it has limits — eventually, the biggest available server still isn't enough. A load balancer enables **horizontal scaling** (more machines), which has no practical ceiling. Most modern web architecture is designed around horizontal scaling.

**The load balancer is a single point of failure.** If the load balancer itself goes down, nothing works. Cloud-managed load balancers (like DigitalOcean's) address this by being redundant internally — they run on multiple machines behind the scenes so that the service itself is highly available. This is another reason to use a managed service rather than running your own.

**Reference:**
- [DigitalOcean Load Balancers Docs](https://docs.digitalocean.com/products/networking/load-balancers/)
- [Cloudflare — What is load balancing?](https://www.cloudflare.com/learning/performance/what-is-load-balancing/)

---

## 11. Firewalls

### What Is a Firewall?

A **firewall** is a security system that monitors and controls incoming and outgoing network traffic based on a set of rules. It acts as a gatekeeper between your server (a trusted network) and the internet (an untrusted network).

### Why Does It Exist?

When you create a server on DigitalOcean, it has a public IP address — anyone on the internet can attempt to connect to it. Without a firewall:

- A web server on port 80 is accessible (good — that's the point)
- SSH on port 22 is accessible (necessary for administration, but also for attackers)
- Any other service you accidentally start is also accessible (very bad)

A firewall lets you define explicit rules: "Allow connections on port 80 (HTTP) and port 22 (SSH). Block everything else."

### Inbound vs. Outbound Rules

| Rule Type | What It Controls | Example |
|---|---|---|
| **Inbound** | Traffic coming *into* your server from the internet | "Allow HTTP on port 80" |
| **Outbound** | Traffic going *out of* your server to the internet | "Allow all outbound" (the default — your server can reach the internet) |

In most configurations, you tightly control **inbound** rules (only allow necessary ports) and leave **outbound** rules open (your server needs to download updates, connect to APIs, etc.).

### DigitalOcean Cloud Firewall

DigitalOcean provides a **cloud-level firewall** — it operates at the network level, outside your server. This means:

- Traffic is blocked before it reaches your Droplet (it never touches your server's resources)
- It applies to Droplets based on **tags** (just like the load balancer)
- No software to install or configure on the server itself

In the course exercise, you:
1. Create a firewall in the DigitalOcean dashboard
2. Add an inbound rule allowing **HTTP** (port 80)
3. The default SSH rule (port 22) is already included
4. Apply it to Droplets with the `web` tag

### What My Professor Didn't Explain

**Bastion host pattern:** In production, you typically would NOT allow SSH (port 22) from the entire internet to every server. Instead, you set up a single, hardened server called a **bastion host** (or "jump box"). SSH from the internet is only allowed to the bastion. From the bastion, you can SSH to your internal servers. This dramatically reduces the attack surface.

```
Internet → [Firewall: Allow SSH only to Bastion]
                    │
              ┌─────▼─────┐
              │  Bastion   │ ← Only server with public SSH access
              │   Host     │
              └──────┬─────┘
                     │ Internal network
           ┌─────────┼─────────┐
      ┌────▼───┐ ┌───▼────┐ ┌──▼─────┐
      │ Web 01 │ │ Web 02 │ │ DB 01  │  ← No public SSH
      └────────┘ └────────┘ └────────┘
```

**Cloud firewall vs. host firewall:** A cloud firewall (like DigitalOcean's) protects at the network level. Linux also has its own built-in firewall called **`iptables`** (or its modern front-end, `nftables`/`ufw`). In production, you often use *both* — the cloud firewall as the first line of defense, and a host-level firewall as defense-in-depth. For this course, the cloud firewall alone is sufficient.

**Reference:**
- [Cloudflare — What is a firewall?](https://www.cloudflare.com/learning/security/what-is-a-firewall/)
- [DigitalOcean Cloud Firewall Docs](https://docs.digitalocean.com/products/networking/firewalls/)

---

## 12. `curl` — Testing from the Command Line

### What Is `curl`?

**`curl`** (short for "Client for URLs") is a command-line tool for making HTTP requests. It supports a huge number of protocols, but you will primarily use it to test web servers and APIs.

Think of `curl` as a text-only web browser. Instead of rendering a web page visually, it shows you the raw response.

### Why Use `curl`?

- **Test a web server** without opening a browser
- **Test an API** by sending POST, PUT, DELETE requests (things a browser doesn't easily do)
- **Automate testing** in scripts
- **Debug** by seeing exact headers, status codes, and response bodies

### Basic Usage

#### Fetch a web page (GET request)

```bash
curl http://example.com
```

This sends a GET request and prints the HTML response to your terminal.

#### Send data to an API (POST request)

```bash
curl -X POST http://YOUR_SERVER_IP/api -d "hi server"
```

##### Command Breakdown

| Part | Purpose |
|---|---|
| `curl` | The curl command |
| `-X POST` | Use the HTTP POST method (instead of the default GET) |
| `http://YOUR_SERVER_IP/api` | The URL to send the request to |
| `-d "hi server"` | The **data** (request body) to send. The `-d` flag also automatically sets the Content-Type header to `application/x-www-form-urlencoded` |

### Useful `curl` Flags

| Flag | What It Does | Example |
|---|---|---|
| `-X METHOD` | Specify the HTTP method | `curl -X POST ...` |
| `-d "data"` | Send data in the request body | `curl -d "hello" ...` |
| `-H "header"` | Add a custom header | `curl -H "Content-Type: application/json" ...` |
| `-i` | Include response headers in the output | `curl -i http://example.com` |
| `-v` | Verbose mode — shows the full request and response | `curl -v http://example.com` |
| `-o file` | Save the response to a file | `curl -o page.html http://example.com` |
| `-s` | Silent mode — suppresses the progress bar | `curl -s http://example.com` |

### Practical Example: Testing the Course API

```bash
# Test from your LOCAL machine (not from the server)
curl -X POST http://143.198.246.74/api -d "hello from load balancer"
```

Expected output:
```
hello from load balancer
```

The API echoes back exactly what you sent. If you see this, your entire chain is working: client → load balancer → Caddy (reverse proxy) → API backend → response.

**Reference:**
- [curl Official Site](https://curl.se)
- [curl for Windows](https://curl.se/windows/)

---

## 13. Hands-On: Deploying a Web Server on DigitalOcean

This section walks through the full deployment from the professor's instructions, reorganized into a clear sequence with explanations of *why* each step exists.

### Architecture Overview

By the end, you will have:

```
                Internet
                   │
            ┌──────▼──────┐
            │   Firewall   │  ← Allows only HTTP (80) and SSH (22)
            └──────┬───────┘
                   │
            ┌──────▼──────┐
            │Load Balancer │  ← Distributes traffic to both servers
            │ (Public IP)  │
            └──────┬───────┘
                   │
          ┌────────┼────────┐
          │                 │
   ┌──────▼──────┐  ┌──────▼──────┐
   │  Server 01   │  │  Server 02   │
   │  - Caddy     │  │  - Caddy     │  ← Serves static site + reverse proxy
   │  - API       │  │  - API       │  ← Backend API
   └──────────────┘  └──────────────┘
```

### Phase 1: Create the Droplets

**Create two new DigitalOcean Droplets** with these settings:

| Setting | Value |
|---|---|
| **OS** | Debian 13 (stable) |
| **Region** | San Francisco — SFO2 or SFO3 |
| **Droplet Type** | Basic |
| **CPU** | Premium AMD, Premium Intel, or Regular SSD |
| **Size** | 1 GB RAM / 25 GB SSD / 1000 GB Transfer |
| **SSH Key** | The key you added in Week 1 |
| **Hostname** | `yourname-web` (e.g., `nathan-web`) |
| **Tags** | `web` |
| **Quantity** | 2 |

**Why Debian 13 (stable) instead of Debian 14 (testing)?** "Testing" is great for experimenting with new features. "Stable" is for production workloads — the packages are thoroughly tested, security updates are prioritized, and things don't break unexpectedly. Since you're building a web server, stability matters.

**Why 1 GB RAM?** The 512 MB option is underpowered for running both a web server (Caddy) and an API service on the same Droplet simultaneously.

**Why the `web` tag?** The load balancer and firewall will use this tag to automatically target these servers — no need to manually specify IPs.

**Why 2 servers?** A load balancer requires at least two servers to balance between. This is also the foundation of high availability — if one server fails, the other keeps serving.

### Phase 2: User Setup

SSH into each server and set up a regular (non-root) user:

```bash
ssh root@YOUR_DROPLET_IP
```

#### Step 1: Install Git and clone the setup scripts

```bash
apt update
apt upgrade
apt install git
```

#### Step 2: Clone and run the user setup scripts

```bash
git clone https://git.sr.ht/~nathan_climbs/2420-wk5-scripts
cd 2420-wk5-scripts
```

Make the scripts executable and run them:

```bash
chmod +x mkuser lck-root
```

- **`mkuser`** — Creates a regular user with a password and copies SSH keys. **Run this first.**
- **`lck-root`** — Disables root SSH login. Run this after confirming you can log in as your new user.

**Why disable root SSH?** The `root` account has unlimited power and is the first username attackers try. Disabling root SSH login and using a regular user with `sudo` adds a layer of security. If an attacker guesses a regular user's credentials, they still need to escalate to root.

### Phase 3: Clone the Starter Files

Log in as your new regular user and clone the starter repository:

```bash
git clone https://git.sr.ht/~nathan_climbs/web-server-starter-2420
```

This repository contains all the configuration files you need:
- `index.html` — The static web page
- `caddy` — The Caddy web server binary
- `Caddyfile` — Caddy's configuration file
- `caddy.service` — A systemd service file for Caddy
- `api` — The API binary
- `web-api.service` — A systemd service file for the API

**None of these files need editing.** You just need to copy them to the correct locations.

### Phase 4: Set Up Caddy (Web Server)

#### Step 1: Create a system user for Caddy

```bash
sudo useradd -rmd /home/caddy -s /usr/sbin/nologin caddy
```

##### Command Breakdown

| Flag | Purpose |
|---|---|
| `-r` | Create a **system account** (gets a low UID, won't appear on login screens). System accounts are for services, not humans |
| `-m` | Create the home directory if it doesn't exist |
| `-d /home/caddy` | Set the home directory to `/home/caddy`. Caddy stores internal files (like TLS certificates) here |
| `-s /usr/sbin/nologin` | Set the login shell to `nologin` — this user cannot log in interactively. This is a security measure: if an attacker somehow compromises the Caddy service, they cannot use the caddy user to get a shell |
| `caddy` | The username to create |

**Why a dedicated user?** This follows the **principle of least privilege**. Instead of running Caddy as `root` (which would give it unlimited access to the entire system), you run it as a dedicated user that only has access to the files it needs. If Caddy is compromised, the damage is contained.

#### Step 2: Create the document root directory

```bash
sudo mkdir -p /web/caddy
```

This is where your website's HTML files will live.

#### Step 3: Copy the static site

```bash
sudo cp ~/web-server-starter-2420/index.html /web/caddy/
```

#### Step 4: Set ownership

```bash
sudo chown -R caddy:caddy /web/caddy
```

This makes the `caddy` user and group the owners of the website directory. The Caddy process (running as the caddy user) needs to read these files.

| Part | Purpose |
|---|---|
| `chown` | **Change owner** — modifies the user and group ownership of files |
| `-R` | **Recursive** — apply to all files and subdirectories inside `/web/caddy` |
| `caddy:caddy` | Set both the owner and group to `caddy` |

#### Step 5: Install the Caddy binary

```bash
sudo cp ~/web-server-starter-2420/caddy /usr/local/bin/
```

**Why `/usr/local/bin/`?** In the Linux filesystem hierarchy:
- `/usr/bin/` — Programs installed by the system package manager
- `/usr/local/bin/` — Programs installed manually by the system administrator

Since you're installing Caddy manually (not via `apt`), it goes in `/usr/local/bin/`. Both directories are in the system's `$PATH`, so you can run `caddy` from anywhere.

#### Step 6: Set up the Caddy configuration

```bash
sudo mkdir -p /etc/caddy
sudo cp ~/web-server-starter-2420/Caddyfile /etc/caddy/
```

**Why `/etc/caddy/`?** The `/etc/` directory is the standard location for configuration files on Linux. Every major service stores its config there: `/etc/nginx/`, `/etc/ssh/`, `/etc/caddy/`.

The Caddyfile configures two endpoints:
- **`/`** (root) — Serves the static `index.html` from `/web/caddy`
- **`/api`** — Reverse proxies to the API backend running on `localhost`

The configuration also sets the server to listen on **port 80 (HTTP)**. In production with a real domain name, you would use the domain name here and Caddy would automatically set up HTTPS on port 443.

#### Step 7: Install and start the Caddy service

```bash
sudo cp ~/web-server-starter-2420/caddy.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl start caddy
sudo systemctl status caddy
```

| Command | Purpose |
|---|---|
| `cp ... /etc/systemd/system/` | Place the service file where systemd expects unit files |
| `systemctl daemon-reload` | Tell systemd to re-scan for new or changed unit files. **Required after adding or modifying any service file** |
| `systemctl start caddy` | Start the Caddy service immediately |
| `systemctl status caddy` | Verify it's running (look for `active (running)` in the output) |

#### Step 8: Test the static site

Open your browser and visit `http://YOUR_DROPLET_IP`. You should see your HTML page. Remember to use `http://` (not `https://`) since the server is configured for port 80.

#### Step 9: Enable Caddy to start at boot

```bash
sudo systemctl enable caddy
```

This creates a symlink so that Caddy starts automatically whenever the server reboots. Without `enable`, you'd have to manually `start` Caddy after every reboot.

### Phase 5: Set Up the API

#### Step 1: Install the API binary

```bash
sudo cp ~/web-server-starter-2420/api /usr/local/bin/
```

#### Step 2: Install and start the API service

```bash
sudo cp ~/web-server-starter-2420/web-api.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl start web-api
sudo systemctl status web-api
```

#### Step 3: Enable the API service

```bash
sudo systemctl enable web-api
```

#### Step 4: Test the API

Run this **on your local machine** (not on the server):

```bash
curl -X POST http://YOUR_SERVER_IP/api -d "hi server"
```

Expected response:
```
hi server
```

If you see the echoed text, the entire chain works: your request hit Caddy on port 80, Caddy recognized the `/api` path, forwarded it to the backend API via reverse proxy, and the API sent back the response.

**Repeat all of Phase 4 and Phase 5 on your second server.**

### Phase 6: Create the Load Balancer

1. In the DigitalOcean dashboard, click **Create** → **Load Balancer**
2. Set the region to **SFO3** (must match your Droplets)
3. Under "Connect Droplets," use the **`web` tag** (not individual Droplet names)
4. Leave all other settings at their defaults
5. Create the load balancer and wait for it to become active (may take a few minutes)

Once active, the dashboard shows:
- The load balancer's **public IP address** (this is what you'll share with users)
- The **health status** of each Droplet (should both show "healthy")

**Test everything again** using the load balancer's IP:

```bash
# Test the static site
curl http://LOAD_BALANCER_IP

# Test the API
curl -X POST http://LOAD_BALANCER_IP/api -d "hello from load balancer"
```

### Phase 7: Create the Firewall

1. In the DigitalOcean dashboard, click **Create** → **Firewall** (under Networking)
2. Give it a name (e.g., `web-firewall`)
3. **Inbound rules:**
   - SSH (port 22) — already included by default
   - Add: **HTTP** (port 80)
4. Under "Apply to Droplets," use the **`web` tag**
5. Create the firewall

After creating the firewall, test everything one more time with the load balancer IP to confirm traffic still flows correctly.

### Phase 8: Final Verification

All testing is done **from your local machine** using the **load balancer's IP address**:

```bash
# Test the static site in a browser
# Visit: http://LOAD_BALANCER_IP

# Test the API with curl
curl -X POST http://LOAD_BALANCER_IP/api -d "hi server"
```

Both should work exactly as they did when testing individual servers — but now traffic is going through the load balancer and firewall.

---

## 14. Key Takeaways and Study Tips

### SFTP — Quick Reference

| Task | Command |
|---|---|
| Connect | `sftp debian` |
| Upload file | `put filename` |
| Upload directory | `put -R dirname` |
| Download file | `get filename` |
| Download directory | `get -R dirname` |
| List remote files | `ls` |
| List local files | `lls` |
| Change remote directory | `cd path` |
| Change local directory | `lcd path` |
| Disconnect | `bye` or `exit` |

### Web Infrastructure — How the Pieces Fit Together

```
User's Browser
      │
      ▼
  Firewall ──────── Allows only HTTP (80) and SSH (22)
      │
      ▼
Load Balancer ───── Distributes requests across servers
      │
  ┌───┼───┐
  ▼       ▼
Server A  Server B
  │         │
  ▼         ▼
Caddy ──────────── Serves static files + reverse proxies /api
  │         │
  ▼         ▼
API  ───────────── Backend service (echoes request body)
```

### Key Concepts to Understand

| Concept | One-Sentence Summary |
|---|---|
| **Web server** | Software that listens for HTTP requests and serves files or responses |
| **Reverse proxy** | A middleman that forwards client requests to backend servers |
| **Load balancer** | Distributes traffic across multiple servers for performance and reliability |
| **Firewall** | Controls which network traffic is allowed in and out |
| **API** | A set of HTTP endpoints that software uses to exchange data |
| **`curl`** | A command-line tool for making HTTP requests |
| **`sftp`** | A secure file transfer tool that runs over SSH |
| **systemd service** | A background process managed by `systemctl` (start, stop, enable, status) |

### The `systemctl` Workflow for Any New Service

Every service you deploy follows the same pattern:

```bash
# 1. Copy the .service file to the systemd directory
sudo cp myapp.service /etc/systemd/system/

# 2. Tell systemd about the new file
sudo systemctl daemon-reload

# 3. Start the service
sudo systemctl start myapp

# 4. Verify it's running
sudo systemctl status myapp

# 5. Enable it to survive reboots
sudo systemctl enable myapp
```

This pattern applies to Caddy, the API, NFS mounts, and any future service you deploy.

---

> **References:**
> - [OpenBSD sftp man page](https://man.openbsd.org/sftp.1)
> - [Mozilla Developer Network — What is a web server?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_web_server)
> - [Nginx Official Site](https://nginx.org/en/)
> - [Caddy Official Site](https://caddyserver.com)
> - [Cloudflare — What is a reverse proxy?](https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/)
> - [AWS — What is an API?](https://aws.amazon.com/what-is/api/)
> - [DigitalOcean Load Balancers Docs](https://docs.digitalocean.com/products/networking/load-balancers/)
> - [Cloudflare — What is load balancing?](https://www.cloudflare.com/learning/performance/what-is-load-balancing/)
> - [Cloudflare — What is a firewall?](https://www.cloudflare.com/learning/security/what-is-a-firewall/)
> - [DigitalOcean Cloud Firewall Docs](https://docs.digitalocean.com/products/networking/firewalls/)
> - [curl Official Site](https://curl.se)
> - [curl for Windows](https://curl.se/windows/)
