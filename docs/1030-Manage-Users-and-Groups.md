# Manage Users and Groups

Currently, this chapter does not cover users from any LDAP or directory service.

## Why it matters
Linux is a multi-user operating system — and that’s both its strength and one of its biggest security risks. Every user, every group, and every permission defines who can access what. If this isn’t organized properly, you lose control fast. Poorly managed accounts are one of the most common entry points for privilege escalation, data leaks, and lateral movement inside a compromised system.

Managing users and groups is more than creating usernames and passwords. It means:

- separating human users from system or service accounts,
- grouping users consistently instead of granting permissions individually,
- assigning only the privileges required for each role (least privilege),
- and keeping the overall structure clean, predictable, and auditable.

If you skip this, you end up with a chaotic permission jungle, orphaned accounts, overly powerful users, and no clear ownership. That’s exactly the kind of environment attackers love.

This chapter focuses on locally managed accounts. We distinguish between:

- **Real users** — humans who log in, work, and need controlled access.
- **Technical users** — accounts without shell access, created only to run services, daemons, or applications.

Both types must be organized in groups. Groups are your security backbone: define permissions once and apply them to all members. Without groups, you end up assigning rights per user — unmanageable and unsafe on any multi-user system.

## How it works

Linux controls access through three elements: **users**, **groups**, and **permissions**. Every file, directory, and service on the system is owned by a user and by a group. Permissions define what the owner, the group, and everyone else may do.

The workflow is simple:

1. **Create users**  
   Real users get login shells. Technical users get a locked shell and no home directory.  
   This prevents services from acting like normal users.

2. **Assign users to groups**  
   Groups define roles: “admins”, “editors”, “backup”, “webservice”, etc.  
   A user inherits all permissions of all groups they are part of.

3. **Assign permissions on files and directories**  
   Permissions are always set on the filesystem — never directly on users.
   Define access by assigning the correct owning user and owning group.
   When a user is added to or removed from that group, their effective permissions update instantly.

4. **Use least privilege**  
   Each group should only have the exact rights it needs.  
   No write access for read-only roles. No execute rights where not required.

5. **Separate system accounts from human accounts**  
   Services like `www-data`, `mysql`, or `nginx` run under dedicated technical users.  
   They get:
   - no shell,  
   - no password,  
   - minimal filesystem permissions,  
   - and isolated groups.

6. **Keep the structure clean**  
   Use a consistent naming scheme, periodically review memberships, and remove unused accounts.  
   A clean permission model is easier to audit and much harder to abuse.

This approach keeps user management predictable, transparent, and resistant to privilege escalation.

### A note on file vs. group permissions

Linux does not assign permissions directly to users or groups.  
Permissions are always attached to **files and directories**, and each of them has:

- an owning user,
- an owning group,
- permission sets for user, group, and others.

A user gains access indirectly:
- by **owning** the file or directory, or
- by being a **member of the group that owns** the file or directory.

This means permissions are defined on the filesystem, while groups act as a role model that determines who inherits those permissions.

File and directory permissions are organized into three permission classes:
- **owner**  
- **group**  
- **others** (everyone who is neither the owner nor a member of the owning group)

As a security best practice, the **others** class should never receive meaningful permissions.  
Ideally, it is set to `0` (`---`) on all sensitive paths.

### Services and file permissions

Every service on a Linux system runs as a specific user and group.  
Even if a service is managed by systemd or another supervisor, it still interacts with the filesystem — reading configuration files, writing logs, accessing sockets, or storing data.

Because of this, service permissions are always tied to **file and directory ownership**.  
A service only has access to what the user and group assigned to it are allowed to access.  
This is why technical users such as `www-data`, `mysql`, or `nginx` exist: they isolate services and restrict them to the exact files they need.

### Best Practices

- **Avoid permissions for `others` on sensitive paths**  
  The `others` class should not have meaningful permissions on system or application data.  
  On security-relevant directories, `others` should ideally be set to `0` (`---`).

- **Separate human and technical accounts**  
  Real users get login shells. Service accounts do not.  
  Technical accounts should have:
  - no login shell (`/usr/sbin/nologin`),
  - no password,
  - minimal permissions.

- **Follow the least-privilege principle**  
  Users and services receive only the rights needed for their tasks — nothing more.

