---
title: "Hybrid Cloud Lab Part 6: Resilience & Automation"
date: 2026-03-14 10:00:00 +0000
categories: [IT Infrastructure, DevOps]
tags: [storage, backups, automation, bash, disaster-recovery]
---

# Hybrid Cloud Lab Part 6: Resilience & Automation

We’ve reached the final part of my Hybrid Cloud Infrastructure Lab series. We’ve built the network, the identity system, and the cloud bridge. But no infrastructure is complete without a solid safety net. Today, I’m talking about **Storage, Disaster Recovery (DR), and Automation.**

## Persistent Storage with OpenMediaVault

In an enterprise environment, data needs a central home. I deployed **OpenMediaVault (OMV)** on a lightweight VM (1GB RAM) to act as my lab’s Network Attached Storage (NAS).

OMV provides:
- **SMB Shares**: Easily accessible from the Windows Management workstation.
- **NFS Mounts**: Mounted persistently on my CentOS application servers for low-latency file access.

The key lesson here was managing persistence. Using `/etc/fstab` with the `_netdev` option ensured that my servers wouldn't hang on boot if the NAS was still starting up—a small but critical detail for system stability.

## The Three-Tier Disaster Recovery Strategy

"Backup" is just a copy; "Disaster Recovery" is a plan. I implemented a three-tier strategy to ensure that even a total hardware failure wouldn't wipe out months of work.

1.  **Tier 1 (Physical)**: An air-gapped USB drive containing full full-VM snapshots (.vma.zst) for one-click restoration via the Proxmox GUI.
2.  **Tier 2 (Cloud Edge)**: Daily automated backups of EC2 configurations (Nginx, Guacamole, IPsec) to an **AWS S3** bucket.
3.  **Tier 3 (Cloud On-Prem)**: A centralized script on `centos-vm2` that pulls the pfSense configuration and monitoring data, then uploads it to S3 through the IPsec tunnel.

### Ransomware Protection
My S3 bucket uses a **write-only IAM policy**. The backup script can upload files, but it doesn't have the permission to delete or overwrite them. Coupled with S3 versioning, this means even if a script is compromised, the historical backups stay safe.

## Self-Healing Automation

One of my favorite features of this lab is its ability to "heal" itself. Because I have a dynamic ISP IP, the AWS Security Groups would frequently block my traffic after an IP rotation.

I wrote a bash script that runs every 5 minutes:
1.  Queries **DuckDNS** for my current home IP.
2.  Compares it to the IP allowed in the AWS Security Group.
3.  Updates the Security Group via the AWS CLI if they don’t match.

This turned a frustrating manual task into a background process that just works.

## Conclusion: The Lab is Never Finished

Building this lab has been the most challenging and rewarding experience of my studies. It taught me that enterprise engineering isn't just about knowing the tools; it's about understanding the "why" behind the architecture.

The lab is running, segmented, and secure—but it’s not finished. Next, I’m looking at Ansible for orchestration and Terraform to codify my AWS setup.

Thank you for following along with this series. I hope it inspires you to build your own infrastructure!

**Aymane Aboukhaira**
*Networks & Systems — ISMONTIC, Tangier*
