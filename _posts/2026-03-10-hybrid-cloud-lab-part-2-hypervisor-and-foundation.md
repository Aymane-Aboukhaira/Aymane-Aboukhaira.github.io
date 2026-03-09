---
title: "Hybrid Cloud Lab Part 2: Hypervisor & Virtualization Foundation"
date: 2026-03-10 10:00:00 +0000
categories: [IT Infrastructure, Virtualization]
tags: [proxmox, pfsense, networking, hardware-troubleshooting]
---

# Hybrid Cloud Lab Part 2: Hypervisor & Virtualization Foundation

Behind every cloud infrastructure is a solid hypervisor. For this lab, I chose **Proxmox VE 8**, an open-source virtualization platform that turned my old laptop into a powerful mini-datacenter.

But as any sysadmin will tell you, the road to a stable environment is often paved with hardware quirks.

## The First Hurdle: Intel NIC Negotiation

Immediately after installing Proxmox, I hit my first real-world problem. The Intel I219-LM ethernet adapter on my Dell Latitude was failing to auto-negotiate, causing the network to drop intermittently.

This wasn't a software bug but a classic hardware-driver mismatch. I had to go under the hood and manually force the speed and duplex settings in `/etc/network/interfaces` using `ethtool`:

```bash
iface nic0 inet manual
    # Force 100Mbps full-duplex to bypass auto-negotiation failure
    pre-up ethtool -s nic0 speed 100 duplex full autoneg off
```

It wasn't a "glamorous" fix, but it was effective—and it taught me that enterprise infrastructure often requires these specific, manual tweaks.

## Setting up the pfSense Firewall

With a stable hypervisor, I deployed **pfSense CE** as my virtual router and firewall. pfSense acts as the "central nervous system" of the entire lab.

### Virtual Network Layout
I configured two bridges in Proxmox to separate traffic:
- `vmbr0`: The WAN uplink connected to my home network.
- `vmbr1`: A virtual bridge for the isolated internal lab LAN.

pfSense sits between these two, managing NAT, routing, and providing DHCP services for the lab. This isolation means I can experiment, break things, and reboot VMs without ever interrupting my home internet connection.

## Boot Order & Resource Management

With only **8GB of RAM**, every megabyte counts. I configured the boot order in Proxmox to ensure a smooth start-up after any power failure:

1.  **pfSense**: Starts first with a 30s delay. Routing and DHCP must be up before anything else.
2.  **DC01 (Active Directory)**: Starts second with a 60s delay. DNS needs to be ready for the other servers.
3.  **Application Servers**: Cloned from a base CentOS 9 template to minimize overhead.

Watching the pfSense dashboard go green and seeing the first VM pull a DHCP lease was a huge win. The foundation was set.

In the next part, I'll talk about how I moved away from a "flat" network and implemented **VLAN Segmentation** for a Zero-Trust architecture.

See you in Part 3!
