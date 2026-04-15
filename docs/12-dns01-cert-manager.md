# 12 — Switch cert-manager to DNS-01

By default cert-manager uses HTTP-01 challenges to verify domain ownership. This fails in two common scenarios:

- **Country WAF rules** block Let's Encrypt servers (non-Algerian IPs)
- **Hairpin NAT** — your router doesn't support loopback traffic (cert-manager self-check fails)

DNS-01 solves both — it verifies ownership by creating a DNS TXT record via the Cloudflare API. No HTTP connection to your server needed.

---

## Step 1 — Create a Cloudflare API token

1. Cloudflare dashboard → **My Profile** → **API Tokens**
2. Click **Create Token** → use **Edit zone DNS** template
3. Under **Zone Resources** → select your domain
4. Click **Continue** → **Create Token**
5. Copy the token — you only see it once

---

## Step 2 — Store the token as a Kubernetes secret

```bash
kubectl create secret generic cloudflare-api-token \
  --from-literal=api-token=YOUR_CLOUDFLARE_API_TOKEN \
  -n cert-manager
```

---

## Step 3 — Replace letsencrypt-prod with DNS-01 version

```bash
kubectl delete clusterissuer letsencrypt-prod
```

```bash
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your@email.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - dns01:
        cloudflare:
          apiTokenSecretRef:
            name: cloudflare-api-token
            key: api-token
EOF
```

---

## Step 4 — Create a second DNS-01 issuer (optional but recommended)

Useful for namespaces or services that use a different issuer name:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-dns
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your@email.com
    privateKeySecretRef:
      name: letsencrypt-dns
    solvers:
    - dns01:
        cloudflare:
          apiTokenSecretRef:
            name: cloudflare-api-token
            key: api-token
EOF
```

---

## Step 5 — Verify both issuers are ready

```bash
kubectl get clusterissuer
```

Expected:

```
NAME               READY
letsencrypt-prod   True
letsencrypt-dns    True
```

---

## Step 6 — Force re-issue existing certificates

If you have existing certificates that were issued with HTTP-01, delete and let them re-issue:

```bash
kubectl delete certificate <cert-name> -n <namespace>
```

The ingress controller will automatically trigger a new certificate request using DNS-01.

---

!!! note "Cloudflare proxy during issuance"
    DNS-01 works with the Cloudflare proxy (orange cloud) enabled and with any WAF country rules active — the verification is entirely DNS-based, not HTTP-based.
