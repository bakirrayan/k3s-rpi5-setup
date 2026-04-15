# K3s on Raspberry Pi 5

Production-ready single-node Kubernetes cluster setup — with ingress, SSL, persistent storage and dynamic DNS.

## Stack

| Component | Purpose |
|---|---|
| K3s | Lightweight certified Kubernetes (etcd backend) |
| MetalLB | Bare-metal load balancer |
| ingress-nginx | Subdomain routing |
| cert-manager | Automatic SSL via Let's Encrypt |
| Longhorn | Persistent NVMe storage |
| Cloudflare | DNS proxy + DDoS protection |
| ddclient | Dynamic DNS updater |

## Hardware

- Raspberry Pi 5 — 8GB RAM
- NVMe SSD — 500GB
- Ubuntu Server 24.04 LTS 64-bit

## Documentation

Full step-by-step guides available at the GitHub Pages site.

## Guides

| # | Guide |
|---|---|
| 01 | OS Preparation |
| 02 | K3s Installation |
| 03 | MetalLB |
| 04 | ingress-nginx |
| 05 | cert-manager |
| 06 | Longhorn Storage |
| 07 | Cloudflare DNS + DDNS |
| 08 | Router Port Forwarding |
| 09 | Deploying a Service |
| 10 | WireGuard VPN |
| 11 | Cloudflare Zero Trust |
| 12 | DNS-01 cert-manager |