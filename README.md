# Linux-Notes
Yo wassup. These are my notes for Nathan's Linux Class. I know his notes are ahh and have little to no examples with barely any explanations so these notes provide you with a better understanding of concepts. I took the material from Nathan's notes on GitLab and enhanced it with more examples and more explainations

## Week 1
Topics Covered:
- SSH Basics: How to securely control remote servers and transfer files without using passwords.
- Key Setup (ssh-keygen): Generating your private key (stays on your PC) and public key (uploaded to the server).
- Config Shortcuts: Using the ~/.ssh/config file to create nicknames for servers so you can just type ssh debian instead of long IP strings.
- Permissions Matter: You must set your private key to 600 and your .ssh folder to 700 or the connection will be rejected for being "insecure."
- Troubleshooting: Common fixes for "Permission Denied" (usually wrong keys or permissions) and using -vvv to see exactly where a connection is failing.

[Notes]()
