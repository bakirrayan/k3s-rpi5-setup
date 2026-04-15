# K3s on Raspberry Pi 5

> Production-ready single-node Kubernetes cluster — with ingress, SSL, persistent storage and dynamic DNS.

---

## Stack

| Component | Purpose |
|---|---|
| **K3s** | Lightweight certified Kubernetes (etcd backend) |
| **MetalLB** | Bare-metal load balancer — assigns real LAN IPs |
| **ingress-nginx** | Routes subdomain traffic to the right pod |
| **cert-manager** | Automatic SSL via Let's Encrypt |
| **Longhorn** | Persistent storage on NVMe SSD |
| **Cloudflare** | DNS proxy, DDoS protection, wildcard subdomain |
| **ddclient** | Dynamic DNS — keeps Cloudflare updated on IP change |

---

## Hardware

- Raspberry Pi 5 — 8GB RAM
- NVMe SSD — 500GB (boot drive)
- Ubuntu Server 24.04 LTS 64-bit

---

## What you get

- Every service accessible at `https://service.yourdomain.com`
- SSL certificates issued and renewed automatically
- Your domain always points to your cluster even when your ISP changes your IP
- All traffic proxied and protected by Cloudflare

---

## Follow the guides in order

1. [01 — OS Preparation](01-os-preparation.md)
2. [02 — K3s Installation](02-k3s-installation.md)
3. [03 — MetalLB](03-metallb.md)
4. [04 — ingress-nginx](04-ingress-nginx.md)
5. [05 — cert-manager](05-cert-manager.md)
6. [06 — Longhorn Storage](06-longhorn.md)
7. [07 — Cloudflare DNS + DDNS](07-cloudflare-ddns.md)
8. [08 — Router Port Forwarding](08-port-forwarding.md)
9. [09 — Deploying a Service](09-deploying-a-service.md)
10. [10 — WireGuard VPN](10-wireguard-vpn.md)
11. [11 — Cloudflare Zero Trust](11-cloudflare-zero-trust.md)
12. [12 — DNS-01 cert-manager](12-dns01-cert-manager.md)

---

!!! tip "Single node"
    This setup is designed for a single Raspberry Pi 5. The etcd backend used in K3s allows you to add worker nodes later if needed.
