# Hardening sudo and managing sudo users


## Why it matters

After disabling direct root login, all administrative actions must be performed through a sudo-enabled user. By default, every member of the `sudo` group receives unrestricted root-level access — powerful, but risky.  
Hardening sudo allows you to enforce least-privilege principles, define who may run which commands, and ensure that administrative actions follow a clear, auditable security policy.

---

## How it works

Sudo hardening starts with the `/etc/sudoers` file. This file defines exactly which users or groups may run which commands, under which conditions. The goal is to replace the default “everyone with sudo can do everything” model with a controlled, role-based structure.

Before editing anything, think about how your administrators should be organized:  
- Which users need full administrative access?  
- Which users only need specific commands, such as restarting services or managing logs?  
- Which tasks should always require a password or logging?  
- Which commands must never be allowed at all?  

Once these roles are clear, you define them in `/etc/sudoers` or create separate rule files under `/etc/sudoers.d/`. This allows you to assign precise permissions, enforce least-privilege policies, and keep administrative responsibilities clean and auditable.

---

## Configuration Examples

This is the default Ubuntu sudoers file. It demonstrates the standard permissions and highlights where custom rules can be added.

```bash
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

# This fixes CVE-2005-4890 and possibly breaks some versions of kdesu
# (#1011624, https://bugs.kde.org/show_bug.cgi?id=452532)
Defaults        use_pty

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "@include" directives:

@includedir /etc/sudoers.d
```

### Understanding the important lines

The key line is:

```
root    ALL=(ALL:ALL) ALL
```

This means:  
**root can run any command, on any host, as any user or any group.**

The lines for groups:

```
%admin ALL=(ALL) ALL
%sudo  ALL=(ALL:ALL) ALL
```

show that all users in the groups `admin` or `sudo` gain the same unrestricted privileges.  
This is simple — but dangerous in larger environments.

### Defining your own admin roles

You can create additional groups to build an administrative hierarchy.  
Example:

```
%webadmin ALL=(root) /usr/bin/systemctl restart apache2
```

This gives the group `webadmin` the **single permission** to restart the webserver — nothing else.

This is the core of sudo hardening:  
**granular permissions instead of full access for everyone.**

---

## Commands Overview

```bash
# Edit sudoers safely
visudo

# Create a new sudoers policy file
sudo visudo -f /etc/sudoers.d/webadmin

# Add a new admin group
groupadd webadmin

# Add a user to that group
usermod -aG webadmin mike
```

---

## Final Checklist

- [ ] Roles and privilege levels defined  
- [ ] `/etc/sudoers` not modified directly (use `/etc/sudoers.d/`)  
- [ ] Least-privilege rules applied  
- [ ] No users with unrestricted sudo unless absolutely necessary  
- [ ] Sudo logging enabled (Defaults use_pty)  
- [ ] Configuration validated with `visudo`  
- [ ] Users assigned to correct admin groups  

---

[Go Back to main menu](/Readme.md)  
[Go Back to chapter menu](docs/1000-Basic-Security-Settings.md)
