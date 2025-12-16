# Linux Hardening Guide
A modern, practical, and security-focused hardening guide for Linux systems in real-world environments.  
This project provides clear, actionable steps to secure Linux servers, desktops, virtual machines, and containerized workloads — with a strong focus on digital sovereignty, reliability, and long-term maintainability.

This guide is created for system administrators, DevOps engineers, security architects, and anyone who wants to build secure, autonomous, and resilient open-source infrastructure.

## Real-World Experience

This guide is built entirely on **practical, hands-on experience** from designing and operating secure open-source infrastructures, hosting environments, SME platforms, and automation-driven systems.  
Every recommendation has been tested, validated, and refined in real deployments.

## Living Document

This guide will continue to evolve over time.  
New chapters and updates will be added incrementally, based on real security needs, emerging threats, modern infrastructure requirements, and available time to contribute.

The goal is to keep this guide **relevant, practical, and continuously improving**.

## Scope & Philosophy

The focus of this guide is to provide **modern, maintainable, automation-friendly, and sovereignty-oriented** security practices for today’s infrastructure — without outdated theory or unnecessary complexity.

It covers widely used enterprise and server-grade Linux distributions:
- **Ubuntu LTS**
- **Debian Stable**
- **AlmaLinux / Rocky Linux / Red Hat-compatible systems**
- **Container and virtualization environments (LXC, KVM/QEMU)**

Obsolete systems, fringe distros, and theoretical "security theatre" are intentionally excluded.

## What This Guide Covers (and What It Does Not)

This guide is a working document and will increase constantly. At the moment I am working along at this so I will add new chapters as far as I do have time. Above you will find topics I want to cover or it is being covered in dependent of the current working state.

### ✔ Included
- Basic Linux security
- SSH security and access control  
- User management and sudo  
- systemd sandboxing  
- Firewalling (nftables, UFW)  
- Kernel hardening (sysctl)  
- Logging and auditing  
- Hardening common services (web, database, PHP-FPM)  
- Virtual machine & container security (KVM, LXC/Incus)  
- Automation patterns for enforcing security  
- Practical guidance for SMEs and self-hosted environments  

### ❌ Not included
- Academic theory  
- Outdated distributions  
- Kernel compilation  
- Installation basics (partitioning, encryption setup, bootloader design)  
- Exotic tools or fringe distros  

## Guide Structure

All detailed content is stored in the `docs/` directory as standalone chapters.  
You can read them in sequence or jump directly to the topics relevant to your setup.

Each chapter follows a consistent pattern:
- **Why it matters**  
- **How it works**  
- **Configuration examples**  
- **Commands**  
- **A final checklist**  

This structure ensures clarity, consistency, and long-term maintainability.

## Project Origin

Created and maintained by **Joerg Lott** — computer scientist, open-source architect, and long-time Linux user who runs his own global self-hosted cloud across 44 datacenters.  
This guide reflects a practical, sovereignty-driven approach: no cloud is better than the one you control yourself.

# Content

[Basic Security Settings](docs/1000-Basic-Security-Settings.md)