- **Use clear naming conventions**  
  Examples:  
  - Groups: `admins`, `developers`, `backup`, `webservice`  
  - System users: `nginx`, `mysql`, `backup-agent`

- **Review accounts regularly**  
  Remove unused users and stale group memberships.  
  Lock accounts instead of deleting if unsure.

- **Avoid shared accounts**  
  Each human must have an individual account for accountability and auditing.

- **Restrict shells and environments**  
  Assign restricted shells to users who do not need full system access.  
  Remove shell access from all service accounts.

- **Use password policies via PAM**  
  Enforce strong passwords and expiration rules for human users.

- **Document your permission model**  
  A simple table or README helps keep clarity when more admins join or the system grows.

- **Automate where possible**  
  Use configuration management (Ansible, Puppet, etc.) to keep user and group setups consistent across systems.

A disciplined user and group structure is the foundation of a secure Linux system. Without it, every other hardening step becomes unreliable.

### Common Mistakes

- **Using shared or generic accounts**  
  “admin”, “operator”, “support” without individual logins destroys accountability.  
  After an incident, you cannot tell who did what.

- **Letting service accounts have login shells**  
  A technical user with `/bin/bash` is a gift for attackers.  
  Service accounts should never log in interactively.

- **Keeping default accounts active**  
  Unused system accounts (games, lp, sync, etc.) increase attack surface.

- **Not reviewing accounts regularly**  
  Old employees, abandoned containers, test accounts — all leave doors open.

- **Giving users too many groups**  
  “Just add them to the admin group” is how privilege escalation starts.  
  Groups must match roles, not convenience.

- **Using weak or no password policies**  
  Especially dangerous on local systems that lack MFA.  
  PAM policies should always enforce strong rules.

- **Mixing real and technical users**  
  If you put a daemon into a human group or give a human access to a service group, you create unintended file access paths.

- **Leaving home directories world-readable**  
  Sensitive configuration files become exposed.  
  Home permissions should default to `700`.

- **Ignoring system logs**  
  Failed logins, group changes, and privilege escalations must be monitored.  
  If no one watches the logs, attackers stay invisible.

These mistakes are exactly what attackers exploit first — avoid them at all costs.

## Configuration Examples

### Create a real (human) user with a login shell
```bash
sudo useradd -m -s /bin/bash alice
sudo passwd alice
```

### Create a technical (service) user without login shell
```bash
sudo useradd -r -s /usr/sbin/nologin -M webservice
```

### Create a system user for a daemon (no password, no shell)
```bash
sudo useradd -r -s /usr/sbin/nologin -M --no-create-home backup-agent
```

### Create a group and assign users to it
```bash
sudo groupadd developers
sudo usermod -aG developers alice
```

### Create multiple groups for role-based access
```bash
sudo groupadd admins
sudo groupadd editors
sudo groupadd backup
```

### Assign users to multiple groups
```bash
sudo usermod -aG admins alice
sudo usermod -aG backup bob
```

### Check group membership
```bash
groups alice
```

### Set secure home directory permissions
```bash
chmod 700 /home/alice
```

### Enforce ownership for a project directory
```bash
sudo chown -R :developers /var/www/project
sudo chmod -R 770 /var/www/project
```

### Create shared directories with group ownership
```bash
sudo mkdir /srv/shared
sudo chgrp editors /srv/shared
sudo chmod 770 /srv/shared
```

### Disable shell access for an existing user
```bash
sudo usermod -s /usr/sbin/nologin bob
```

### Lock a user account (temporarily disable)
```bash
sudo usermod -L alice
```

### Unlock a user account
```bash
sudo usermod -U alice
```

### Expire a password immediately (forces reset at next login)
```bash
sudo passwd -e alice
```

### List all users including system accounts
```bash
getent passwd
```

### List all groups
```bash
getent group
```

### Find users with non-standard or unexpected shells
```bash
awk -F: '($7 !~ /(bash|sh|zsh|fish|nologin)/)' /etc/passwd
```

### Delete a user safely (keep home for review)
```bash
sudo userdel alice
```

### Delete user including home and mail spool
```bash
sudo userdel -r alice
```


## Commands

