# 11 — Cloudflare Zero Trust

Protect all your subdomains behind Cloudflare Access — anyone trying to reach a service must authenticate with their email first. Combined with WireGuard, this is the most secure setup for a homelab.

---

## How it works

```
User visits subdomain
        ↓
Cloudflare Access — email authentication required
        ↓
One-time code sent to email
        ↓
User authenticated — request reaches your cluster
```

No credentials exposed, no brute force possible, no IP restrictions needed.

---

## Step 1 — Open Zero Trust

Cloudflare dashboard → **Zero Trust** in the left sidebar (or `one.dash.cloudflare.com`)

---

## Step 2 — Create the main application

Go to **Access** → **Applications** → **Add an application** → **Self-hosted and private**

| Field | Value |
|---|---|
| Application name | `K3s Services` |
| Session duration | `24h` |
| Subdomain | `*` |
| Domain | `yourdomain.com` |

Click **Next**.

---

## Step 3 — Add an Allow policy

| Field | Value |
|---|---|
| Policy name | `Allowed users` |
| Action | `Allow` |
| Include rule | `Emails` |
| Email values | list every email you want to allow |

Click **Save policy** → **Save application**.

---

## Step 4 — Add a Bypass for the VPN subdomain

The VPN login page must remain publicly accessible — otherwise users can't reach it to authenticate and get VPN access.

**Add another application** → **Self-hosted and private**:

| Field | Value |
|---|---|
| Application name | `VPN bypass` |
| Subdomain | `vpn` |
| Domain | `yourdomain.com` |

Click **Next** → **Add a policy**:

| Field | Value |
|---|---|
| Policy name | `Bypass` |
| Action | `Bypass` |
| Include | `Everyone` |

Click **Save policy** → **Save application**.

---

## Step 5 — Test

Open a private/incognito browser and visit any protected subdomain. You should be redirected to a Cloudflare login page asking for your email. Enter an allowed email, receive a one-time code, and you're in.

---

## Troubleshooting

**526 Invalid SSL certificate** — this means Cloudflare reached your server but found no valid certificate. This happens when a subdomain has no deployed service yet. Either deploy the service or temporarily set SSL mode to **Full** (not strict) in Cloudflare SSL/TLS settings until the service is deployed.

**Redirect loop** — make sure the VPN bypass policy uses `Bypass` action with `Everyone`, not `Allow`.

---

!!! tip "Session duration"
    Set session duration to `24h` for convenience or `1h` for stricter security. Users will need to re-authenticate after the session expires.

---

!!! success "Ready"
    Your cluster is now fully secured. Proceed to deploying your first service.
