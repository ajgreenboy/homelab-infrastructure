# Homelab Infrastructure

**Production-grade home server environment demonstrating enterprise DevOps practices and modern infrastructure patterns**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker)](https://www.docker.com/)
[![Linux](https://img.shields.io/badge/OS-Debian%2013-A81D33?logo=debian)](https://www.debian.org/)
[![Network](https://img.shields.io/badge/Router-OpenWRT-00B5E2?logo=openwrt)](https://openwrt.org/)
[![Monitoring](https://img.shields.io/badge/Monitoring-Prometheus%20%2B%20Grafana-E6522C?logo=prometheus)](https://prometheus.io/)

---

## Table of Contents

- [Overview](#overview)
- [Network Architecture](#network-architecture)
- [Infrastructure Components](#infrastructure-components)
- [Services & Applications](#services--applications)
- [Monitoring & Observability](#monitoring--observability)
- [Remote Access](#remote-access)
- [Hardware Specifications](#hardware-specifications)
- [Skills Demonstrated](#skills-demonstrated)
- [Future Enhancements](#future-enhancements)
- [Documentation](#documentation)

---

## Overview

This repository documents a production homelab environment built on Debian 13, showcasing enterprise-grade infrastructure practices including containerized service orchestration, comprehensive monitoring, automated backups, and secure remote access. The infrastructure serves as both a learning platform and a functional media/cloud services deployment.

**Core Principles:**
- **Infrastructure as Code:** All services defined declaratively via Docker Compose
- **Observability First:** Comprehensive metrics collection and visualization
- **Security by Design:** Defense-in-depth with VPN mesh networking and zero-trust access
- **Automated Operations:** Self-healing services with automated updates and backups
- **Scalable Architecture:** Modular design supporting future expansion

**Service Categories:**
- **34 containerized services** across 5 functional stacks
- **Media automation** with *arr ecosystem + Jellyfin + Plex
- **Cloud services** including photo management and file sync
- **Smart home** integration with Home Assistant
- **Full-stack monitoring** with Prometheus + Grafana + Uptime Kuma
- **Comprehensive management** with Homepage dashboard, Portainer, Dockge, and Dozzle

**External Access:**
- Jellyfin Media Server: [jellyfin.unfunky.xyz](https://jellyfin.unfunky.xyz) (Cloudflare Tunnel)
- Audiobookshelf: Secure Cloudflare Tunnel access
- Private services accessible via Tailscale mesh VPN

---

## Network Architecture

### Topology Overview

```
Internet (Xfinity ISP / Dynamic IP with DDNS)
    ↓
Cloudflare (DNS Management + Zero Trust Gateway + DDoS Protection)
    ↓
ASUS RT-AC3100 Router (OpenWRT 23.05, Gigabit Routing, Dual-band WiFi)
    ↓
Primary Network (192.168.1.0/24)
    ├─ SFF Debian Server (192.168.1.29) - Main infrastructure host
    ├─ Raspberry Pi 3B+ - Moode Audio Server
    └─ Trusted devices, workstations, mobile devices
    ↓
Docker Network Architecture (5 isolated networks)
    ├─ proxy_network      - Reverse proxy & management services
    ├─ media_network      - Media automation stack (*arr apps)
    ├─ cloud_network      - Cloud services & databases
    ├─ monitoring_network - Metrics collection & visualization
    └─ valheim_network    - Game server isolation
    ↓
Tailscale Mesh VPN (WireGuard-based, 5+ devices)
    └─ Encrypted remote access to all services
```

### Network Design Rationale

| Component | Technology Choice | Justification |
|-----------|------------------|---------------|
| **Router Firmware** | OpenWRT 23.05 | Superior control over routing, firewall rules, and advanced network features compared to stock firmware |
| **VPN Architecture** | Tailscale WireGuard mesh | Zero-configuration P2P encrypted access, no port forwarding required, sub-50ms latency |
| **Service Isolation** | Docker network segmentation | Logical separation of service stacks, improved security posture, simplified firewall rules |
| **Container Runtime** | Docker with Compose V2 | Industry-standard orchestration, simplified dependency management, rapid deployment/rollback |
| **Public Access** | Cloudflare Tunnel | Eliminates port forwarding, DDoS protection, SSL/TLS automation, zero-trust access control |

---

## Infrastructure Components

### Physical Hosts

#### Primary Server: Custom SFF x86 System

| Component | Specification | Purpose |
|-----------|---------------|---------|
| **Motherboard** | MSI B450 | AMD AM4 platform, PCIe for GPU |
| **Processor** | AMD Ryzen 5 2600 (6C/12T @ 3.4GHz) | Multi-threaded workload handling |
| **Memory** | 16GB DDR4 | Container orchestration, in-memory caching |
| **Boot Drive** | 120GB SSD (80GB used, 46%) | OS + Docker system volumes |
| **Data Drive** | 6TB HDD (1.8TB used, 34%) | Media library storage |
| **GPU** | AMD Radeon R5 340 | Hardware-accelerated transcoding (VA-API) |
| **OS** | Debian 13 (Trixie) - Headless | Stable, long-term support |
| **Network** | Gigabit Ethernet | 1000 Mbps wired connection |
| **Power** | 80W idle / 125W load | ~$105/year @ $0.15/kWh |
| **Uptime** | 99.5%+ | Prometheus-monitored availability |

**Current Capacity:**
- **CPU Usage:** ~11% average (Valheim server primary consumer)
- **RAM Usage:** 5.4GB / 16GB used (66% free capacity)
- **Storage:** 3.7TB free (room for significant expansion)
- **⚠️ Root Disk:** 46% full - monitor and cleanup recommended

**Storage Architecture:**
```
/dev/sda (120GB SSD) - LVM Layout
├─ /boot/efi     (976MB)   - UEFI boot partition
├─ /boot         (977MB)   - Kernel and initramfs
└─ LVM VG        (109.9GB)
   ├─ root       (81.3GB)  - Root filesystem (46% full)
   └─ swap       (4.4GB)   - Swap space

/dev/sdb1 (5.5TB HDD) - Media Storage
└─ /mnt/storage (34% used)
   ├─ media/              - Jellyfin & Plex libraries
   │  ├─ movies/
   │  ├─ tv/
   │  ├─ music/
   │  ├─ books/
   │  ├─ audiobooks/
   │  ├─ podcasts/
   │  └─ photos/          - Immich photo library
   ├─ downloads/          - SABnzbd staging
   │  ├─ complete/
   │  └─ incomplete/
   ├─ appdata/            - Persistent container data
   │  ├─ nextcloud/
   │  ├─ immich/
   │  ├─ homeassistant/
   │  ├─ prometheus/
   │  └─ postgresql/
   └─ backups/            - Automated backup storage
      ├─ duplicati/       - Cloud backup staging
      └─ system/          - Local system backups
```

**Docker Configuration Storage:**
```
/opt/docker-configs/   - Service configurations (SSD for fast access)
~/docker/              - Docker Compose definitions
  ├─ core/             - Management & infrastructure services
  ├─ media/            - Media automation stack
  ├─ cloud/            - Cloud services
  ├─ monitoring/       - Metrics collection & visualization
  └─ gaming/           - Game servers (Valheim)
```

---

## Services & Applications

**Deployment Model:** All services run as Docker containers, managed via Docker Compose with Portainer and Dockge for orchestration and monitoring.

### Core Management Stack (11 containers)

| Service | Container | Port | Function | Access |
|---------|-----------|------|----------|--------|
| **Homepage** | homepage | 3000 | Unified dashboard with service widgets and live stats | Local + Tailscale |
| **Nginx Proxy Manager** | nginx-proxy-manager | 81 | Reverse proxy with automatic SSL/TLS certificate management | Local + Tailscale |
| **Portainer** | portainer | 9000 | Docker container orchestration and management UI | Local + Tailscale |
| **Dockge** | dockge | 5001 | Docker Compose stack management and editor | Local + Tailscale |
| **Dozzle** | dozzle | 8081 | Real-time log viewer for all containers | Local + Tailscale |
| **File Browser** | filebrowser | 8082 | Web-based file manager for server storage | Local + Tailscale |
| **Duplicati** | duplicati | 8200 | Automated backup service with cloud integration | Local + Tailscale |
| **Watchtower** | watchtower | - | Automated container image updates (daily at 4 AM) | Background |
| **Autoheal** | autoheal | - | Automatically restarts unhealthy containers | Background |
| **Cloudflared (Plex)** | cloudflared-plex | - | Cloudflare Tunnel for Plex public access | Background |
| **Cloudflared (Audiobooks)** | cloudflared-audiobookshelf | - | Cloudflare Tunnel for Audiobookshelf | Background |

**Homepage Dashboard Features:**
- Live service status and health checks
- Integrated widgets for media services (Jellyfin, Plex, Sonarr, Radarr, Prowlarr, SABnzbd, Jellyseerr)
- Quick access links to all 34 services
- Organized by category: Media, Downloads, Cloud, Smart Home, Management, Monitoring
- API integration for real-time statistics

**Watchtower Configuration:**
- Update schedule: Daily at 4:00 AM CST
- Telegram notifications on container updates
- Automatic cleanup of old images
- Monitors all containers except databases (safety precaution)

**Automated Healing:**
- Autoheal monitors container health checks
- Automatically restarts unhealthy containers
- 60-second check interval with 5-minute startup grace period

### Media Automation Stack (9 containers)

| Service | Container | Port | Function | External Access |
|---------|-----------|------|----------|-----------------|
| **Jellyfin** | jellyfin | 8096 | Media streaming server with GPU transcoding | **Public** (Cloudflare Tunnel) |
| **Plex** | plex | 32400 | Music streaming server with remote access | Tailscale VPN |
| **Audiobookshelf** | audiobookshelf | 13378 | Audiobook & podcast streaming with mobile sync | **Public** (Cloudflare Tunnel) |
| **Jellyseerr** | jellyseerr | 5055 | Media request and discovery platform | Tailscale VPN |
| **Sonarr** | sonarr | 8989 | TV series automation and library management | Internal |
| **Radarr** | radarr | 7878 | Movie automation and library management | Internal |
| **Prowlarr** | prowlarr | 9696 | Centralized indexer manager for *arr ecosystem | Internal |
| **SABnzbd** | sabnzbd | 8080 | Usenet download client with category routing | Internal |

**Integration Flow:**
```
Jellyseerr (User Request)
    ↓
Sonarr/Radarr (Search for content via Prowlarr)
    ↓
SABnzbd (Download from Usenet)
    ↓
Sonarr/Radarr (Process, rename, move to media library)
    ↓
Jellyfin/Plex (Media appears in library, ready for streaming)
```

**Media Server Features:**

**Jellyfin (Primary Video Streaming):**
- GPU Hardware Transcoding: AMD Radeon R5 340 (VA-API)
- Supported codecs: H.264, HEVC, VC1, VP8, VP9
- Performance: 3-4 concurrent 1080p transcodes
- Power efficiency: ~30W under transcode vs ~80W CPU-only
- Public access via Cloudflare Tunnel

**Plex (Music Streaming):**
- Dedicated music library management
- Remote access and mobile apps
- Plexamp support for advanced music features
- CloudSync integration

**Audiobookshelf (Audiobooks & Podcasts):**
- Chapter support and bookmarking
- Mobile app with offline downloads
- Progress sync across devices
- Podcast episode management
- Public access via Cloudflare Tunnel

### Cloud Services Stack (7 containers)

| Service | Container | Port | Function | Data Storage |
|---------|-----------|------|----------|--------------|
| **Nextcloud** | nextcloud | 8888 | File sync, sharing, and collaboration platform | /mnt/storage/appdata/nextcloud |
| **Nextcloud DB** | nextcloud-db | - | PostgreSQL database for Nextcloud | Persistent volume |
| **Nextcloud Redis** | nextcloud-redis | - | In-memory cache for performance | Ephemeral |
| **Immich** | immich-server | 2283 | AI-powered photo management and organization | /mnt/storage/media/photos |
| **Immich Microservices** | immich-microservices | - | Background job processing (thumbnails, ML) | Shared volume |
| **Immich ML** | immich-machine-learning | - | Facial recognition and object detection | Model cache |
| **Immich DB** | immich-postgres | - | PostgreSQL + pgvector for embeddings | Persistent volume |

**Nextcloud Features:**
- File synchronization across devices
- Calendar and contact management
- Document collaboration (Office integration)
- End-to-end encryption support
- Mobile apps for iOS/Android
- Redis caching for optimal performance

**Immich Features:**
- AI-powered facial recognition
- Object and scene detection
- Geographic photo clustering
- Automatic backup from mobile devices
- Privacy-focused (self-hosted alternative to Google Photos)
- Version: v1.117.0 (stable deployment)

### Smart Home Integration (1 container)

| Service | Container | Port | Function | Network Mode |
|---------|-----------|------|----------|--------------|
| **Home Assistant** | homeassistant | 8123 | Home automation platform and device orchestration | **Host mode** (for device discovery) |

**Features:**
- IoT device integration and control
- Automation rule engine
- Voice assistant integration capability
- Energy monitoring and analytics
- Local control (no cloud dependency)

**Note:** Runs in host network mode to enable mDNS/discovery protocols for smart home devices.

### Monitoring & Observability Stack (4 containers)

| Service | Container | Port | Function | Update Frequency |
|---------|-----------|------|----------|-----------------|
| **Prometheus** | prometheus | 9090 | Time-series metrics database and alerting engine | 15 seconds |
| **Grafana** | grafana | 3001 | Metrics visualization and dashboard platform | Real-time |
| **Node Exporter** | node-exporter | 9100 | System-level metrics (CPU, RAM, disk, network) | Continuous |
| **Uptime Kuma** | uptime-kuma | 3002 | Service uptime monitoring and status page | 60 seconds |

**Metrics Collected:**
- **System:** CPU usage (per-core), memory utilization, disk I/O, network throughput
- **Containers:** Per-container CPU/memory, network I/O, filesystem usage, restart counts
- **Docker:** Total container count, image storage, volume usage
- **Services:** HTTP endpoint health checks, response times, SSL certificate expiration

**Uptime Kuma Features:**
- HTTP/HTTPS monitoring for all web services
- Docker container health monitoring
- Notification support (Telegram, Discord, email)
- Public status page capability
- Incident history and uptime statistics

**Grafana Dashboards:**
1. **Node Exporter Full** - Comprehensive system monitoring
2. **Docker Container Metrics** - Per-container resource tracking
3. **System Overview** - High-level infrastructure health
4. **Custom Dashboards** - Service-specific monitoring

### Game Servers (1 container)

| Service | Container | Port | Function | Status |
|---------|-----------|------|----------|--------|
| **Valheim** | valheim | 2456-2457 | Dedicated Viking survival multiplayer server | Running (CPU: 11%) |

**Configuration:**
- Server type: Dedicated (Linux)
- Network: Isolated valheim_network
- Backups: Automated hourly world saves
- Auto-updates: Every 3 hours
- Capacity: 2-10 concurrent players

---

## Monitoring & Observability

### Monitoring Architecture

**Philosophy:** Comprehensive observability through metrics collection, visualization, uptime monitoring, and log aggregation.

**Data Pipeline:**
```
System Resources → Node Exporter (port 9100) ──┐
Docker Containers → Container Metrics ─────────┤
HTTP Endpoints → Uptime Kuma (port 3002) ──────┼→ Prometheus (port 9090) → Grafana (port 3001)
Container Logs → Dozzle (port 8081) ───────────┤
Service Status → Homepage (port 3000) ─────────┘
```

### Multi-Layer Monitoring Approach

**1. Infrastructure Monitoring (Prometheus + Grafana)**
- System resource metrics (CPU, RAM, disk, network)
- Container resource utilization
- 30-day historical data retention
- 15-second metric collection interval

**2. Service Uptime Monitoring (Uptime Kuma)**
- HTTP health checks for all web services
- Docker container health status
- SSL certificate monitoring
- Incident tracking and notifications
- Public status page capability

**3. Log Monitoring (Dozzle)**
- Real-time log streaming from all containers
- Search and filter capabilities
- Multi-container log aggregation
- No log persistence (ephemeral viewing)

**4. Dashboard & Management (Homepage)**
- Service status overview
- Integrated widgets with live statistics
- Quick access to all services
- Health check integration

### Key Metrics & Dashboards

**System Health:**
- CPU: ~11% average (Valheim server primary load)
- RAM: 5.4GB / 16GB used (66% free)
- Root Disk: 46% full (monitoring required)
- Data Disk: 34% full (3.7TB free)
- Network: Gigabit fully utilized
- Uptime: 99.5%+ availability

**Service Monitoring:**
- All 34 containers monitored for health
- Automated restart on failure (Autoheal)
- Daily update checks (Watchtower)
- Real-time log access (Dozzle)

---

## Remote Access

### Tailscale VPN Mesh Network

**Architecture:** WireGuard-based peer-to-peer encrypted mesh network connecting 5+ devices

**Connected Nodes:**
- Debian server (designated exit node for internet routing)
- Personal laptop (primary administration interface)
- Mobile device (iOS - remote monitoring)
- Family member devices (limited access scope)
- Additional trusted devices

**Advantages:**
- **Zero Configuration:** Automatic NAT traversal, no manual port forwarding
- **Low Latency:** Direct peer-to-peer connections when network topology permits
- **High Security:** WireGuard cryptography, automatic key rotation
- **Reliability:** Measured latency <50ms for local network access
- **Simplicity:** Single command deployment, cross-platform compatibility

**Use Cases:**
- Secure SSH access to server infrastructure from anywhere
- Access to internal web services (Portainer, Grafana, *arr apps, Homepage)
- Remote media streaming via Jellyfin (alternative to Cloudflare)
- Network-level access to entire homelab from remote locations
- Exit node functionality for secure internet browsing when traveling

### Public Web Access

**Jellyfin Media Server:**
- **URL:** https://jellyfin.unfunky.xyz
- **Access Method:** Cloudflare Tunnel (cloudflared container)
- **Security Layers:**
  1. Cloudflare DDoS protection and WAF
  2. Jellyfin user authentication
  3. SSL/TLS encryption (automatic certificate management)
- **Performance:** CDN-accelerated delivery, sub-200ms latency globally

**Audiobookshelf:**
- **Access Method:** Cloudflare Tunnel (dedicated cloudflared container)
- **Function:** Public access to audiobook and podcast library
- **Security:** User authentication required

### Access Control Matrix

| Service | Local Network | Tailscale VPN | Public Internet | Authentication |
|---------|---------------|---------------|-----------------|----------------|
| Homepage | ✅ Direct | ✅ Direct | ❌ Blocked | N/A |
| Jellyfin | ✅ Direct | ✅ Direct | ✅ Cloudflare Tunnel | User login |
| Plex | ✅ Direct | ✅ Direct | ❌ Blocked | Plex account |
| Audiobookshelf | ✅ Direct | ✅ Direct | ✅ Cloudflare Tunnel | User login |
| Portainer | ✅ Direct | ✅ Direct | ❌ Blocked | Admin login |
| Dockge | ✅ Direct | ✅ Direct | ❌ Blocked | N/A |
| Grafana | ✅ Direct | ✅ Direct | ❌ Blocked | Admin login |
| Uptime Kuma | ✅ Direct | ✅ Direct | ❌ Blocked | Admin login |
| Nextcloud | ✅ Direct | ✅ Direct | ❌ Blocked | User login |
| Immich | ✅ Direct | ✅ Direct | ❌ Blocked | User login |
| Home Assistant | ✅ Direct | ✅ Direct | ❌ Blocked | User login |
| Sonarr/Radarr | ✅ Direct | ✅ Direct | ❌ Blocked | API key |
| SSH (port 22) | ✅ Key-based | ✅ Key-based | ❌ Blocked | SSH keys only |

---

## Hardware Specifications

### Server Hardware Details

**Form Factor:** Small Form Factor (SFF) for space-efficient deployment

**Power Consumption & Efficiency:**
- Idle state: ~80W (system + HDD spin)
- Media streaming (no transcode): ~90W
- Single transcode (GPU): ~110W
- Multiple transcodes + downloads: ~125W peak
- Annual power cost: ~$105 @ $0.15/kWh (assuming 80% idle, 20% active)

**Reliability Metrics:**
- Uptime: 99.5%+ (measured via Prometheus)
- Mean time between failures (MTBF): No hardware failures in 12+ months
- Unplanned downtime: <4 hours/month (primarily OS updates)
- Planned maintenance windows: Monthly (system updates, testing)

**Storage Performance:**
- Boot SSD sequential read/write: ~500 MB/s / ~450 MB/s
- Media HDD sequential read/write: ~180 MB/s / ~160 MB/s
- Docker volume I/O: Direct SSD access (optimal for databases)
- Media streaming: HDD sufficient for 10+ concurrent streams

### Network Equipment Details

**ASUS RT-AC3100 (OpenWRT 23.05)**
- **Chipset:** Broadcom BCM4709C0 (1.4GHz dual-core ARM Cortex-A9)
- **Memory:** 512MB DDR3 RAM, 128MB NAND flash storage
- **WiFi Capabilities:**
  - 2.4GHz: 802.11n (up to 1000 Mbps)
  - 5GHz: 802.11ac (up to 2167 Mbps)
  - Simultaneous dual-band operation
- **Ethernet:** 4x Gigabit LAN ports, 1x Gigabit WAN port
- **Features:**
  - QoS traffic prioritization
  - DDNS integration (No-IP, DuckDNS)
  - UPnP for automatic port mapping
  - Advanced firewall with custom iptables rules

---

## Skills Demonstrated

This homelab infrastructure showcases proficiency across multiple technical domains:

### Systems Administration
- **Linux Server Management:** Debian 13 installation, configuration, and maintenance
- **Storage Management:** LVM configuration, filesystem optimization, automated mounting, capacity planning
- **User & Permission Management:** Sudo configuration, group membership, file permissions
- **System Monitoring:** Resource tracking, performance optimization, uptime management
- **Security Hardening:** SSH key-based authentication, firewall configuration (UFW), user isolation
- **Capacity Planning:** Proactive monitoring of disk usage and resource allocation

### Containerization & Orchestration
- **Docker Fundamentals:** Image management, container lifecycle, volume/network configuration
- **Docker Compose:** Multi-container application definitions, service dependencies, network isolation
- **Service Deployment:** 34-container infrastructure with declarative configuration
- **Container Networking:** Custom bridge networks, service discovery, inter-container communication
- **Resource Management:** CPU/memory limits, restart policies, health checks
- **Container Health:** Automated healing with health check monitoring

### Management & Operations
- **Service Dashboard:** Homepage deployment with widget integration and API management
- **Container Orchestration:** Portainer and Dockge for visual management
- **Log Aggregation:** Centralized logging with Dozzle for troubleshooting
- **Backup Strategy:** Duplicati integration with cloud storage providers
- **File Management:** Web-based file browser for remote administration
- **Update Automation:** Watchtower configuration with notification integration

### Networking
- **Network Architecture:** LAN design, static IP assignment, DHCP reservation
- **Router Configuration:** OpenWRT firmware, advanced routing, port forwarding
- **VPN Technologies:** Tailscale mesh network deployment, WireGuard protocol understanding
- **Reverse Proxy:** Nginx Proxy Manager configuration, SSL/TLS certificate automation
- **DNS Management:** Cloudflare integration, DDNS for dynamic IP handling
- **Service Isolation:** Docker network segmentation for security

### Monitoring & Observability
- **Metrics Collection:** Prometheus time-series database configuration and tuning
- **Visualization:** Grafana dashboard design, panel creation, query optimization
- **System Metrics:** Node Exporter deployment, custom metric collection
- **Uptime Monitoring:** Uptime Kuma deployment with health checks and notifications
- **Log Management:** Real-time log viewing with Dozzle
- **Service Integration:** Homepage widget integration with API key management

### Security
- **Access Control:** Multi-layer authentication, principle of least privilege
- **Network Security:** Firewall rule design, port management, service isolation
- **Encryption:** SSL/TLS certificate management, VPN encryption (WireGuard)
- **SSH Hardening:** Key-based authentication, root login disabled
- **Service Hardening:** Container isolation, health checks, automated restart
- **Zero Trust Architecture:** Cloudflare Zero Trust integration for public services

### Cloud & DevOps Practices
- **Infrastructure as Code:** Declarative service definitions, version-controlled configurations
- **Automated Deployment:** Docker Compose-based service orchestration
- **Backup Strategy:** Automated backup with Duplicati and cloud integration
- **Update Management:** Watchtower automated container updates with Telegram notifications
- **Documentation:** Comprehensive README, inline comments, troubleshooting guides
- **Version Control:** Git-based configuration management

### Application Integration
- **Media Automation:** *arr ecosystem integration (Sonarr, Radarr, Prowlarr, etc.)
- **Multi-Server Setup:** Jellyfin for video + Plex for music + Audiobookshelf for audiobooks
- **Database Administration:** PostgreSQL deployment and configuration for Nextcloud/Immich
- **Caching Layers:** Redis integration for application performance
- **API Integration:** Cloudflare API, Telegram Bot API, service webhooks
- **Smart Home:** Home Assistant device integration and automation

---

## Future Enhancements

### Immediate Priority (Next 1-2 Weeks)

**Root Disk Cleanup:**
- Current: 46% full (approaching capacity limits)
- Action: Docker system prune, old image cleanup, log rotation
- Target: Reduce to <40% usage

**Container Version Updates:**
- Current: Many containers 16-20 months old
- Action: Update pinned versions or switch to :latest tags
- Priority: Immich, Jellyfin, Portainer, Sonarr, Radarr, Jellyseerr

**Power Monitoring:**
- Action: Deploy smart plug with power monitoring
- Integration: Home Assistant + Homepage widget
- Purpose: Track server power consumption and costs

### Security Improvements (High Priority - 2-4 weeks)

**Fail2Ban Intrusion Prevention:**
- Deploy Fail2Ban for SSH and web service protection
- Configure jails: SSH (3 attempts), HTTP 404/403 scanning detection
- Telegram integration for real-time IP ban notifications
- Custom Prometheus exporter for ban metrics
- GeoIP mapping of banned IPs for attack pattern analysis

**Network Segmentation:**
- Implement VLAN 1 (Production) and VLAN 2 (IoT/Untrusted)
- Configure inter-VLAN firewall rules (deny IoT → Production)
- Migrate smart home devices to isolated IoT network
- Document VLAN configuration in OpenWRT

**2FA & Access Hardening:**
- Implement 2FA for critical services (Portainer, Grafana, Nextcloud)
- Consider Authelia or Authentik for unified SSO
- Audit and document all service authentication mechanisms

### Monitoring & Alerting (Medium Priority - 1-2 months)

**Grafana Alert Rules:**
- CPU usage > 80% sustained (5 minute window)
- Memory usage > 90% available
- Disk space > 85% full (boot drive and media drive)
- Container restart loops (3+ restarts in 10 minutes)
- Service downtime detection

**Centralized Logging:**
- Deploy Loki + Promtail stack for log aggregation
- Integrate with Grafana for unified observability
- Log retention policy (30-day rotation)
- Replace Dozzle with persistent log solution

**Additional Exporters:**
- Jellyfin metrics exporter (stream count, transcode load)
- SABnzbd metrics exporter (queue depth, download rate)
- Nextcloud metrics exporter (user count, storage usage)
- Portainer metrics integration

### Infrastructure Automation (Medium Priority - 2-3 months)

**Ansible Deployment:**
- Convert Docker Compose to Ansible playbooks
- Automated server provisioning from bare metal
- Idempotent configuration management
- Secret management with Ansible Vault
- Multi-environment support (dev/staging/prod)

**Backup Enhancements:**
- Offsite backup replication (cloud storage integration via Duplicati)
- Automated restore testing (monthly validation)
- Database-specific backup strategies (pg_dump automation)
- Backup monitoring and alerting via Uptime Kuma

**GitOps Workflow:**
- GitHub repository for infrastructure-as-code
- Automated deployment on git push (webhooks)
- Version-controlled service configurations
- Rollback capabilities
- Change tracking and audit log

### Service Expansion (Low Priority - 3-6 months)

**Planned New Services:**
- **Vaultwarden:** Self-hosted password manager (Bitwarden alternative)
- **Paperless-ngx:** Document scanning and management
- **AdGuard Home:** DNS-level ad blocking and tracking prevention
- **Actual Budget:** Personal finance and budgeting application
- **Kavita:** eBook and manga reader
- **Bazarr:** Subtitle management for media library
- **Tautulli:** Plex statistics and monitoring

**Home Assistant Expansion:**
- Voice assistant integration (local processing)
- Energy monitoring dashboards
- Automation rule library expansion
- Mobile app presence detection

**Game Server Expansion:**
- Additional game servers (Minecraft, Terraria, etc.)
- Pterodactyl Panel for unified game server management

### Hardware Upgrades (Long-Term - 6-12 months)

**Compute & Storage Expansion:**
- RAM upgrade: 16GB → 32GB (support more containers)
- Storage upgrade: Additional 6TB HDD (RAID 1 mirror for redundancy)
- UPS addition: Graceful shutdown on power loss, runtime monitoring
- Network upgrade: 2.5GbE NIC for faster local transfers

---

## Documentation

### Repository Structure

```
homelab-infrastructure/
├─ README.md                    - This file (updated overview and architecture)
├─ docs/
│  ├─ WHATS-NEXT-ROADMAP.md    - Detailed enhancement roadmap
│  ├─ IMPLEMENTATION-COMPLETE.md - Recent implementation log
│  ├─ CONTAINERS-FIXED.md       - Container troubleshooting history
│  ├─ security-monitoring.md    - Security implementation guide
│  └─ network-diagram.png       - Network topology visualization
├─ docker/
│  ├─ core/
│  │  └─ docker-compose.yml     - Management & infrastructure services
│  ├─ media/
│  │  └─ docker-compose.yml     - Media automation stack
│  ├─ cloud/
│  │  └─ docker-compose.yml     - Cloud services
│  ├─ monitoring/
│  │  └─ docker-compose.yml     - Prometheus + Grafana + Uptime Kuma
│  └─ gaming/
│     └─ valheim/
│        └─ docker-compose.yml  - Valheim dedicated server
├─ scripts/
│  ├─ backup-homelab.sh         - Automated backup script
│  └─ homelab-status.sh         - System status checker
└─ LICENSE                      - MIT License
```

---

## Project Timeline

**Initial Deployment:** December 2025
**Major Expansion:** February 2026
**Last Updated:** February 2026
**Status:** 🟢 Active Production

### Recent Updates (February 2026)

**Infrastructure Expansion:**
- Expanded from 24 to 34 containers
- Added Homepage dashboard with service widgets
- Deployed Dockge for Docker Compose management
- Integrated Dozzle for real-time log viewing
- Added File Browser for web-based file management
- Implemented Duplicati for automated cloud backups
- Deployed Uptime Kuma for service monitoring
- Added Autoheal for automatic container recovery

**Media Services:**
- Added Plex for dedicated music streaming
- Deployed Audiobookshelf with Cloudflare Tunnel access
- Configured second Cloudflare Tunnel for public access

**Game Servers:**
- Deployed Valheim dedicated server
- Isolated gaming network for security

**Monitoring Enhancements:**
- Integrated Uptime Kuma for HTTP health checks
- Added Homepage widgets with live API statistics
- Enhanced Grafana dashboard coverage

**Optimizations:**
- Resolved container health issues
- Configured API key management for all services
- Improved service isolation with dedicated networks
- Documented capacity planning metrics

---

## Contact & Links

**Albert Weiner**

📧 Email: [ajgreenboy@gmail.com](mailto:ajgreenboy@gmail.com)
💼 LinkedIn: [linkedin.com/in/al-weiner-29865529a](https://www.linkedin.com/in/al-weiner-29865529a/)
🐙 GitHub: [github.com/ajgr33nboy](https://github.com/ajgr33nboy)
🌐 Portfolio: [unfunky.xyz](https://unfunky.xyz)

**Live Demos:**
- 🎬 Jellyfin Media Server: [jellyfin.unfunky.xyz](https://jellyfin.unfunky.xyz)
- 📚 Audiobookshelf: Secure public access via Cloudflare Tunnel
- 📊 Monitoring Dashboard: Internal access via Tailscale VPN
- 📷 Photo Gallery: Internal access via Tailscale VPN

---

## Acknowledgments

**Technologies Used:**
- Debian Project
- Docker & Docker Compose
- Prometheus, Grafana & Uptime Kuma
- Tailscale (WireGuard)
- Cloudflare
- LinuxServer.io container images
- Servarr ecosystem (*arr applications)
- Gethomepage (Homepage dashboard)
- Louislam (Dockge, Uptime Kuma)

**Community Resources:**
- r/homelab
- r/selfhosted
- Servarr Discord community
- LinuxServer.io Discord
- Tailscale community forum
- Homepage Discord

---

## License

This project documentation is released under the MIT License. Configuration files and scripts are provided as-is for educational and reference purposes.

See [LICENSE](LICENSE) file for full details.

---

**🏠 Built with passion in Minneapolis, Minnesota**
**⚡ Powered by Debian, Docker, and caffeine**
**🎯 Demonstrating enterprise DevOps practices at home scale**
**📊 34 containers, 5.5TB storage, 99.5%+ uptime**

---

*"The best way to predict the future is to build it."*
