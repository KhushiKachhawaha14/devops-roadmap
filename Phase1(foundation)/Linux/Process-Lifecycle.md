# 🏗️ 1. The "Fork & Exec" Idiom: How Processes are Born

In Linux, a new process is almost never created from scratch. Instead, it is **cloned** and then **transformed**.

**The Two-Step Birth**:

**1 fork() (The Clone)**: An existing process **(the Parent)** creates an exact duplicate of itself called the Child. They share the same code and open files at first, but have different PIDs (Process IDs).

**2 exec() (The Transformation)**: The Child process calls exec() to wipe its current memory and load a **new program** (like nginx or python)

## Why this matters for DevOps:

- **The Shell**: When you type ls in Bash, Bash **forks** a child and that child **execs** the ls binary.

- **Web Servers**: Gunicorn or Nginx use a "Master-Worker" model. The Master process **forks** multiple Workers. If a worker crashes, the Master knows exactly which child to replace.

# 📡 2. Signals: The "Remote Control" for Processes

Signals are small notifications sent to a process to tell it to do something. As a DevOps engineer, you use these daily to manage services.

### 🚀 DevOps Signal Reference Table

| Signal      | Name      | Value | DevOps Use Case                                              | Exit Code | Catchable? |
| :---------- | :-------- | :---: | :----------------------------------------------------------- | :-------: | :--------: |
| **SIGHUP**  | Hangup    |  `1`  | **Reload Configs:** Update Nginx/Prometheus without restart. |   `129`   |    Yes     |
| **SIGINT**  | Interrupt |  `2`  | **Manual Stop:** Triggered by `Ctrl+C` in a terminal.        |   `130`   |    Yes     |
| **SIGKILL** | Kill      |  `9`  | **The Hammer:** Forced kill. Use for OOM or stuck pods.      |   `137`   |   **No**   |
| **SIGTERM** | Terminate | `15`  | **Graceful Shutdown:** Kubernetes default for stopping pods. |   `143`   |    Yes     |
| **SIGQUIT** | Quit      |  `3`  | **Troubleshoot:** Stops app and creates a Core Dump.         |   `131`   |    Yes     |

> **Note:** Exit codes are typically calculated as `128 + Signal Number`. If you see **Exit Code 137** in your logs, your container was likely killed by the system (OOM) or a timeout.

# 🧟 3. Zombies & Orphans: The "Glitches"

**Zombie Processes (Z state)**
A process that has finished execution but still has an entry in the process table.

- **The Cause**: The Child died, but the Parent is "busy" and hasn't acknowledged it (called "reaping").

- **The Fix**: You can't kill -9 a zombie (it's already dead!). You must kill or restart the Parent process.

### Orphan Processes

A child whose parent died unexpectedly.

**The Hero: PID 1** (systemd or init) automatically adopts orphans and cleans them up.
