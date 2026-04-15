# 10 — WireGuard VPN (wg-easy)

Deploy a WireGuard VPN server on your cluster using wg-easy. This gives you a web UI to manage VPN clients and a secure tunnel into your cluster.

---

## Why WireGuard

- All services become accessible only through the VPN tunnel
- Simple client setup — QR code scan on mobile, config file on desktop
- Web UI for managing clients at `https://vpn.yourdomain.com`
- Lightweight and fast — built into the Linux kernel

---

## Step 1 — Deploy wg-easy

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: wg-easy
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wg-easy-pvc
  namespace: wg-easy
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wg-easy
  namespace: wg-easy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wg-easy
  template:
    metadata:
      labels:
        app: wg-easy
    spec:
      containers:
      - name: wg-easy
        image: ghcr.io/wg-easy/wg-easy
        env:
        - name: WG_HOST
          value: "vpn.yourdomain.com"
        - name: PASSWORD_HASH
          value: "YOUR_BCRYPT_HASH"
        - name: WG_DEFAULT_DNS
          value: "1.1.1.1"
        ports:
        - containerPort: 51820
          protocol: UDP
        - containerPort: 51821
          protocol: TCP
        securityContext:
          capabilities:
            add: ["NET_ADMIN", "SYS_MODULE"]
        volumeMounts:
        - name: wg-data
          mountPath: /etc/wireguard
      volumes:
      - name: wg-data
        persistentVolumeClaim:
          claimName: wg-easy-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: wg-easy
  namespace: wg-easy
spec:
  selector:
    app: wg-easy
  ports:
  - name: vpn
    port: 51820
    protocol: UDP
  - name: web
    port: 51821
    protocol: TCP
  type: LoadBalancer
EOF
```

---

## Step 2 — Generate bcrypt password hash

wg-easy v14+ requires a bcrypt hash instead of a plain text password:

```bash
htpasswd -bnBC 10 "" 'your-password' | tr -d ':\n'
```

Use the output as the value for `PASSWORD_HASH` in the deployment above.

To update an existing deployment:

```bash
kubectl set env deployment/wg-easy -n wg-easy PASSWORD_HASH='your-hash'
```

!!! warning
    Always use single quotes around the hash to prevent `$` signs from being interpreted by the shell.

---

## Step 3 — Forward WireGuard port on your router

Forward UDP port `51820` to the wg-easy LoadBalancer IP:

```bash
kubectl get svc -n wg-easy
```

Note the `EXTERNAL-IP` and forward UDP 51820 to it on your router.

!!! note
    This must be a User-defined UDP rule — there is no preset for WireGuard in most router UIs. If your router doesn't support custom UDP port forwarding, use DMZ pointed at the wg-easy LoadBalancer IP.

---

## Step 4 — Create Ingress for wg-easy web UI

```bash
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wg-easy
  namespace: wg-easy
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - vpn.yourdomain.com
    secretName: wg-easy-tls
  rules:
  - host: vpn.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wg-easy
            port:
              number: 51821
EOF
```

---

## Step 5 — Verify

```bash
kubectl get pods -n wg-easy
kubectl get svc -n wg-easy
kubectl get pvc -n wg-easy
kubectl get ingress -n wg-easy
kubectl get certificate -n wg-easy
```

All pods should be `Running`, PVC `Bound`, certificate `READY = True`.

---

## Step 6 — Add VPN clients

Go to `https://vpn.yourdomain.com` → log in → click **+ New Client** → name the client → scan the QR code with the WireGuard app or download the config file.

**WireGuard apps:**
- iOS / Android: WireGuard (App Store / Play Store)
- Windows / macOS: wireguard.com
- Linux: `sudo apt install wireguard`

---

!!! success "Ready"
    Proceed to [11 — Cloudflare Zero Trust](11-cloudflare-zero-trust.md)
