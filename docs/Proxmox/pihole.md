# Pi-hole and Unbound DNS

## Overview

The homelab network uses two Pi-hole instances running with Unbound as local recursive DNS resolvers.

The purpose of having two DNS servers is to avoid having a single point of failure. If one Proxmox node or DNS VM is unavailable, clients can continue resolving DNS using the second resolver.

## Architecture

```text
                 Home Router
                     |
          DHCP DNS Configuration
                     |
          -----------------------
          |                     |
       dns01                 dns02
       pve1 VM               pve2 VM
          |                     |
      Pi-hole +             Pi-hole +
       Unbound                Unbound
```

Both DNS servers are independent and run the same software stack:

* Debian
* Pi-hole
* Unbound

## Router Configuration

The router advertises both DNS servers to clients through DHCP.

Example:

```text
Primary DNS:
192.168.20.2

Secondary DNS:
192.168.20.3
```

## Pi-hole Configuration

Pi-hole is used for:

* Local DNS records for homelab services
* Ad blocking
* DNS management

The main customizations currently made in Pi-hole are local DNS records.

```text
grafana.homelab.local
prometheus.homelab.local
argocd.homelab.local
```

## Unbound Configuration

Each Pi-hole instance forwards DNS queries to a local Unbound resolver.

This provides:

* Recursive DNS resolution
* No dependency on external DNS providers
* Better privacy compared to forwarding all queries to a public resolver

The DNS flow is:

```text
Client
  |
  v
Pi-hole
  |
  v
Unbound
  |
  v
DNS Root Servers
```

## VM Placement

The DNS servers run on separate Proxmox nodes.

Current placement:

```text
dns01:
Proxmox node: pve1

dns02:
Proxmox node: pve2
```

Keeping them on separate nodes protects against a single Proxmox host failure.

## Maintenance

Changes made to Pi-hole configuration should be applied to both instances.

Currently, the only configuration that requires manual synchronization is local DNS records.

Future improvements:

* Manage Pi-hole DNS records through Ansible
* Store DNS configuration in Git
* Automate deployment and updates

## Verification

Useful checks:

Check Pi-hole:

```bash
systemctl status pihole-FTL
```

Check Unbound:

```bash
systemctl status unbound
```

Test local DNS:

```bash
dig grafana.homelab.local
```

Test recursive DNS:

```bash
dig example.com
```

