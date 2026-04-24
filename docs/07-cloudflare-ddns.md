# 07 — Cloudflare DNS + DDNS

Point your domain to your cluster and keep it updated automatically when your ISP changes your public IP.

---

## Why Cloudflare

- Free DNS management with wildcard subdomain support
- Hides your real public IP behind Cloudflare's proxy
- Free DDoS protection
- API support for DDNS updates

---

## Step 1 — Add domain to Cloudflare

1. Create a free account at [cloudflare.com](https://cloudflare.com)
2. Click **Add a domain** → enter your domain → select the **Free plan**
3. Cloudflare scans and imports your existing DNS records
4. Note the two nameservers provided (e.g. `xxx.ns.cloudflare.com`)

---

## Step 2 — Update nameservers on your registrar

Log in to your domain registrar and replace the existing nameservers with the two Cloudflare nameservers from Step 1.

Verify propagation (can take 10 min to a few hours):

```bash
nslookup -type=NS yourdomain.com 8.8.8.8
```

Cloudflare will show the domain as **Active** once propagation is complete.

---

## Step 3 — Configure DNS records in Cloudflare

Get your public WAN IP:

```bash
curl ifconfig.me
```

Add these two A records in Cloudflare DNS:

| Type | Name | Value | Proxy |
|---|---|---|---|
| A | `@` | your WAN IP | Proxied (orange cloud) |
| A | `*` | your WAN IP | Proxied (orange cloud) |

The wildcard `*` covers all subdomains automatically — no need to add records per service.

---

## Step 4 — SSL/TLS settings

In Cloudflare → **SSL/TLS** → **Overview**:

- Set encryption mode to **Full (strict)**

In **SSL/TLS** → **Edge Certificates**:

- **Always Use HTTPS** → On
- **Minimum TLS Version** → TLS 1.2

---

## Step 5 — Create Cloudflare API token

1. Cloudflare dashboard → **My Profile** → **API Tokens**
2. Click **Create Token** → use **Edit zone DNS** template
3. Under **Zone Resources** → select your domain
4. Click **Continue** → **Create Token**
5. Copy the token — you only see it once

---

## Step 6 — Install DDNS updater on RPi

```bash
sudo apt install -y ddclient
```

When the interactive setup appears, press Enter through everything — the config will be overwritten.

Edit the config:

```bash
sudo nano /etc/ddclient.conf
```

Replace everything with:

```
daemon=300
syslog=yes
ssl=yes

protocol=cloudflare
zone=yourdomain.com
ttl=1
login=token
password=YOUR_CLOUDFLARE_API_TOKEN
usev4=webv4
webv4=ipify-ipv4
*.yourdomain.com,yourdomain.com
```

Enable and test:

```bash
sudo systemctl enable ddclient
sudo systemctl start ddclient
sudo ddclient -daemon=0 -debug -verbose -noquiet
```

Expected output:

```
SUCCESS: *.yourdomain.com -- Updated successfully to x.x.x.x
SUCCESS: yourdomain.com -- Updated successfully to x.x.x.x
```

!!! note
    "Skipped: IPv4 address was already set" is also a success — it means the IP in Cloudflare already matches.

---

## Fix — ddclient starts before DNS is ready

On boot, ddclient sometimes starts before the network DNS is fully available, causing it to fail silently. Add a startup delay:

```bash
sudo mkdir -p /etc/systemd/system/ddclient.service.d
sudo nano /etc/systemd/system/ddclient.service.d/override.conf
```

Add:

```ini
[Service]
ExecStartPre=/bin/sleep 30
```

Reload:

```bash
sudo systemctl daemon-reload
```

---

## DNS record strategy

Use only two A records in Cloudflare:

| Type | Name | Value | Proxy |
|---|---|---|---|
| A | `@` | your WAN IP | Proxied (orange cloud) |
| A | `*` | your WAN IP | Proxied (orange cloud) |

The wildcard `*` covers all subdomains automatically. Do not add explicit per-service records — they become stale when your IP changes unless also listed in ddclient.conf.

---

!!! success "Ready"
    Proceed to [08 — Router Port Forwarding](08-port-forwarding.md)
