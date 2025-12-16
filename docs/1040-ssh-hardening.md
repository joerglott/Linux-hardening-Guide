# SSH hardening

## Why it matters
Most Linux servers are maintained via SSH. SSH (Secure Shell) is available on almost all Linux, Unix, and macOS systems and provides the ability to connect to remote systems and access their shell. For each connection from the outside, the system must expose an open port – in the default SSH configuration this is TCP port 22. Everyone in the world knows this port. Gaining access to that port and then into the remote system is the opportunity to become a system administrator – possibly even `root`. This is exactly what you need to avoid. Best practice is to make access to the system as secure as possible.

A common mistake is to simply move SSH to another port. While this can reduce noise from bots and automated scanners, it **does not provide real security**. Modern port scanners such as `nmap` can scan all open ports, fingerprint services, and gather information about a remote system. Therefore, port redirection must never be considered a security measure.

## How it works
The main task is to restrict several privileges on the SSH service itself. By default, the `sshd` service may allow password logins, public key logins, and even root logins. This is dangerous: earlier we explained that direct `root` login should not be possible, and SSH would otherwise allow it unless explicitly disabled.

Public key authentication is the most secure and recommended way to access a remote system. Each user who needs access receives a private and a public key. With this setup, only that user can authenticate to the system. In addition, **root login must always be disabled**.

SSH protocol version 1 is obsolete and insecure. Modern Linux distributions no longer support SSHv1 at all in their servers. Only some old SSH clients still implement the protocol so they can connect to legacy systems that have never been updated. New deployments must *never* use SSHv1.

The first step is to create SSH keys for passwordless login. Consider whether you want to protect the private key with a passphrase. In larger environments, administrators often avoid passphrases because they are prompted for the passphrase at every login. After generating the keys, you need to distribute the **public** key to the remote servers. Be extremely careful: only the **public** key must be copied to the server; the **private** key must remain on the client machine and be securely protected.

Next, adjust the SSH server configuration file — on Ubuntu this is `sshd_config` located in `/etc/ssh`. As `root`, edit the file and:

- disable root login (`PermitRootLogin no`),
- disable password authentication (`PasswordAuthentication no`),
- ensure SSH protocol 2 is enforced (`Protocol 2`).

After these changes, you must manage SSH keys carefully for all users who need remote access.

Be careful: misconfigurations in `sshd_config` can lock you out of the system. Always keep one active SSH session open when applying changes so you can recover if something goes wrong.

## Configuration Examples

### Generate an SSH key pair

There are different types of SSH key algorithms available.

The US **National Institute of Standards and Technology (NIST)** recommends using:

- **RSA** with at least 3072 bits  
- or **NIST ECDSA curves** (e.g. P-384) with at least 384 bits

These algorithms are widely used, standardized and often required in regulated environments.

At the same time, many administrators and cryptographers prefer **Ed25519**, which is based on the Curve25519 elliptic curve and uses the EdDSA signature scheme. Ed25519 is *not* a NIST curve – and that is one of the reasons some people explicitly like it.

After the Dual_EC_DRBG scandal (a NIST-recommended random number generator design that turned out to allow potential backdoors), trust in NIST recommendations, especially around elliptic curves, was damaged for some people. Ed25519 was designed outside this ecosystem with safer curve parameters and a simpler, more robust construction.

Key properties of Ed25519:

- Only **one fixed key size** (256 bits)  
- Security roughly comparable to **~3000-bit RSA**  
- Fast key generation and signing  
- Small keys and signatures  
- Uses curve parameters that are considered safer than traditional NIST ECDSA curves  
- Not supported by very old SSH clients, but well supported on all modern systems

Because of this, Ed25519 is a strong default choice for **modern environments**, while RSA with 3072 or 4096 bits remains useful for **legacy compatibility** and compliance constraints.

```bash
# RSA key (NIST-style, good for compatibility)
ssh-keygen -t rsa -b 3072

# Ed25519 key (modern, fast, non-NIST curve)
ssh-keygen -t ed25519
```

You will receive two files
- id_rsa
- id_rsa.pub

for rsa keys or
- id_ecdsa
- id_ecdsa.pub

for ECDSA keys. The `.pub` file is the public file you can transfer to the remote server. The other file is the privat key and you need to keep it private. Do not share that file.

### Transferring the public key to the remote system
For secure environments, you should avoid relying on `ssh-agent` and `ssh-add`. While they make passphrase-protected keys easier to use, they also keep unlocked keys in memory,  
which increases the attack surface if the local system or session is compromised.  

For most administrative use cases, transferring the public key manually with `scp` or via `ssh-copy-id` is sufficient and provides better control over which keys are used.

I am recommend the usage of `scp` to copy the public key to the remote server. Or - in case of having direct access to the system - use an USB Stick or something else to transfer the key file safely.

```bash
scp id_rsa.pub remoteuser@[ip-address or dns-name-of-system]:/home/[username]/
```

With this you are transfering safely the public key to the server. At the end we would not save that key file here. We should transfer it to a safe directory and changing the file permissions accoringly.

### Configuring the sshd service on the remote system
To make the ssh service safe you should edit the `/etc/ssh/sshd_config` file. First of all we need to adjust some lines. Search in the standard sshd config file these lines:

First of all - changing the port number only is not security enough. At least with tools like nmap you can find out the real port number on which the ssh agent is listening. For that - in my eyes - it does not matter on which port the ssh daemon is listening - standard is 22. In case of feeling better you can remove the `#` from the line `#Port 22` and change the port to any portnumber you prefer.

