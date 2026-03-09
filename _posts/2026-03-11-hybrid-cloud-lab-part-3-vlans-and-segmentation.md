---
title: "Hybrid Cloud Lab Part 3: VLANs & Zero-Trust Segmentation"
date: 2026-03-11 10:00:00 +0000
categories: [IT Infrastructure, Networking]
tags: [pfsense, vlans, security, zero-trust]
---

# Hybrid Cloud Lab Part 3: VLANs & Zero-Trust Segmentation

A "flat" network where every device can talk to every other device is a security nightmare. If one machine is compromised, the attacker has a direct path to everything else. This is why **Network Segmentation** is a cornerstone of enterprise security.

In Part 3 of this series, I’m digging into how I implemented a **Zero-Trust** architecture using VLANs and pfSense.

## The VLAN Strategy

I divided my lab into four distinct networks, each with its own purpose and security level:

| VLAN | Name | Purpose |
|---|---|---|
| **10** | MGMT | Management traffic, Active Directory, Domain Controller. |
| **20** | SERVICES | Infrastructure services like Prometheus and Grafana monitoring. |
| **30** | DMZ | Isolated network for public-facing web applications. |
| **40** | STORAGE | Restricted segment for NAS storage (SMB/NFS). |

## The Proxmox "Virtual Bridge" Challenge

Implementing VLAN tags (802.1Q) on a virtual bridge in Proxmox turned out to be more complex than expected. Proxmox requires at least one physical NIC attached to a VLAN-aware bridge to validate it. Since my internal bridge (`vmbr1`) was purely virtual, my VMs wouldn't boot.

The solution? A **Kernel Dummy Interface**. I injected a dummy NIC via the Linux kernel, bound it to the bridge, and suddenly everything clicked into place:

```bash
auto dummy0
iface dummy0 inet manual
    # Create a virtual dummy interface to satisfy Proxmox's bridge validation
    pre-up ip link add dummy0 type dummy || true
```

## Enforcing the Zero-Trust Matrix

With the VLANs established, I defined strict firewall rules in pfSense to control inter-VLAN traffic. The goal was simple: **"Block by default, allow by exception."**

For example, a machine in the **DMZ (VLAN 30)** can access the internet to serve web traffic, but it is explicitly blocked from even "seeing" the **Management (VLAN 10)** or **Storage (VLAN 40)** networks. If an attacker compromises a web app, they are stuck in a digital "dead end."

## The Pivot to Enterprise

This segmentation wasn't just for show; it was a response to real feedback. My initial design was flat, and moving to this model taught me exactly how much overhead—and security—comes with enterprise-grade networking.

In the next part, we'll look at the "brains" inside these VLANs: **Active Directory** and my **Centralized Monitoring** stack.

Stay tuned for Part 4!
