# 🐧 The Linux Internalist: DevOps Filesystem Mastery
> **A Practical Guide to Inodes, IPC, and File Descriptors for System Reliability.**

In the world of DevOps, the filesystem isn't just a place to store code—it's the backbone of system performance, container communication, and infrastructure stability. This repository breaks down complex Linux internals into actionable DevOps patterns.

---

## 1. The 7 Linux File Types
In Linux, **"Everything is a file."** Identifying the file type is the first step in troubleshooting system resources. Use `ls -l` and inspect the **first character** of the permission string.

| Symbol | Type | DevOps Context |
| :---: | :--- | :--- |
| **`-`** | **Regular File** | Configuration files, binaries, application logs. |
| **`d`** | **Directory** | Folder structures, mount points, volume targets. |
| **`l`** | **Symbolic Link** | Versioning, Zero-downtime deployments. |
| **`p`** | **Named Pipe** | Inter-process communication (IPC) without Disk I/O overhead. |
| **`s`** | **Socket** | Network/Local communication (e.g., Docker or MySQL sockets). |
| **`c`** | **Character Device** | Stream devices (TTYs, `/dev/urandom`). |
| **`b`** | **Block Device** | Hardware storage (Hard drives, EBS volumes, partitions). |



---

## 2. Inodes: The Identity of the Filesystem
An **Inode** (Index Node) is a metadata structure on the disk. It stores everything about a file (permissions, owner, size, timestamps) **except** its name and the actual data content.

### 🛠️ DevOps Use Case: Inode Exhaustion
A disk can return a `No space left on device` error even if `df -h` shows plenty of GBs remaining. This usually means the system has run out of Inodes (common in CI/CD environments with millions of small temporary files).

**Key Troubleshooting Commands:**
* `df -i`: Check Inode usage percentages across all mounted filesystems.
* `ls -i`: View the specific Inode number assigned to a file.
* `stat <filename>`: View detailed metadata (Access, Modify, and Change times).

---

## 3. Links: Symbolic vs. Hard
Understanding links is crucial for managing application versions and optimizing storage snapshots.



### Comparison Table
| Feature | Symbolic Link (Soft) | Hard Link |
| :--- | :--- | :--- |
| **Points to** | The Filename/Path. | The Inode (Direct Data). |
| **Across Disks** | Yes. | No (Must stay on the same partition). |
| **If Original Deleted** | Link breaks ("Orphaned"). | Link stays valid (Data persists). |
| **Directories** | Allowed. | Generally not allowed. |

### 🛠️ DevOps Use Case: Zero-Downtime Deployment
To swap versions instantly without stopping the service, use an atomic symlink update:
```bash
# Force-create a symbolic link pointing to the new version
ln -sfn /opt/app_v2.1 /opt/app_current
```

## 4. Named Pipes (FIFOs) & File Descriptors
A Named Pipe (p) acts as a "tunnel." In DevOps, we use them for Log Shifting or Sidecar Communication. By using File Descriptors (FD), we keep the pipe "warm" to prevent the consumer process from exiting prematurely.

🛠️ Practical Task: The Bulletproof Log Filter
This setup ensures a persistent, real-time error log without restarting the filter process.

```bash
# Preparation: Create the pipe
mkfifo log-pipe

# 1. Open the pipe for writing using File Descriptor 3
# This prevents the pipe from closing between individual echo messages
exec 3> log-pipe

# 2. Start background grep (The Consumer)
# --line-buffered ensures logs appear instantly in the file instead of waiting for a buffer
grep --line-buffered "ERROR" < log-pipe >> system_logs &

# 3. Send messages using the descriptor (The Producer)
# Redirecting to >&3 sends data to the open file descriptor
echo "ERROR: Database Timeout" >&3
echo "INFO: User login" >&3
echo "ERROR: Disk Full on /dev/nvme0n1" >&3

# 4. Verify - The grep process remains running even after these echoes
cat system_logs

# 5. Cleanup: Close the descriptor to signal EOF to grep
exec 3>&-
rm log-pipe
```
## 5. Summary of DevOps Best Practices
- Monitor Inodes: Include df -i in Prometheus/Grafana alerts to prevent "Invisible" disk fullness.

- Buffer Management: Always use --line-buffered when piping logs to avoid lag in monitoring dashboards.

- The "Ghost File" Fix: Use lsof +L1 or lsof | grep deleted to find deleted files that are still consuming disk space because a process is holding them open.

- Descriptor Safety: Always explicitly close your File Descriptors (exec 3>&-) in scripts to avoid resource leaks

