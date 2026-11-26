# Ubuntu Cloud-init

## Why it matters
Cloud-init is a powerful tool developed by Canonical to automate the initial configuration and deployment of Linux servers. It allows administrators to deploy large numbers of servers with identical configurations and minimal manual intervention. Cloud-init supports multiple Linux distributions and can retrieve configuration data from different sources, including cloud providers, local configuration files and network services.

This is particularly useful when deploying and managing large server infrastructures. However, if this is not your deployment strategy, cloud-init might introduce an unnecessary configuration layer.

For example, if you install your servers manually and use Ansible to configure and manage them, you already have a powerful automation framework. In this scenario, cloud-init is not required. Unfortunately, cloud-init is included in standard Ubuntu Server installation images and may remain installed and enabled even if you never intend to use it. This can become a security issue.

Cloud-init does not necessarily stop modifying system configurations after the initial installation. Depending on its configuration, certain modules can execute during subsequent boots. During one of my security audits, I discovered that cloud-init had created an additional SSH configuration file without my knowledge. This configuration affected the effective SSH security settings and undermined my manually hardened SSH configuration.

An additional configuration management layer that operates outside your intended administrative control is something you should avoid. My recommendation: **If you do not use cloud-init, remove it completely from your system.**

## How it works
Cloud-init integrates into the Linux boot process and performs several initialization stages. During these stages, it retrieves configuration data from available data sources and applies the corresponding settings to the system.

Depending on the configuration, cloud-init can:

- Configure network interfaces.
- Create and configure user accounts.
- Install SSH public keys.
- Modify SSH server configurations.
- Install additional software packages.
- Execute scripts and commands.
- Configure storage and mount points.

Cloud-init uses different execution frequencies. Some modules **run only once per instance**, while others can execute during **every boot**. Therefore, the assumption that cloud-init only runs during the first system startup is incorrect. If you use Ansible or another configuration management solution to control your infrastructure, you should avoid having multiple independent systems modifying the same configuration files.

## Configuration Examples

### Deinstall the cloud-init pakage
If you do not need cloud-init, remove the package completely. Be careful and check before deinstallation you need it or not. Check also you have installed with cloud-init or not. If not and you are sure that no running configuration will be affected keep going in deinstall cloud-init

First, remove cloud-init and its package configuration:

To deinstall the cloud-init service you should deinstall `cloud-*` and afterwards use the autoremove option to deinstall all related packages from the system.

```bash
sudo apt purge cloud-init
```

Purge will remove also packet configurations while remove only removes the packages. In this case it will be good for use purge. The next step will be an cleanup. You will use:

```bash
sudo apt autoremove --purge
```

Finally, clean up the local package cache:

```bash
sudo apt clean
```

## Commands

```bash
# deinstall the cloud-* packages from the system
sudo apt purge cloud-init

# run autoremove and autoclean to deinstall all not more needed packages
sudo apt autoremove --purge

# cleanup the local package cache
sudo apt clean
```

## Final checklist

Use this checklist to secure the ssh agent on your systems.

### Creating key files
- [ ] Check whether cloud-init is installed.
- [ ] Remove cloud-init if it is not required.
- [ ] Remove unused dependencies.
- [ ] Review the remaining system services.
- [ ] Verify that services and  security settings remain intact.

[Go Back to main menu](Readme.md)
[Go Back to chapter menu](docs/1000-Basic-Security-Settings.md)
