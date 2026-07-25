# SMB Homelab

A small enterprise-style homelab built on Proxmox VE to practice infrastructure administration, networking, virtualization, Linux, Windows Server, Active Directory, firewalls, automation, and secure application deployment.

---

# Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Network Diagram](#network-diagram)
- [Security Zones](#security-zones)
- [Subnets](#subnets)
- [VM Inventory](#vm-inventory)
- [Security Rules](#security-rules)

---

# Overview

This project simulates a small business enterprise network following common security and infrastructure best practices.

## Goals

- Learn Proxmox VE administration
- Deploy segmented enterprise networks
- Configure OPNsense firewalls
- Build secure DMZ and Internal networks
- Deploy Active Directory
- Build a highly available PostgreSQL cluster
- Deploy Redis for application sessions and caching
- Configure HAProxy load balancing
- Practice Linux administration
- Learn Windows Server administration
- Prepare for infrastructure and cloud engineering roles

---

# Architecture

The environment follows a layered security model.

```
Internet
      │
      ▼
Firewall 1 (Edge)
      │
      ▼
DMZ
      │
Firewall 2 (Internal)
      │
      ▼
Internal Server Network
      │
      ├── Workstation Network
      └── Management Network
```

Traffic between every security zone is denied unless explicitly permitted.

---

# Network Diagram

<img width="1912" height="1002" alt="SMB architecture drawio (1)" src="https://github.com/user-attachments/assets/a5edb56d-d45e-4605-9ad7-571375f6fdeb" />


---

# Security Zones

| Zone | Purpose |
|------|---------|
| Internet | External users |
| DMZ | Public-facing services |
| Internal Server Network | Application servers and business services |
| Workstation Network | Administrative workstations |
| Management Network | Infrastructure management |

---

# Subnets

## DMZ (10.0.1.0/24)

### Web Tier

- Rocky Linux Web Server 1
- Rocky Linux Web Server 2

### Reverse Proxy

- Alpine HAProxy Load Balancer

### Remote Access

- VPN Server

---

## Internal Server Network (10.0.2.0/24)

### Identity Services

Windows Server

- Active Directory
- DNS
- DHCP

### File Services

- Rocky Linux File Server

### Database Services

**PostgreSQL Cluster**

- Rocky Linux PostgreSQL DB 1
- Rocky Linux PostgreSQL DB 2

**Redis**

- Alpine Redis Server

### Internal Reverse Proxy

- Alpine HAProxy Load Balancer

---

## Workstation Network (10.0.3.0/24)

Administrative workstations

- Windows Workstation
- Linux Workstation

> Final number and operating systems may change during development.

---

## Management Network (10.0.10.0/24)

### Hypervisors

- Proxmox

### Firewalls

- OPNsense Firewall 1
- OPNsense Firewall 2

---

# VM Inventory

| VM | Operating System | Purpose | Network |
|----|------------------|---------|---------|
| OPNsense Firewall 1 | OPNsense | Edge Firewall | Management |
| OPNsense Firewall 2 | OPNsense | Internal Firewall | Management |
| Web Server 1 | Rocky Linux | Web Server | DMZ |
| Web Server 2 | Rocky Linux | Web Server | DMZ |
| HAProxy (DMZ) | Alpine Linux | Reverse Proxy / Load Balancer | DMZ |
| VPN Server | Linux | Remote Access | DMZ |
| Windows Server | Windows Server | Active Directory, DNS, DHCP | Internal |
| File Server | Rocky Linux | SMB File Server | Internal |
| PostgreSQL DB 1 | Rocky Linux | PostgreSQL Primary | Internal |
| PostgreSQL DB 2 | Rocky Linux | PostgreSQL Replica | Internal |
| Redis Server | Alpine Linux | Redis Cache / Sessions | Internal |
| HAProxy (Internal) | Alpine Linux | Internal Load Balancer | Internal |
| Proxmox | Proxmox VE | Hypervisor | Management |
| Windows Workstation | Windows | Administration | Workstation |
| Linux Workstation | Linux | Administration | Workstation |

---

# Security Rules

## General Security Policy

- Default deny between all security zones.
- Only explicitly required ports are permitted.
- No direct Internet access to internal services.
- Administrative access is allowed only through the VPN.
- All servers synchronize time with a common NTP source.
- All Linux servers are patched regularly.
- Least-privilege access is enforced.

---

## Firewall Policy

### DMZ → Internal

**Default Policy**

- ❌ DENY ALL

**Allowed Exceptions**

| Source | Destination | Protocol |
|---------|-------------|----------|
| Web Servers | PostgreSQL Cluster | TCP 5432 |
| Web Servers | Redis Server | TCP 6379 |

---

### VPN → Internal

**Default Policy**

- ❌ DENY ALL

**Allowed Services**

| Service | Protocol |
|----------|----------|
| DNS | TCP/UDP 53 |
| Kerberos | TCP/UDP 88 |
| LDAPS | TCP 636 |
| Global Catalog (LDAPS) | TCP 3269 |
| SMB | TCP 445 |

VPN users may access only:

- Active Directory
- DNS
- SMB File Server

All other services are denied.

---

## Firewall Matrix

| Source | Destination | Action |
|---------|-------------|--------|
| Internet | DMZ | Allow HTTP/HTTPS |
| Internet | Internal Server Network | Deny |
| Internet | Management Network | Deny |
| Internet | Workstation Network | Deny |
| DMZ | Internal Server Network | Deny |
| Web Servers | PostgreSQL Cluster | Allow TCP 5432 |
| Web Servers | Redis Server | Allow TCP 6379 |
| VPN | Active Directory | Allow |
| VPN | DNS | Allow |
| VPN | SMB File Server | Allow |
| VPN | PostgreSQL | Deny |
| VPN | Redis | Deny |
| VPN | Management Network | Deny |

---

## Web Servers

- Accessible only through the public IP address.
- Internal users access services through the same public address as external users.
- Authentication is handled locally by the application.
- Web servers do not authenticate directly against Active Directory.
- User sessions are stored in Redis.
- Web servers may access only:
  - PostgreSQL (TCP 5432)
  - Redis (TCP 6379)

---

## PostgreSQL Cluster

### Location

- Internal Server Network

### Communication

Connections are accepted only from:

- Web Servers
- Backup Server
- Authorized DBA Workstation

### Security

- Listen only on the server subnet.
- SSL enabled.
- SCRAM-SHA-256 authentication.
- Administrative access restricted to authorized systems.
- VPN users have no database access.

---

## Redis Server

### Deployment

- Dedicated virtual machine
- Containerized
- Clustered / Replicated
- Persistent storage enabled

### Security

- Listen only on the server subnet.
- No Internet exposure.
- Authentication enabled.
- Accept connections only from Web Servers.

### Purpose

- Session storage
- Application cache

---

## File Server

Users authenticate using:

- Active Directory username
- Active Directory password

---

## Backup Server

Scheduled backups include:

- PostgreSQL Cluster
- File Server

Backup traffic may originate only from the Backup Server.

---

## Automation Server

Backup traffic is permitted only from the Backup Server.

---

## Management Network

### Members

- Proxmox
- OPNsense Firewall 1
- OPNsense Firewall 2

### Access

Only authorized administrators may connect using:

- SSH
- HTTPS

The Management Network has no Internet exposure.
