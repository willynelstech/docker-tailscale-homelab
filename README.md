# Containerized Homelab & Zero Trust Mesh Network (Docker + Tailscale)

## Overview
A lightweight, self-hosted infrastructure environment deployed on resource-constrained local hardware. This lab demonstrates container orchestration, isolated bridge networking, and Zero Trust remote access using WireGuard-based mesh networking—eliminating the need to expose ports to the public internet.

---

## Architecture & Stack
* **Host Environment:** Local Linux Host running Docker Engine & Docker Compose
* **Overlay Network:** Tailscale (WireGuard-encrypted mesh VPN) providing point-to-point peer connectivity and private DNS resolution (MagicDNS)
* **Container Networking:** Custom Docker bridge networks isolating inter-container traffic from the host's physical LAN interface
* **Security Model:** Zero Trust / Identity-based access control (no inbound firewall port forwarding on the edge router)

---

## Key Implementation Highlights
1. **Zero-Exposure Remote Access:** Deployed Tailscale to establish encrypted peer-to-peer tunnels across NAT boundaries without opening external WAN ports (`0.0.0.0/0`), mitigating automated internet scanning and brute-force attack surfaces.
2. **Service Containerization:** Defined repeatable, declarative service stacks using `docker-compose.yml` with persistent volume mounts and restart policies (`unless-stopped`).
3. **Network Segmentation & DNS:** Configured internal container DNS resolution alongside Tailscale MagicDNS to route internal service requests cleanly across remote devices.

---

## Verification & Operational Checks
* **Mesh Connectivity:** Verified direct peer-to-peer DERP relay bypass and encrypted tunnel latency via `tailscale status` and `tailscale ping`.
* **Container Health & Isolation:** Audited active bridge subnets, port bindings, and container resource utilization via `docker network inspect` and `docker ps`.