### User Management
```bash
# Create a human user with home and login shell
sudo useradd -m -s /bin/bash USERNAME

# Set or change password
sudo passwd USERNAME

# Create a technical/service user (no home, no login)
sudo useradd -r -s /usr/sbin/nologin -M SERVICEUSER

# Create a system user for daemons
sudo useradd -r -s /usr/sbin/nologin -M --no-create-home DAEMONUSER

# Delete user (keep home directory)
sudo userdel USERNAME

# Delete user and home directory
sudo userdel -r USERNAME

# Lock user account
sudo usermod -L USERNAME

# Unlock user account
sudo usermod -U USERNAME

# Force password change at next login
sudo passwd -e USERNAME

# Disable login shell for an existing user
sudo usermod -s /usr/sbin/nologin USERNAME

# Change login shell
sudo chsh -s /bin/bash USERNAME

# Create a group
sudo groupadd GROUPNAME

# Add user to group
sudo usermod -aG GROUPNAME USERNAME

# Remove user from a group
sudo gpasswd -d USERNAME GROUPNAME

# Change group ownership of a directory
sudo chgrp GROUPNAME /path/to/dir

# Set permissions for a group directory
sudo chmod 770 /path/to/dir

# Change owner of a file or directory
sudo chown USERNAME /path/to/file

# Change owner and group
sudo chown USERNAME:GROUPNAME /path/to/file

# Recursive ownership change
sudo chown -R USERNAME:GROUPNAME /path/to/dir

# Secure a home directory
chmod 700 /home/USERNAME

# List all users
getent passwd

# List all groups
getent group

# Show group membership of a user
groups USERNAME

# Show login activity
last

# Check lock status of a user
sudo passwd -S USERNAME

# Find users with non-standard or unexpected shells
awk -F: '($7 !~ /(bash|sh|zsh|fish|nologin)/)' /etc/passwd

# List system/service accounts (UID < 1000)
awk -F: '($3 < 1000) {print $1}' /etc/passwd

# Find users with empty passwords
sudo awk -F: '($2 == "") {print $1}' /etc/shadow

# Create a shared directory
sudo mkdir /srv/shared

# Assign group
sudo chgrp GROUPNAME /srv/shared

# Set secure permissions
sudo chmod 770 /srv/shared
```


## Final Checklist

Use this checklist to validate that your local user and group management is secure and properly hardened.

### Accounts

- [ ] Root login disabled (console + SSH)
- [ ] No shared or generic accounts exist
- [ ] All human users have individual accounts
- [ ] All technical/service users have:
  - [ ] no login shell (`/usr/sbin/nologin`)
  - [ ] no password
  - [ ] no home directory (unless required)
  - [ ] isolated groups

### Groups & Permissions

- [ ] Group-based permissions for shared resources instead of ad-hoc per-user tweaks
- [ ] Clear naming conventions for groups (e.g., `admins`, `backup`, `webservice`)
- [ ] Clear naming conventions for system users (e.g., `nginx`, `mysql`)
- [ ] Users are only members of groups required by their role
- [ ] No user is part of excessive or unnecessary groups
- [ ] All shared directories use group ownership and correct permissions

### Password & Authentication

- [ ] Strong password policies via PAM enforced
- [ ] No accounts with empty passwords
- [ ] Password expiration configured for human users
- [ ] 2FA/MFA enabled where possible (SSH, sudo sessions)

### Shell & Access

- [ ] No service user can log in interactively
- [ ] Restricted shells for users with limited access
- [ ] Home directories are not world-readable (`700`)
- [ ] No orphaned users or old employees left in the system
- [ ] Login shell of each user matches their purpose

### Monitoring & Auditing

- [ ] Regular review of `/etc/passwd`, `/etc/shadow`, `/etc/group`
- [ ] Logs of login attempts are monitored (`last`, `journalctl -u ssh`)
- [ ] sudo logs are reviewed regularly
- [ ] Account lockouts are logged and reviewed
- [ ] System automatically alerts on suspicious login activity

### Maintenance

- [ ] User and group definitions are documented
- [ ] Offboarding process removes or locks accounts immediately
- [ ] Automation tool (e.g., Ansible) maintains consistent setups
- [ ] Periodic audits scheduled (monthly or quarterly)

---

If all boxes are checked, your user and group management is clean, auditable, and significantly hardened against privilege escalation.



[Go Back to main menu](Readme.md)  
[Go Back to chapter menu](docs/1000-Basic-Security-Settings.md)
