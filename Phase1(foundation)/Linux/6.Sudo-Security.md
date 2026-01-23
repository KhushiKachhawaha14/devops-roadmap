# 🛡️ 1. Sudo Internals: How it works

The sudo (Substitute User DO) command allows a permitted user to execute a command as the superuser or another user, as specified by the security policy.

- **The /etc/sudoers File**: This is the "Brain" of sudo. It defines who can run what.

- **The visudo Command**: Never edit the sudoers file with a normal editor like nano or vim. Always use sudo visudo. It checks for syntax errors before saving. If you break the sudoers file, you might lock yourself out of the system.

- **The Sudo Cache**: By default, sudo "remembers" your password for 5 or 15 minutes so you don't have to type it every time.

# 🔑 2. Anatomy of a Sudoer Entry

When you look inside /etc/sudoers, you see lines like this: khushi ALL=(ALL:ALL) ALL

| Part          | Meaning        | Description                                                   |
| :------------ | :------------- | :------------------------------------------------------------ |
| **khushi**    | **User**       | The specific user account this rule applies to.               |
| **ALL (1st)** | **Hosts**      | Allows the user to run commands on any host (network server). |
| **(ALL:ALL)** | **User:Group** | The user can act as **any user** and/or **any group**.        |
| **ALL (2nd)** | **Commands**   | Allows the user to execute **any command** on the system.     |

# ⚠️ 3. Security Implications for DevOps

As a DevOps engineer, you often manage servers where developers need to restart services but shouldn't be able to delete the whole database.

### A. Privilege Escalation

If you give a user sudo access to a text editor (like vi or nano), they can "escape" the editor to a root shell.

- **Dangerous**: khushi ALL=(ALL) /usr/bin/vi (Khushi can now open /etc/shadow and change passwords).

### B. Passwordless Sudo

Often used for automation scripts (like Jenkins).

**Rule**: jenkins ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx

**Security Risk**: If the jenkins user is compromised, the attacker can restart Nginx, but they **cannot** do anything else as root.

# 🚩 Security Risks to Watch For

- **Sudo Shell Escapes**: Giving sudo access to editors (vim, less, more) or compilers can allow users to bypass restrictions and gain a full root shell.

- **NOPASSWD Abuse**: Only use NOPASSWD for specific, non-destructive commands required by automation.

- **The Sudo Group**: Adding a user to the sudo (Ubuntu) or wheel (RHEL) group gives them Full Root Access. Use this sparingly.

---

# ❓ Interview Questions

**Q: Why use visudo instead of nano /etc/sudoers?**

- A: visudo performs a syntax check. A single typo in sudoers can break the sudo system for all users, locking everyone out.

**Q: What is the difference between sudo -i and sudo -s?**

- A: sudo -i (login shell) provides the root user's environment, while sudo -s (shell) keeps your current user's environment.
