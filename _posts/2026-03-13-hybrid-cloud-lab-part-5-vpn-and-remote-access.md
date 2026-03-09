---
title: "Hybrid Cloud Lab Part 5: Hybrid Connectivity & Secure Access"
date: 2026-03-13 10:00:00 +0000
categories: [IT Infrastructure, Cloud Computing]
tags: [aws, vpn, ipsec, guacamole, security]
---

# Hybrid Cloud Lab Part 5: Hybrid Connectivity & Secure Access

This is where the project truly earns the "Hybrid Cloud" title. In this part, I’m detailing how I securely connected my on-premise Proxmox lab to the AWS cloud and implemented a remote access stack that allows me to manage the entire lab from any browser in the world.

## The Bridge: IPsec Site-to-Site VPN

I built an **IPsec IKEv2 Site-to-Site VPN** tunnel between my local pfSense firewall and a strongSwan responder running on an **AWS EC2** instance (Ubuntu 24.04).

### The Challenge of NAT Traversal
Because my home network has a dynamic public IP and sits behind a residential NAT, I had to design an asymmetric configuration:
- **Local (pfSense)** acts as the **Initiator**. It proactively "punches" through the NAT to find the static AWS IP.
- **Remote (AWS)** acts as the **Responder**. It waits for the connection from `%any` IP.

To keep the NAT state table from expiring and dropping the tunnel, I configured pfSense to send a "keep-alive" ping to the EC2 private IP every 10 seconds.

## Secure Remote Management

With the tunnel established, I deployed **Apache Guacamole** and **Nginx Proxy Manager** on the EC2 instance using Docker Compose.

- **Nginx Proxy Manager**: Handles HTTPS and SSL termination using Let's Encrypt.
- **Apache Guacamole**: Provides clientless RDP, SSH, and SFTP access.
- **Authentication**: Guacamole authenticates against the **Active Directory** on my local Windows Server—traversing the IPsec tunnel to verify credentials. No passwords are ever stored on the public cloud.

## Zero-Trust Security Stack

To protect this public-facing edge, I implemented three layers of security:

1.  **AWS Security Groups**: Restricted to my current home IP only for administrative ports (SSH, UDP 500/4500).
2.  **Fail2ban**: I injected custom rules into the `DOCKER-USER` iptables chain. If someone tries to brute-force the Nginx login, they are banned at the network level.
3.  **MFA**: Mandatory TOTP Multi-Factor Authentication for every Guacamole login.

### The Impact
Suddenly, my physical location didn't matter. I could open a browser on a laptop at a cafe in Tangier and get a full RDP session to my domain controller at home, with the traffic protected by AES-256 encryption.

In the final part of this series, we’ll look at the "Safety Net:" **Storage, Disaster Recovery, and Automation.**

Don't miss the conclusion in Part 6!
