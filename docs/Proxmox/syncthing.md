# Syncthing Homelab Setup

## Overview

Syncthing is used as a self-hosted file synchronization service for personal data such as Obsidian vaults.

The setup uses a dedicated Syncthing VM running on Proxmox. Devices connect directly to Syncthing and synchronize folders between peers.

Current architecture:

```text
                 Syncthing VM
                      |
          +-----------+-----------+
          |           |           |
       Desktop    Laptop    Other Devices
```

## Purpose

This replaces third-party sync services by keeping synchronization inside my homelab.

Primary use case:

* Synchronize Obsidian notes between devices
* Maintain an always-online sync peer
* Keep data under local control

## Infrastructure

### Syncthing VM

Hosted on:

```text
Proxmox Cluster
```

Hostname:

```text
syncthing
```

Internal DNS:

```text
syncthing.homelab.local
```

Web interface:

```text
http://syncthing.homelab.local:8384
```

The Syncthing GUI is protected with authentication.

## Network Configuration

Syncthing VM:

```text
IP:
192.168.20.17
```

DNS record managed by Pi-hole:

```text
syncthing.homelab.local -> 192.168.20.17
```

The GUI should only be accessible from the internal LAN.

## VM Layout

The Syncthing VM stores synchronized data separately from the operating system.

Example:

```text
/home/<user>/sync
└── obsidian
    ├── Notes
    ├── Attachments
    └── .obsidian
```

The Syncthing data directory is separate from the VM OS to make rebuilding easier.

## Installing Syncthing

Install package:

```bash
sudo apt update
sudo apt install syncthing
```

Enable the user service:

```bash
systemctl --user enable syncthing.service
systemctl --user start syncthing.service
```

Enable startup without login:

```bash
loginctl enable-linger $USER
```

Check status:

```bash
systemctl --user status syncthing.service
```

## Adding a New Device

### 1. Install Syncthing

Install Syncthing on the new device.

Example Arch Linux:

```bash
sudo pacman -S syncthing
```

Enable service:

```bash
systemctl --user enable syncthing.service
systemctl --user start syncthing.service
```

Open the local GUI:

```text
http://localhost:8384
```

---

### 2. Get Device ID

On the new device:

```text
Actions
 -> Show ID
```

Copy the Device ID.

---

### 3. Add Device to Syncthing VM

Open:

```text
http://syncthing.homelab.local:8384
```

Go to:

```text
Remote Devices
 -> Add Remote Device
```

Enter:

```text
Device ID:
<new device ID>
```

Give it a descriptive name:

Example:

```text
arch-desktop
laptop
phone
```

Save.

---

### 4. Accept Device Pairing

On the new device, accept the incoming device request.

The two devices should now show as connected.

---

### 5. Share Folders

On the Syncthing VM:

```text
Folders
 -> obsidian
 -> Edit
 -> Sharing
```

Select the new device.

Save.

The new device will receive a folder invitation.

---

### 6. Choose Local Folder

On the new device:

Example:

```text
~/Documents/Obsidian/MyVault
```

Set folder type:

```text
Send & Receive
```

Save.

---

## Troubleshooting

### Check Syncthing service

```bash
systemctl --user status syncthing.service
```

### Check logs

```bash
journalctl --user -u syncthing.service
```

### Check GUI port

```bash
ss -tulpn | grep 8384
```

Expected:

```text
127.0.0.1:8384
```

or:

```text
192.168.20.17:8384
```

### Check DNS

```bash
nslookup syncthing.homelab.local
```

Expected:

```text
192.168.20.17
```

