---
title: "Hybrid Cloud Lab Part 1: The Vision & Architecture"
date: 2026-03-09 10:00:00 +0000
categories: [IT Infrastructure, Cloud Computing]
tags: [architecture, planning, virtualization, hybrid-cloud]
---

# Hybrid Cloud Lab Part 1: The Vision & Architecture

Welcome to the first part of my Hybrid Cloud Infrastructure Lab series. As a Networks & Systems engineering student, I’ve always found that theoretical knowledge only takes you so far. To truly understand how enterprise systems interact, you have to build them, break them, and fix them yourself.

In this series, I'm documenting my journey of building a production-grade hybrid cloud environment from scratch.

## The Objective

The goal of this project was to simulate a real-world enterprise environment using limited resources:
- Commodity hardware (an old laptop).
- AWS Free Tier credits.
- Four months of dedicated work.

I wanted to demonstrate hands-on skills across networking, Linux/Windows administration, security, and DevOps automation.

## The Architecture at a Glance

The lab is split between an on-premise environment running on Proxmox and a cloud edge running on AWS. They are connected via a secure IPsec Site-to-Site VPN.

```text
[Admin Workstation]
         |
         | HTTPS / SSH
         |
[Proxmox VE 8 - On-Premise Host]
         |
         |--- pfSense (Firewall + IPsec VPN Gateway)
         |         |
         |         |===[ IPsec Site-to-Site VPN (IKEv2/AES-256) ]===
         |                                |
         |                      [AWS EC2 Cloud Edge]
         |                      |- Nginx Proxy Manager (SSL)
         |                      |- Apache Guacamole (Remote Access)
         |                      |- Fail2ban Security
         |                      |- S3 Backup Storage
         |
         |--- VLAN 10: Management (Windows AD, DNS, DHCP)
         |--- VLAN 20: Services (Prometheus, Grafana Monitoring)
         |--- VLAN 30: DMZ (Isolated Web Applications)
         |--- VLAN 40: Storage (OpenMediaVault NAS)
```

## Design Philosophy

When planning this lab, I followed three core principles that I believe are essential for any modern sysadmin:

1.  **Defense in Depth**: No single security control is enough. Every path has multiple independent layers (Firewalls, SG, MFA, Fail2ban).
2.  **Least Privilege**: Components only get the permissions they absolutely need. IAM policies are scoped tightly, and VLANs prevent lateral movement.
3.  **Infrastructure as Documentation**: Every decision is documented. If the lab burns down today, I should be able to rebuild it using this repository.

## The Hardware Constraint

One of the most valuable lessons I learned was working within constraints. My main server is a Dell Latitude E5470 with just **8GB of RAM**. This forced me to be extremely efficient with virtualization—planning which VMs run simultaneously and optimizing memory usage at every layer.

In the next post, I’ll dive into the foundation: setting up Proxmox VE 8 and the initial networking hurdles I faced.

Stay tuned!
