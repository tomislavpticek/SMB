# Network Design

## Security Rules

### General Security Policy

- Default deny between security zones.
- Only explicitly required ports are permitted.
- No direct Internet access to internal services.
- Administrative access is permitted only through the VPN.
- All servers synchronize time with a common NTP source.
- All Linux servers are patched regularly.
- Least-privilege access is enforced.

---

### DMZ → Internal

**Default Policy**

- ❌ DENY ALL

**Exceptions**

| Source | Destination | Port |
|---------|-------------|------|
| Web Servers (DMZ) | PostgreSQL Cluster | TCP 5432 |
| Web Servers (DMZ) | Redis | TCP 6379 |

---

### VPN → Internal

**Default Policy**

- ❌ DENY ALL

**Allowed Services**

| Service | Port |
|----------|------|
| DNS | TCP/UDP 53 |
| Kerberos | TCP/UDP 88 |
| LDAPS | TCP 636 |
| Global Catalog (LDAPS) | TCP 3269 |
| SMB | TCP 445 |

VPN users may access only:

- Active Directory
- DNS
- SMB File Server

All other internal services are denied.

---

### Web Servers

- Accessible only through the public IP address.
- Internal users access the same public address as external users.
- Authentication is handled by the application.
- Web servers do **not** authenticate directly against Active Directory.
- User sessions are stored in Redis.
- Web servers may access only:
  - PostgreSQL (TCP 5432)
  - Redis (TCP 6379)

---

### PostgreSQL Cluster

**Location**

- Internal Network

**Communication**

- Accepts connections only from:
  - Web Servers
  - Backup Server
  - Authorized DBA Workstation

**Security**

- Listen only on the server subnet.
- SSL enabled.
- SCRAM-SHA-256 authentication.
- Administrative access restricted to authorized systems.
- VPN users have no database access.

---

### Redis

**Deployment**

- Dedicated VM
- Containerized
- Clustered / Replicated
- Persistent storage enabled

**Security**

- Listen only on the server subnet.
- No Internet exposure.
- Authentication enabled.
- Accept connections only from Web Servers.

**Purpose**

- Session storage
- Application cache

---

### File Server

Authentication:

- Active Directory username
- Active Directory password

---

### Backup Server

Scheduled backups:

- PostgreSQL Cluster
- File Server

Only the Backup Server may initiate backup traffic.

---

### Automation Server

Backup traffic is permitted only from the Backup Server.

---

### Management Network (10.0.10.0/24)

**Members**

- Proxmox
- OPNsense Firewall 1
- OPNsense Firewall 2

**Access**

- Authorized administrators only
- SSH
- HTTPS

No Internet exposure.

---

# Network Layout

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

- Windows Server
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

Administrative workstations:

- Windows Workstation
- Linux Workstation

> Final number and operating systems are subject to change.

---

## Management Network (10.0.10.0/24)

### Hypervisors

- Proxmox

### Firewalls

- OPNsense Firewall 1
- OPNsense Firewall 2
