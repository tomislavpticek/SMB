# SMB Homelab

A small enterprise-style homelab built on **Proxmox VE** to practice
infrastructure administration, networking, virtualization, Linux,
Windows Server, Active Directory, firewalls, automation, monitoring, and
secure application deployment.

------------------------------------------------------------------------

# Table of Contents

-   [Overview](#overview)
-   [Architecture](#architecture)
-   [Network Diagram](#network-diagram)
-   [Security Zones](#security-zones)
-   [Subnets](#subnets)
-   [VM Inventory](#vm-inventory)
-   [Security Rules](#security-rules)

------------------------------------------------------------------------

# Overview

This project simulates a small business enterprise network following
common infrastructure and security best practices. The goal is to gain
practical hands-on experience with enterprise networking,
virtualization, operating systems, identity management, database
clustering, monitoring, automation, and secure application hosting.

## Goals

-   Learn Proxmox VE administration
-   Deploy segmented enterprise networks
-   Configure OPNsense firewalls
-   Build secure DMZ and Internal Server networks
-   Deploy Active Directory
-   Configure DNS and DHCP
-   Deploy a highly available PostgreSQL cluster
-   Deploy Redis for application sessions and caching
-   Configure HAProxy load balancing
-   Practice Linux administration
-   Practice Windows Server administration
-   Implement monitoring and backup solutions
-   Learn infrastructure automation
-   Prepare for Infrastructure and Cloud Engineering roles

------------------------------------------------------------------------

# Architecture

The environment follows a layered defense model.

``` text
Internet
    │
    ▼
OPNsense Firewall 1 (Edge)
    │
    ▼
DMZ
    │
OPNsense Firewall 2 (Internal)
    │
    ├── Internal Server Network
    ├── Workstation Network
    └── Management Network
```

Traffic between security zones is denied by default unless explicitly
permitted.

------------------------------------------------------------------------

# Network Diagram

Replace this placeholder with the latest exported draw.io diagram.

------------------------------------------------------------------------

# Security Zones

  Zone                      Purpose
  ------------------------- -------------------------------------------
  Internet                  External users
  DMZ                       Public-facing services
  Internal Server Network   Core infrastructure and business services
  Workstation Network       Administrative workstation
  Management Network        Infrastructure management

------------------------------------------------------------------------

# Subnets

## DMZ (10.0.1.0/24)

### Web Tier

-   Rocky Linux Web Server 1
-   Rocky Linux Web Server 2

### Reverse Proxy

-   Alpine HAProxy Load Balancer

### Remote Access

-   Rocky Linux OpenVPN Server

------------------------------------------------------------------------

## Internal Server Network (10.0.2.0/24)

### Infrastructure Services

-   Rocky Linux Automation Server
-   Rocky Linux Backup Server
-   Rocky Linux Monitoring Server

### Identity Services

-   Windows Server
    -   Active Directory
    -   DNS
    -   DHCP

### File Services

-   Rocky Linux File Server

### Database Services

#### PostgreSQL Cluster

-   Alpine PostgreSQL HAProxy Load Balancer
-   Rocky Linux PostgreSQL DB1
-   Rocky Linux PostgreSQL DB2

#### Redis

-   Alpine Redis Server

------------------------------------------------------------------------

## Workstation Network (10.0.3.0/24)

### Administrative Workstation

-   Windows Workstation

> A Linux workstation may be added in the future.

------------------------------------------------------------------------

## Management Network (10.0.10.0/24)

### Infrastructure

-   Proxmox
-   OPNsense Firewall 1
-   OPNsense Firewall 2

------------------------------------------------------------------------

# VM Inventory

  ------------------------------------------------------------------------------
  VM            Operating System                Purpose          Network
  ------------- ------------------------------- ---------------- ---------------
  Proxmox       Proxmox VE                      Hypervisor       Management

  OPNsense      OPNsense                        Edge Firewall    Management
  Firewall 1                                                     

  OPNsense      OPNsense                        Internal         Management
  Firewall 2                                    Firewall         

  Rocky Linux   Rocky Linux                     Remote VPN       DMZ
  OpenVPN                                       Access           
  Server                                                         

  Rocky Linux   Rocky Linux                     Public Web       DMZ
  Web Server 1                                  Server           

  Rocky Linux   Rocky Linux                     Public Web       DMZ
  Web Server 2                                  Server           

  Alpine        Alpine Linux                    Reverse Proxy /  DMZ
  HAProxy Load                                  Load Balancer    
  Balancer                                                       

  Windows       Windows Server                  Active           Internal
  Server                                        Directory, DNS,  
                                                DHCP             

  Rocky Linux   Rocky Linux                     SMB File Server  Internal
  File Server                                                    

  Rocky Linux   Rocky Linux                     Infrastructure   Internal
  Automation                                    Automation       
  Server                                                         

  Rocky Linux   Rocky Linux                     Scheduled        Internal
  Backup Server                                 Backups          

  Rocky Linux   Rocky Linux                     Monitoring and   Internal
  Monitoring                                    Alerting         
  Server                                                         

  Alpine        Alpine Linux                    PostgreSQL Load  Internal
  PostgreSQL                                    Balancer         
  HAProxy Load                                                   
  Balancer                                                       

  Rocky Linux   Rocky Linux                     PostgreSQL       Internal
  PostgreSQL                                    Primary          
  DB1                                                            

  Rocky Linux   Rocky Linux                     PostgreSQL       Internal
  PostgreSQL                                    Replica          
  DB2                                                            

  Alpine Redis  Alpine Linux                    Cache and        Internal
  Server                                        Session Store    

  Windows       Windows                         Administration   Workstation
  Workstation                                                    
  ------------------------------------------------------------------------------

------------------------------------------------------------------------

# Security Rules

## General Security Policy

-   Default deny between all security zones.
-   Only explicitly required ports are permitted.
-   No direct Internet access to internal services.
-   Administrative access is permitted only through the VPN.
-   Least-privilege access is enforced.
-   All Linux servers are patched regularly.
-   All systems synchronize with a common NTP source.

------------------------------------------------------------------------

## Firewall Policy

### DMZ → Internal

**Default Policy**

-   Deny all

**Allowed Exceptions**

  Source        Destination          Protocol
  ------------- -------------------- ----------
  Web Servers   PostgreSQL Cluster   TCP 5432
  Web Servers   Redis Server         TCP 6379

### VPN → Internal

**Default Policy**

-   Deny all

**Allowed Services**

  Service          Protocol
  ---------------- ------------
  DNS              TCP/UDP 53
  Kerberos         TCP/UDP 88
  LDAPS            TCP 636
  Global Catalog   TCP 3269
  SMB              TCP 445

VPN users may access only:

-   Active Directory
-   DNS
-   SMB File Server

All other services remain inaccessible.

------------------------------------------------------------------------

## Firewall Matrix

  Source        Destination               Action
  ------------- ------------------------- ------------------
  Internet      DMZ                       Allow HTTP/HTTPS
  Internet      Internal Server Network   Deny
  Internet      Workstation Network       Deny
  Internet      Management Network        Deny
  DMZ           Internal Server Network   Deny by default
  Web Servers   PostgreSQL Cluster        Allow TCP 5432
  Web Servers   Redis Server              Allow TCP 6379
  VPN           Active Directory          Allow
  VPN           DNS                       Allow
  VPN           SMB File Server           Allow
  VPN           PostgreSQL                Deny
  VPN           Redis                     Deny
  VPN           Management Network        Deny

------------------------------------------------------------------------

## Web Servers

-   Accessible only through the public IP address.
-   Internal users access services through the same public address.
-   Authentication is handled by the application.
-   User sessions are stored in Redis.
-   Web servers may access only PostgreSQL and Redis.

------------------------------------------------------------------------

## PostgreSQL Cluster

### Allowed Clients

-   Web Servers
-   Rocky Linux Backup Server
-   Authorized DBA Workstation

### Security

-   Listen only on the Internal Server Network.
-   SSL enabled.
-   SCRAM-SHA-256 authentication.
-   Administrative access restricted to authorized systems.

------------------------------------------------------------------------

## Redis Server

### Deployment

-   Dedicated virtual machine
-   Containerized
-   Persistent storage enabled
-   High-availability ready

### Security

-   Listen only on the Internal Server Network.
-   No Internet exposure.
-   Authentication enabled.
-   Accept connections only from the Web Servers.

### Purpose

-   Session storage
-   Application cache

------------------------------------------------------------------------

## File Server

Users authenticate using Active Directory credentials.

------------------------------------------------------------------------

## Backup Server

Scheduled backups include:

-   PostgreSQL Cluster
-   File Server

Backup traffic may originate only from the Backup Server.

------------------------------------------------------------------------

## Automation Server

Used for infrastructure automation and configuration management.

------------------------------------------------------------------------

## Monitoring Server

Provides infrastructure monitoring and alerting.

------------------------------------------------------------------------

## Management Network

### Members

-   Proxmox
-   OPNsense Firewall 1
-   OPNsense Firewall 2

### Access

Only authorized administrators may connect using:

-   SSH
-   HTTPS

The Management Network has no Internet exposure.
