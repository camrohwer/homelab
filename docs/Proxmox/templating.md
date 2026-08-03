# Proxmox Debian VM Template

I wanted a clean Debian base template that I can use whenever I need to create a new VM in my homelab.

Instead of installing Debian from scratch every time, I can clone this template and start with a known-good minimal system.

## Creating the base VM

I downloaded the Debian 12 (bookworm) ISO and created a new VM in Proxmox.

Basic configuration:

```text
CPU:
1-2 cores

Memory:
1-2GB

Disk:
16GB+

Network:
VirtIO adapter
vmbr0 bridge
```

During the Debian installation:

* Installed the minimal Debian system
* Enabled SSH server
* Did not install a desktop environment
* Installed standard system utilities

After installation, the system was updated:

```bash
apt update
apt upgrade -y
```

## Preparing the VM before converting to a template

Before converting the VM into a template, I removed machine-specific information so every clone starts clean.

### Removed SSH host keys

SSH keys are unique to each machine and should not be copied into cloned VMs.

```bash
rm -f /etc/ssh/ssh_host_*
```

### Clear machine identity

Remove the existing machine ID:

```bash
truncate -s 0 /etc/machine-id
```

## Using the template

When creating a new VM:

1. Right-click the template
2. Select **Clone**
3. Choose:

```text
Mode:
Full Clone

Target Node:
Select the Proxmox node to deploy on

Target Storage:
Select the local VM storage
```

4. Give the VM:

   * A new VM ID
   * A name

Example:

```text
dns02
grafana
monitoring01
```

5. Start the cloned VM.

## First boot after cloning

Every clone should receive a new identity.

Things to configure:

### Set hostname

Example:

```bash
hostnamectl set-hostname dns02
```

Update `/etc/hosts` if needed.

### Configure networking

Assign:

* New IP address
* New MAC address

## Notes

The template itself does not consume CPU or memory. It only uses storage.

This template will be the starting point for future infrastructure VMs, allowing new services to be deployed quickly while keeping the base operating system consistent.

