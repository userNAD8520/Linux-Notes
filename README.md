# Linux-Notes
Yo wassup. These are my notes for Nathan's Linux Class. I know his notes are not the best so these notes provide you with a better understanding of concepts. I took the material from Nathan's notes on GitLab and enhanced it with more examples and more explainations

## Week 1
Topics Covered:
- Kernel vs. OS: Technically, Linux is just the kernel (the core that talks to hardware). What we actually install are Distros (like Ubuntu or Fedora) which bundle the kernel with tools and a shell.
- The Linux Kernel: Created by Linus Torvalds in 1991, it handles the "heavy lifting" like managing your CPU, RAM, and storage.
- Anatomy of a Distro: A full system includes the kernel, a Package Manager (to install apps), an Init System (to boot up), and a Shell (like Bash).
- Open Source Freedom: Since the code is public, people "fork" (copy and modify) distros and software for everything from servers and cars to the Mars Rover.
- CLI focus: In this class, we interact with the system via the Command Line Interface (CLI) rather than a desktop/GUI.
- [Notes for topics above]()
--------------------------------------------------------------------
- SSH Basics: How to securely control remote servers and transfer files without using passwords.
- Key Setup (ssh-keygen): Generating your private key (stays on your PC) and public key (uploaded to the server).
- Config Shortcuts: Using the ~/.ssh/config file to create nicknames for servers so you can just type ssh debian instead of long IP strings.
- Permissions Matter: You must set your private key to 600 and your .ssh folder to 700 or the connection will be rejected for being "insecure."
- Troubleshooting: Common fixes for "Permission Denied" (usually wrong keys or permissions) and using -vvv to see exactly where a connection is failing.
- [Notes for topics above]()
