# SMB Homelab

A small enterprise-style homelab built on **Proxmox VE** to practice
infrastructure administration, networking, virtualization, Linux,
Windows Server, Active Directory, firewalls, automation, monitoring, and
secure application deployment.

## Table of Contents

-   [Overview](#overview)
-   [Architecture](#architecture)
-   [Network Diagram](#network-diagram)
-   [Security Zones](#security-zones)
-   [Subnets](#subnets)
-   [VM Inventory](#vm-inventory)
-   [Security Rules](#security-rules)

## Overview

This project simulates a secure small business infrastructure using
enterprise design principles.

### Goals

-   Learn Proxmox VE
-   Learn OPNsense
-   Deploy Active Directory
-   Practice Linux and Windows Server administration
-   Build a secure DMZ
-   Build a segmented Internal Server Network
-   Deploy PostgreSQL, Redis and HAProxy
-   Learn monitoring and automation

## Architecture

``` text
Internet
    │
    ▼
OPNsense Firewall 1
    │
    ▼
DMZ
    │
    ▼
OPNsense Firewall 2
    │
    ├── Internal Server Network
    ├── Workstation Network
    └── Management Network
```

Traffic between security zones is denied unless explicitly permitted.

## Network Diagram

> Replace with the latest draw.io export.

## Security Zones

  Zone                      Purpose
  ------------------------- -----------------------------------
  Internet                  External users
  DMZ                       Public-facing services
  Internal Server Network   Servers and business applications
  Workstation Network       Administrative workstation
  Management Network        Infrastructure management

## Subnets

### DMZ (10.0.1.0/24)

#### Web Tier

-   Rocky Linux Web Server 1
-   Rocky Linux Web Server 2

#### Reverse Proxy

-   Alpine HAProxy Load Balancer

#### Remote Access

-   Rocky Linux OpenVPN Server

### Internal Server Network (10.0.2.0/24)

#### Infrastructure

-   Rocky Linux Automation Server
-   Rocky Linux Backup Server
-   Rocky Linux Monitoring Server

#### Identity Services

-   Windows Server (AD, DNS, DHCP)

#### File Services

-   Rocky Linux File Server

#### Database Services

**PostgreSQL Cluster** - Alpine PostgreSQL HAProxy Load Balancer - Rocky
Linux PostgreSQL DB1 - Rocky Linux PostgreSQL DB2

**Redis** - Alpine Redis Server

### Workstation Network (10.0.3.0/24)

-   Windows Administrative Workstation

### Management Network (10.0.10.0/24)

-   Proxmox
-   OPNsense Firewall 1
-   OPNsense Firewall 2

## VM Inventory

  -----------------------------------------------------------------------
  VM                OS                Purpose           Network
  ----------------- ----------------- ----------------- -----------------
  Proxmox           Proxmox VE        Hypervisor        Management

  OPNsense Firewall OPNsense          Edge Firewall     Management
  1                                                     

  OPNsense Firewall OPNsense          Internal Firewall Management
  2                                                     

  Rocky Linux       Rocky Linux       VPN               DMZ
  OpenVPN Server                                        

  Rocky Linux Web   Rocky Linux       Web Server        DMZ
  Server 1                                              

  Rocky Linux Web   Rocky Linux       Web Server        DMZ
  Server 2                                              

  Alpine HAProxy    Alpine Linux      Web Load Balancer DMZ
  Load Balancer                                         

  Windows Server    Windows Server    AD/DNS/DHCP       Internal

  Rocky Linux File  Rocky Linux       File Server       Internal
  Server                                                

  Rocky Linux       Rocky Linux       Automation        Internal
  Automation Server                                     

  Rocky Linux       Rocky Linux       Backup            Internal
  Backup Server                                         

  Rocky Linux       Rocky Linux       Monitoring        Internal
  Monitoring Server                                     

  Alpine PostgreSQL Alpine Linux      DB Load Balancer  Internal
  HAProxy Load                                          
  Balancer                                              

  Rocky Linux       Rocky Linux       PostgreSQL        Internal
  PostgreSQL DB1                                        

  Rocky Linux       Rocky Linux       PostgreSQL        Internal
  PostgreSQL DB2                                        

  Alpine Redis      Alpine Linux      Cache / Sessions  Internal
  Server                                                

  Windows           Windows           Administration    Workstation
  Workstation                                           
  -----------------------------------------------------------------------

## Security Rules

### General Security Policy

-   Default deny between all security zones.
-   Only explicitly required ports are permitted.
-   Administrative access only via VPN.
-   No direct Internet access to internal services.
-   Least-privilege access is enforced.
-   Linux servers are patched regularly.
-   Common NTP source for all systems.

### DMZ → Internal

Default: **DENY ALL**

  Source        Destination          Port
  ------------- -------------------- ----------
  Web Servers   PostgreSQL Cluster   TCP 5432
  Web Servers   Redis Server         TCP 6379

### VPN → Internal

Default: **DENY ALL**

Allowed: - DNS (53) - Kerberos (88) - LDAPS (636) - Global Catalog
(3269) - SMB (445)

VPN users may access only AD, DNS and SMB.

### PostgreSQL Cluster

Allowed clients: - Web Servers - Rocky Linux Backup Server - Authorized
DBA Workstation

Security: - Listen only on server subnet - SSL enabled - SCRAM-SHA-256
authentication

### Redis Server

-   Dedicated VM
-   Authentication enabled
-   No Internet exposure
-   Accepts connections only from Web Servers
-   Used for session storage and application cache

### Backup Server

Backs up: - PostgreSQL Cluster - File Server

### Automation Server

Configuration management and automation.

### Monitoring Server

Infrastructure monitoring and alerting.

### Management Network

Accessible only by authorized administrators over SSH and HTTPS.
