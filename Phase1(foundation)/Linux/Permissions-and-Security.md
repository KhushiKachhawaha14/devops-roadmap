# 👥 1. Users & Groups: The "Who"

Linux is a multi-user system. Every process (like Nginx or Jenkins) runs as a specific user.

- Root User (UID 0): The "God Mode" account. Avoid using this for daily tasks.

- System Users: Accounts like www-data (for web servers) or postgres (for databases).

- Groups: A way to grant the same permissions to multiple users (e.g., a devops group for all engineers).

### 🛠️ Hands-on: Creating a DevOps Team

```bash

# 1. Create a new group for the team
sudo groupadd devops

# 2. Add a user to this group
sudo usermod -aG devops khushi

# 3. Verify group membership
groups khushi
```

# 🔐 2. Permissions: The "What"

When you run ls -l, you see a string like -rwxr-xr--
| Symbol | Permission | Numeric Value |
| :---: | :--- | :---: |
| **r** | Read | 4 |
| **w** | Write | 2 |
| **x** | Execute | 1 |
| **-** | No Permission | 0 |

**The Triad**: User | Group | Others Example: 755 (rwxr-xr-x) means the User can do everything (7), but the Group and Others can only read and execute (5).

### 🛠️ DevOps Use Case: Securing a Private Key

If you store an SSH key with 777 permissions, SSH will refuse to work because it’s "too open."

```bash
# Set permissions so ONLY the owner can read it
chmod 600 id_rsa
```

# 🎭 3. Chown vs. Chmod

- chmod (Change Mode): Changes what can be done (Read/Write/Exec).

- chown (Change Owner): Changes who owns the file.

### DevOps Use Case:

You deploy code as the jenkins user, but the nginx user needs to serve it.

```bash
# Change ownership to the web server user
sudo chown -R www-data:www-data /var/www/html
```

# 🌑 4. Umask: The "Default Filter"

**Umask** (User Mask) defines the default permissions for newly created files and folders. Think of it as a "subtraction" from the maximum possible permissions.

- Max for Files: 666

- Max for Folders: 777

**The Calculation**:If your umask is 022:

- New Folder: $777 - 022 = 755$ (rwxr-xr-x)
- New File: $666 - 022 = 644$ (rw-r--r--)

**DevOps Tip**: In highly secure environments (like banking), we set a strict umask of 077. This ensures that any file created is invisible to everyone except the owner ($666 - 077 = 600$).

### 🚀 DevOps Summary

- chmod 400: For SSH Keys.

- chown www-data: For Web Assets.

- umask 022: For Standard Deployments.

## 🏗️ Real-Life DevOps Use Cases

## 1.The "Jenkins/GitHub Actions" Pipeline
When a CI/CD tool like Jenkins tries to deploy code to a server, it runs as a specific user (usually jenkins).

- The Conflict: If your web folder (/var/www/html) is owned by root, Jenkins won't be able to copy the new code there.
- The Fix: You use chown -R jenkins:www-data /var/www/html so the automation tool can write the files, and the web server group can read them.

## 2. Protecting Sensitive Secrets (SSH Keys & Configs)

DevOps engineers handle private keys (id_rsa) and .env files containing database passwords.

- The Risk: If these files have 777 permissions, any user on that server (or a compromised application) can steal your credentials.
- The Fix: We apply chmod 600 (Read/Write for owner only) to ensure only the authorized process can see the secrets.

## 3. Container Security (Docker & Kubernetes)

By default, many Docker containers run as the root user. This is a massive security risk.

- The DevOps Approach: We use permissions to ensure the container runs as a Non-Root User. We set the umask in the Dockerfile to ensure any files the container creates (like logs) aren't world-writable.

---

# ❓ Important Interview Questions (DevOps Edition)

If you are interviewing for a Junior DevOps role, expect these questions:

### Q1: What happens if you set an SSH Private Key to 777 permissions?

Answer: The SSH client will actually refuse to connect. It will give a warning saying "Permissions are too open" because it’s a security risk. You must use 600 or 400.

### Q2: How do you allow two different users to edit the same folder without giving them Root access?

Answer: You create a Group (e.g., developers), add both users to that group, and set the folder permissions to 775 or 770 so the group has "Write" access.

### Q3: A developer says their script isn't running even though they can see it. What is the first thing you check?

Answer: Check the Execute (x) bit. Even if they can read (r) the file, they cannot run it unless they have execute permissions (e.g., chmod +x script.sh).

### Q4: If the umask is 027, what are the default permissions for a new directory?

Answer: \* Max Directory Permission is 777.$777 - 027 = 750$.Result: User has full access (7), Group can Read/Execute (5), Others have nothing (0).