```bash
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
```
The next configuration option matters in case you are using different networks - for example a productive network and a admin network. In this case you should limit the ssh access to the admin network - that means the networt in which are the admins of the system are located. That delivers a small piece more security in your environment. But when you have only one network card in your server you can leave that setting as it is - because you are not able to align the ssh access to a specifi network because you do not have one.

In case of having some more network cards you can define the IPv4 and IPv6 address of the card, on which you will serve the ssh agent. Also you can define with `AddressFamily` using only IPv4, IPv6 or both (any).

```bash
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
```
In the Authentication section you should prohibit root to login via ssh. For that find the line `PermitRootLogin` and set the value to `no`. With that option root would not be able to login via ssh. This setting is mandantory.

```bash
# Authentication:
PermitRootLogin no
```
The next you should configure is the public key authentication for ssh. For that you are needing the public key we had transfered via `scp` to the server (or USB stick). The line `PubkeyAuthentication` is written as a comment but this option is standard activated. I am removing the `#` always only to know this option is activated.

```bash
PubkeyAuthentication yes
```

The next line you should find is the `AuthorizedKeysFile` line. Standard is refering to the local path `.ssh/authorized_keys` or `.ssh/authorized_keys2` file. But in case you are managing different users you might want to have a bit more control over the keyfiles itself. Using the directories above, the user do have full permissions to the key file. It is not bad, but ot secure enough at all. I recommend to create a extra directory - for example `/etc/ssh/authorized_keys/` - and moving the key files in that directory. To be able to manage different users the key file should name like the user which should be able to access via ssh - for example `mike`.

Before make these changes you should check the ACL package is installed. With this you can add extra file permissions to make the key file be readable for only the ssh user who should be able to read the key file. But the key file itself belongs to root. root do have the right to manage that file. In the sshd_config file add this line.

```bash
AuthorizedKeysFile      /etc/ssh/authorized-keys/%u
```

Now move the key file from the directory where you stored the key file on this server to the specific directory - for example `/etc/ssh/authorized_keys/`. Rename the key file to the name of the user - for example `mike`. Now the line `/etc/ssh/authorized_keys/%u` will call `/etc/ssh/authorized_keys/mike` in case of logging in `mike` via ssh.

But be sure, to set the right permissions to that file. The key file should own root. For that use `chown root: key-file-name` and set the permissions with `chmod` to 400 - that means only root can read the file. It is important that this files does not have too much permissions. You are using `chmod 400 key-file-name` for changing that. Now the user `mike` does not have the permission to read his own key file. Now we are using the `setfacl` command to change this issue. With this we are delegating the right `read` to the user `mike` to make the key file readble for `mike`. For the following command we are in the same directory where the key file is located - for example `/etc/ssh/authorized_keys/`.

```bash
sudo setfacl -m u:mike:r mike
```

This command allows the user `mike` to have readaccess to the key file `mike` which owner is `root`. In the command above the first `mike` in the line will be the file, the second `mike` is the user.

One of the last settings you should manage is disabling the `PasswordAuthentication`. Set this line to `no`. In this case the `PermitEmptyPasswords` setting does not make sense because we do not want using passwords.

```bash
# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
#PermitEmptyPasswords no
```
Now be careful, because if you activate that settings now it can happens that you are locked out from your own system. Be aware to have a console access in that case. The last step you need to do is restarting the ssh service on the server.

```bash
sudo service ssh restart
```
Now the settings are active and your ssh service has more security as before. But you can do more. This will be topic of a alter version.


## Commands

```bash
# transfer the public key file to the server
scp <keyfile>.pub <username>@<ip-or-dns-name>:/<directory-to-store-the-keyfile>

# create a directory to store all public keys for all users
mkdir /etc/ssh/authorized_keys

# move the publickeyfile to that directory and rename it to username (without .pub)
mv /<directory-to-store-the-keyfile>/<keyfile>.pub /etc/ssh/authorized_keys/<username>

# set the ownership and permissions on that file
chown root: /etc/ssh/authorized_keys/<username>
chmod 400 /etc/ssh/authorized_keys/<username>

# edit the ssh config file to make the changes we discussed above
vi /etc/ssh/sshd_config

# save the file, exit the editor

sudo service ssh restart
```


## Final checklist

Use this checklist to secure the ssh agent on your systems.

### Creating key files
- [ ] Creating a key pair as RSA or ED25519
- [ ] Using minimum 3072 bits for a RSA key
- [ ] Transfer securely the public key file to the server
- [ ] Store the private key safe and limit the access (physical and logical)
- [ ] Install the ACL package on the server
- [ ] Creating a key file directory for example `/etc/ssh/authorized_keys/`
- [ ] Move the public key file in this directory and rename it to the name of the user (for example `mike`)
- [ ] Adjust the ownership of that file to `root` and group as well `root`
- [ ] Adjust the permission of that file to `400`
- [ ] Set ACL permissions with `setfacl -m u:<keyfilename>:r <username=keyfilename>`

### Configure the ssh service on the server
- [ ] Open `/etc/ssh/sshd_config` file in a editor of your choice
- [ ] Disallow root login
- [ ] in case of having more network cards specify the network where the agent has to listen
- [ ] Set `PublicKeyAuthentication` on (remove the comment # to be sure it is active (standard))
- [ ] Adjust the path of key files (for example `/etc/ssh/authorized_keys/%u` )
- [ ] Disallow `PasswordAuthentication` for ssh


[Go Back to main menu](Readme.md)
[Go Back to chapter menu](docs/1000-Basic-Security-Settings.md)
