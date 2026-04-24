# K3s on Raspberry Pi 5

> Production-ready Kubernetes cluster — with ingress, SSL, persistent storage, monitoring, automation and Zero Trust security.

---

## Stack

| Component | Purpose |
|---|---|
| **K3s** | Lightweight certified Kubernetes |
| **MetalLB** | Bare-metal load balancer — assigns real LAN IPs |
| **ingress-nginx** | Routes subdomain traffic to the right pod |
| **cert-manager** | Automatic SSL via Let's Encrypt |
| **Longhorn** | Persistent storage on NVMe SSD |
| **Cloudflare** | DNS proxy, DDoS protection, wildcard subdomains |
| **Cloudflare Zero Trust** | Email authentication on all subdomains |
| **ddclient** | Dynamic DNS — keeps Cloudflare updated on IP change |
| **Prometheus + Grafana** | Cluster monitoring and dashboards |
| **n8n** | Self-hosted workflow automation |
| **Falco** | Runtime security monitoring |

---

## Hardware

- Raspberry Pi 5 — 8GB RAM
- NVMe SSD — 500GB (boot drive, via M.2 HAT)
- Ubuntu Server 24.04 LTS 64-bit

---

## What you get

- Every service accessible at `https://service.yourdomain.com`
- SSL certificates issued and renewed automatically
- All subdomains protected by Cloudflare Zero Trust email auth
- Your domain always points to your cluster even when your ISP changes your IP
- Full cluster monitoring via Grafana
- Workflow automation via n8n
- Runtime security via Falco

---

## Architecture

```
Internet
    ↓
Cloudflare (DNS proxy + Zero Trust auth)
    ↓
Your router (DMZ → ingress IP)
    ↓
MetalLB LoadBalancer IP
    ↓
ingress-nginx
    ↓
Your services (Grafana, n8n, etc.)
```

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
10. [10 — Cloudflare Zero Trust](11-cloudflare-zero-trust.md)
11. [11 — DNS-01 cert-manager](12-dns01-cert-manager.md)
12. [12 — Prometheus + Grafana](13-prometheus-grafana.md)
13. [13 — n8n](14-n8n.md)
14. [14 — Security Hardening](15-security-hardening.md)

---

!!! tip "Multi-node"
    This setup supports multiple worker nodes. The control plane runs on one RPi5, workers can be added by running the k3s agent install command with the server token.
