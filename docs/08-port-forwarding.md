# 08 — Router Port Forwarding

Forward ports 80 and 443 from your router to the ingress-nginx IP. Without this, your services are not reachable from the internet and Let's Encrypt cannot issue SSL certificates.

---

## How it works

```
Internet → Router (public IP) → ingress-nginx (192.168.x.201) → Pod
```

Your RPi lives on your local network. Port forwarding tells the router to send all incoming traffic on ports 80 and 443 to the ingress IP assigned by MetalLB.

---

## Rules to create

| Rule name | External port | Internal host | Internal port | Protocol |
|---|---|---|---|---|
| `k8s-http` | 80 | ingress-nginx EXTERNAL-IP | 80 | TCP |
| `k8s-https` | 443 | ingress-nginx EXTERNAL-IP | 443 | TCP |

Get your ingress IP if you don't have it noted:

```bash
kubectl get svc -n ingress-nginx
```

---

## Router setup (general)

1. Log in to your router admin panel — usually at `http://192.168.x.1`
2. Look for a section named **Port Forwarding**, **Port Mapping**, **NAT**, or **Virtual Servers**
3. Create two rules as shown in the table above
4. Save and apply

---

## Verify end-to-end

```bash
curl -I http://yourdomain.com
```

Expected — a `404` from ingress-nginx confirms traffic is flowing correctly:

```
HTTP/1.1 404 Not Found
Server: nginx
```

You can also test from a phone on mobile data (not WiFi) to confirm external access.

---

!!! warning "Never expose SSH"
    Do not forward port 22. Use Cloudflare Tunnel for remote SSH access instead.

---

!!! success "Ready"
    Proceed to [09 — Deploying a Service](09-deploying-a-service.md)
