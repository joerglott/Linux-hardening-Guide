# Basic Security Settings

This chapter outlines the essential security steps every fresh Linux installation must receive — no excuses. These measures apply to both physical and virtual systems and form the baseline for a hardened, dependable setup.

## Basic Security Principles

Before diving in, let’s quickly clarify the core security mindset.  
Every system has a purpose — and you only install what is required to fulfill that purpose.  

Example:  
You deploy a database server. For administration, you need SSH.  
So, this server should run **exactly two services**: the SSH daemon and the database service.  
Nothing else.  

Tools like phpMyAdmin only increase your attack surface and should be strictly avoided.  
The same applies to user management:  
Only users with a legitimate business need should gain access to the system. This includes evaluating whether remote access is required at all. For example, a database administrator should **never** manage the database by connecting remotely to its server shell unless absolutely necessary.

Keep your systems updated.  
An unpatched machine gets weaker over time and becomes an easy target. Updates won’t fix every vulnerability immediately, but they close known issues that attackers can and will exploit. Regular patching is one of the most important security fundamentals — regardless of whether you run Linux or something else.

And finally:  
Be extremely cautious with administrative privileges.  
Carefully decide who truly needs elevated rights. Especially in environments with IT operations, business departments, and application owners, you must evaluate who really requires admin-level access. And if an admin account is unavoidable, define its permissions with precision. Least privilege is the rule — not the exception.

These basics form the foundation for securing any system.

## Content

This chapter currently covers:

- [Linux Root User Access](1010-Linux-Root-User-Access.md)  
- [sudo Hardening](1020-sudo-Hardening.md)
- [Manage Users and Groups](1030-Manage-Users-and-Groups.md)
- [ssh Hardening](1040-ssh-hardening.md)


[Back to main page](../Readme.md)
