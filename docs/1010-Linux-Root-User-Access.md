# Avoiding Direct Root Login


## Why it matters

`root` is the most powerful account on any Linux or Unix system. It can modify everything — even when permissions should prevent it.  
This makes `root` the primary target in brute-force attacks, particularly on SSH. If you inspect your logs, you'll often see thousands of failed login attempts using the username `root`.

For that reason, **root should never be used for direct login**.

Instead, access is delegated to a normal user with controlled `sudo` privileges. This reduces attack surface and adds accountability. `sudo` allows a regular user to run specific administrative commands safely, without logging in as root. After completing these steps, direct root login will be disabled.

> Note: Root login via SSH must be handled separately. For that, refer to [1004-ssh-hardening](1004-ssh-hardening.md)

---

## How it works

1. **Create a regular user**  
   This user handles daily work and can elevate privileges through `sudo` when necessary.

2. **Add the user to the sudoers group**  
   On most Linux distributions `sudo` is installed by default.  
   On BSD systems it must be installed manually (not covered in this guide).

3. **Disable direct root login**  
   Once the sudo-enabled user is working, direct root login can be disabled for the console.  
   SSH hardening is covered in a separate chapter.

> **Important:** Never remove the root account.  
> System components rely on its existence.  
> The goal is to **prevent direct login**, not to delete the account.

---

## Configuration Examples

In this example, we create a new user named `mike`.

> **Note:** You are currently logged in as `root`.

### 1. Create a regular user
```bash
adduser mike
```

### 2. Add the user to the sudo group
```bash
usermod -aG sudo mike
```

### 3. Logout as root
```bash
exit
```

### 4. Login as mike

### 5. Lock the root account for direct login
```bash
sudo passwd -l root
```

### 6. Root login is now disabled  
The account still exists, but cannot be used to authenticate directly.

---

## Commands Overview

```bash
# as root
adduser mike
usermod -aG sudo mike
exit

# login as mike
sudo passwd -l root
```

---

## Final Checklist

- [ ] New user created  
- [ ] User added to sudo group  
- [ ] Root password locked  
- [ ] SSH configured to disallow root login (covered in the next chapter)  
- [ ] System tested: sudo works, root login blocked


[Go Back to main menu](../Readme.md)
[Go Back to chapter menu](1000-Basic-Security-Settings.md)
