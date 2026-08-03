# Proxmox NFS Backup Storage

## Overview

`backup-nfs` is a shared NFS storage location used by the Proxmox cluster to store VM backups and provide an easy way to move VM templates or machines between Proxmox nodes.

The Proxmox cluster consists of three NUC nodes with local storage. Since the VM disks are stored locally on each node, direct cloning or migration between nodes is not possible without shared storage.

Instead, `backup-nfs` provides a central location where VM backups can be stored and restored from any Proxmox node.

Current architecture:

```
             Proxmox Cluster

        +----------+----------+
        |          |          |
       pve1       pve2       pve3
        |          |          |
        +----------+----------+
                   |
                   |
             backup-nfs (NFS)
                   |
             /backup/proxmox
```

The NFS share is currently hosted on `pve1` and mounted as Proxmox storage on the cluster.

## Why This Exists

Each Proxmox node uses local storage:

```
pve1
 └── local-lvm

pve2
 └── local-lvm

pve3
 └── local-lvm
```

Because the VM disks are local, a VM or template created on one node cannot be directly cloned onto another node.

`backup-nfs` provides an intermediate shared storage layer:

```
VM / Template
      |
      v
Backup to backup-nfs
      |
      v
Restore on another Proxmox node
```

This allows templates and VMs to be moved between nodes without manually copying backup files.

## NFS Server Configuration

The NFS server runs on `pve1`.

The exported directory is:

```
/backup/proxmox
```

The export is configured in:

```
/etc/exports
```

Example:

```
/backup/proxmox 192.168.1.0/24(rw,sync,no_subtree_check)
```

After modifying exports:

```bash
exportfs -ra
```

Verify active exports:

```bash
exportfs -v
```

## Proxmox Storage Configuration

The NFS share is added to Proxmox as:

```
Storage ID: backup-nfs
Type: NFS
```

The storage appears in Proxmox:

```
Datacenter
 └── Storage
      └── backup-nfs
```

All Proxmox nodes can access this storage.

## Creating VM Backups

To back up a VM:

1. Select the VM.
2. Open **Backup**.
3. Select:

```
Storage: backup-nfs
Mode: Snapshot
Compression: ZSTD
```

The backup will be stored as:

```
/backup/proxmox/dump/
```

Example:

```
vzdump-qemu-100-2026_08_03-12_00_00.vma.zst
```

## Moving a VM Between Nodes

Example:

```
pve1 → pve3
```

Workflow:

```
pve1 VM
  |
  | Backup
  v
backup-nfs
  |
  | Restore
  v
pve3 VM
```

Steps:

1. Create a backup on the source node.
2. Select the backup from `backup-nfs` on the destination node.
3. Restore the VM.
4. Select the destination node's local storage.

## Moving Templates

Proxmox templates are stored as VM definitions with the template flag enabled.

To move a template:

1. Backup the template:

```
Template
  |
  v
backup-nfs
```

2. Restore it on another node:

```
backup-nfs
  |
  v
pve3
```

3. Convert it back into a template if required:

```bash
qm template <VMID>
```

Alternatively, if a template only needs to become a normal VM:

```bash
qm template <VMID> 0
```

## Managing Backups

List backups:

```bash
ls -lh /backup/proxmox/dump
```

Remove a backup through Proxmox:

```
Datacenter
 └── Storage
      └── backup-nfs
           └── Backups
                └── Remove
```

Or from the command line:

```bash
pvesm free backup-nfs:backup/<backup-file>
```

or

```bash
sudo rm -rf /backup/dump/*
```

from `pve1`

## Current Limitations

`backup-nfs` improves VM portability but is not a complete backup strategy.

Current limitation:

```
pve1 failure
    |
    v
backup-nfs unavailable
```

Because the NFS share is hosted on a Proxmox node, it is not independent storage.

A future improvement would be moving backups to:

* A dedicated Proxmox Backup Server
* A NAS
* Separate backup storage

