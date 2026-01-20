# 🐧 Linux Filesystem Hierarchy for DevOps
In DevOps, the Filesystem Hierarchy Standard (FHS) is the "map" of your infrastructure. Knowing these directories is essential for effective automation, troubleshooting, and deployment.

## 🛠️ Core Directories & DevOps Use Cases

### 1. /etc — The Configuration Hub
   The brain of the system, containing all system-wide configuration files.

Why: You will manage service configurations for tools like Nginx, Docker, or SSH.

How: Managed via Infrastructure as Code (IaC) tools like Ansible, Chef, or Puppet to ensure consistency.

💡 DevOps Tip: Never edit files here manually in production. Always automate changes to maintain a "Single Source of Truth."

### 2. /var — Variable Data (Logs & State)
   Contains data that changes frequently while the system is running.

/var/log: The primary location for system and application logs (e.g., syslog, auth.log).

/var/lib/docker: Where Docker stores its containers, images, and volumes.

How: Set up Log Rotation and use log-shipping agents (Fluentd, Promtail) to send data to ELK or Grafana Loki.

### 3. /usr/local/bin — Custom Binaries & Tools
   The preferred location for user-installed executable programs.

Why: Used to install custom CLI tools like kubectl, terraform, or helm without interfering with system-managed packages.

How: Always ensure this directory is included in your system $PATH variable.

### 4. /home & /root — User Environments
   Personal directories for users and the system administrator.

Why: Essential for managing SSH access.

How: Automate the placement of .ssh/authorized_keys for deployment users (e.g., jenkins or github-actions).

💡 DevOps Tip: Avoid working in /root directly. Use dedicated service accounts with specific sudo privileges.

### 5. /tmp & /run — Ephemeral Data
   Used for temporary files and process IDs (PIDs).

Why: Useful for storing temporary build artifacts during CI/CD jobs.

⚠️ Warning: Files in /tmp are often deleted on reboot. Never store persistent data here.

## 6. /proc & /sys — The Kernel Window
   "Virtual" filesystems that provide a direct interface to the Linux kernel.

Why: Critical for performance tuning and deep container debugging.

How: Use sysctl to tweak kernel parameters (e.g., network limits, memory management) which interact with /proc/sys.
(sysctl is the tool used to view and modify kernel parameters at runtime)
