# Self-Hosted Media & Photo Cloud with Zero-Trust Remote Access

## Overview
A self-hosted media streaming and photo backup server deployed on a repurposed Linux laptop. This project uses Docker and Portainer for container orchestration, implements a multi-tier storage layout across three physical drives to balance I/O performance with bulk capacity, and uses Tailscale to provide encrypted remote administration and media streaming without exposing ports to the public internet.

---

## System Architecture & Tiered Storage Design

To optimize performance and prevent disk I/O bottlenecks on laptop hardware, storage is separated by workload across three physical drives:

* **Internal NVMe/SATA SSD:** Houses the host Linux OS, Docker Engine, Portainer, and database volumes (PostgreSQL/Redis for Immich, metadata/transcode cache for Jellyfin) to ensure fast UI responsiveness and database reads/writes.
* **Internal HDD:** Dedicated bulk storage for Immich photo and video uploads, keeping large personal archives off the primary OS drive.
* **External USB SSD:** Dedicated storage for Jellyfin movie and TV show libraries, providing fast sequential read speeds for video streaming.
* **Network Layer (Tailscale Mesh VPN):** Connects the host laptop and remote client devices over encrypted WireGuard tunnels (`100.x.x.x` tailnet), keeping all web interfaces (`Portainer`, `Immich`, `Jellyfin`) completely closed to the public WAN.

---

## Step-by-Step Deployment

### 1. Prepare the Linux Host & Container Runtime
Started with a clean Linux installation on the laptop, updated system packages, installed Docker and Compose support, and granted non-root container management permissions to the primary user account.

### 2. Deploy Portainer for Container Management
Created a persistent volume on the internal SSD for Portainer configuration data and launched the Portainer container with access to the Docker socket and automatic restart policy so stacks and container health can be managed through a browser UI.

### 3. Configure Persistent Drive Mounts
Identified each storage drive by UUID and configured persistent mount points in `/etc/fstab` so the internal HDD and external SSD automatically mount at boot under consistent paths before container services start.

### 4. Deploy Immich First, Then Jellyfin
* **Immich:** Deployed first via Portainer Stacks using environment variables that keep database data on the internal SSD while mapping photo/video uploads to the internal HDD mount.
* **Jellyfin:** Deployed afterward via Portainer Stacks with configuration and transcode/metadata paths on the internal SSD and media libraries mounted read-only from the external SSD.

### 5. Enable Remote Access via Tailscale
Installed Tailscale on the Linux host and joined it to a private tailnet, then installed the Tailscale client on remote devices so administrative tasks, photo backups, and media streaming work outside the home network without opening external WAN ports.

---

## Verification & Operational Checks
* **Mount Persistence Check:** Rebooted the host and confirmed the internal HDD and external SSD mounted to their expected directories before services started, preventing containers from writing data onto unmounted root paths.
* **Container & Stack Health:** Checked running containers, restart policies, and local service ports in Portainer and via container status/log views to confirm Immich and Jellyfin stayed healthy.
* **Remote Access Check:** Connected at home from a different network over Tailscale and verified reachability to Portainer, Immich, and Jellyfin using the host's tailnet address.

---

## Key Takeaways & Troubleshooting Notes
* **Separating Database I/O from Bulk Storage:** Keeping database and metadata volumes on the internal SSD while placing large photo/video libraries on dedicated secondary drives avoids bottlenecks during indexing and library scans.
* **Zero Port Forwarding:** Using a mesh VPN keeps administrative and media services off the public internet, reducing exposure to automated scanning and unauthorized login attempts.

---

## Lessons Learned & Certification Alignment

Building and maintaining this environment bridged  exam objectives with live infrastructure trade-offs across my certifications:

* **CompTIA Network+ (IP Addressing, Virtual Networking & Storage/Network Operations):**
  * **Overlay & Virtual Networking:** Observed how Docker bridge networks isolate container traffic on internal subnets while Tailscale assigns private tailnet IPs (100.x.x.x) for encrypted peer-to-peer routing across NAT boundaries without manual router port forwarding.
  * **Service Availability & Persistence:** Learned why binding persistent storage by drive UUID in `/etc/fstab` prevents service failures and accidental writes to unmounted root directories across system reboots.

* **CompTIA Security+ (Zero Trust Architecture, Attack Surface Reduction & Least Privilege):**
  * **Attack Surface Reduction:** Eliminating inbound WAN firewall rules via a mesh VPN keeps administrative dashboards (Portainer) and media logins off the public internet, removing exposure to automated port scans and credential-stuffing bots.
  * **Least Privilege & Container Isolation:** Configured Jellyfin media volumes as read-only so the streaming container cannot modify or delete source files on the external SSD, and scoped non-root host permissions for daily administration.

* **CompTIA A+ & Linux Administration (Hardware Tiering & I/O Management):**
  * **Storage Performance Tiering:** Discovered firsthand how disk I/O bottlenecks affect application responsiveness—separating database/metadata reads onto the internal NVMe/SATA SSD while offloading bulk photo archives to an internal HDD and video files to an external SSD eliminated UI lag during library scans.
