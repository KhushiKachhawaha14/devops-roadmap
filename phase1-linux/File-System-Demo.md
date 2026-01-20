# 🐧 Linux Hands-on: New Server Setup Task


> **This guide documents the essential first steps a DevOps Engineer takes when provisioning a new Linux server. It covers Performance Tuning, Custom Tooling, and System Observability.**

## 🛠️ The Mission
As a DevOps Engineer, your goal is to prepare a fresh server for production by completing three core objectives: Tune it, Tool it, and Secure it.

#### 1. Tuning the "Engine" (Kernel Limits)
Directory: /proc

Concept: The /proc directory is a virtual filesystem that acts as an interface to the Linux Kernel. It exists in memory (RAM) rather than on the physical disk.

Goal: Check the maximum number of concurrent files the server can handle.

Command:

```bash

cat /proc/sys/fs/file-max
```
DevOps Context: High-traffic applications (like Nginx or Databases) require many open file descriptors. If the request volume exceeds this kernel limit, the application will crash. We check this to ensure our infrastructure can scale.

#### 2. Adding a "Power Tool" (Global Automation)
Directory: /usr/local/bin

Concept: While /bin holds standard system commands, /usr/local/bin is the designated home for custom, user-created scripts that need to be accessible globally.

Goal: Create a custom shortcut command called space to monitor disk health instantly.

Execution:

```bash

# 1. Create the script logic
echo "df -h" > space

# 2. Grant executable permissions
chmod +x space

# 3. Move to the global binary path
sudo mv space /usr/local/bin/
```
The Result: You can now type space from any directory in the terminal to see disk utilization. This demonstrates how we standardize tools across an entire fleet of servers.

#### 3. Investigating the "Paper Trail" (Log Analysis)
Directory: /var/log

Concept: The /var directory contains "Variable" data. /var/log is the "Black Box" of the server, recording every system event and error.

Goal: Monitor real-time authentication attempts to verify server access.

Command:

```bash

# For security and login audits
tail -f /var/log/auth.log

# Note: If auth.log is empty, use the general system log:
tail -f /var/log/syslog
```
DevOps Context: When a deployment fails or a developer cannot connect via SSH, this is the first place we look to diagnose the "Why."

## 💡 Quick Summary Cheat Sheet

| Room (Folder) | Use Case | DevOps Context |
| :--- | :--- | :--- |
| **`/etc`** | The "Settings" folder | Where you configure Nginx, Docker, or system services. |
| **`/var/log`** | The "History" folder | The primary location for troubleshooting and log analysis. |
| **`/usr/local/bin`** | The "Toolbox" | Where custom automation and third-party scripts are stored. |
| **`/proc`** | The "Dashboard" | A virtual interface to check live Kernel, CPU, and RAM stats. |
