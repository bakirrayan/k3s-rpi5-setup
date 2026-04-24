# 06 — Longhorn Storage

Longhorn provides persistent block storage for your cluster using the NVMe SSD. It becomes the default StorageClass — all workloads that need persistent volumes (n8n, Grafana, Prometheus, etc.) will use it automatically.

---

## Step 1 — Install dependencies

```bash
sudo apt install -y open-iscsi nfs-common jq
sudo systemctl enable iscsid
sudo systemctl start iscsid
```

Verify iSCSI is running:

```bash
sudo systemctl status iscsid
```

---

## Step 2 — Install Longhorn

```bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.8.1/deploy/longhorn.yaml
```

Watch pods come up (takes 2–3 minutes):

```bash
kubectl get pods -n longhorn-system -w
```

Press `Ctrl+C` when all pods are stable.

---

## Step 3 — Set Longhorn as default StorageClass

```bash
kubectl patch storageclass local-path \
  -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'

kubectl patch storageclass longhorn \
  -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

---

## Step 4 — Verify

```bash
kubectl get storageclass
```

Expected:

```
NAME                 PROVISIONER             DEFAULT
longhorn (default)   driver.longhorn.io      true
local-path           rancher.io/local-path   false
longhorn-static      driver.longhorn.io
```

---

## Step 5 — Expose Longhorn UI

Create an ingress to access the Longhorn dashboard at `https://longhorn.yourdomain.com`:

```bash
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn
  namespace: longhorn-system
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - longhorn.yourdomain.com
    secretName: longhorn-tls
  rules:
  - host: longhorn.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: longhorn-frontend
            port:
              number: 80
EOF
```

Cloudflare Zero Trust protects it automatically via the wildcard policy.

---

## Step 6 — Set replica count for single node

By default Longhorn creates 3 replicas per volume. On a single node cluster this causes all volumes to show as **Degraded**.

Fix globally: Longhorn UI → **Settings** → `Default Replica Count` → set to `1`.

Fix per volume: click the volume → **Update Replicas** → set to `1`.

---

## Step 7 — Enable Prometheus metrics

Create a ServiceMonitor so Prometheus scrapes Longhorn metrics. This enables the Longhorn Grafana dashboards (IDs 16888 and 22705):

```bash
kubectl apply -f - <<EOF
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: longhorn
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
spec:
  selector:
    matchLabels:
      app: longhorn-manager
  namespaceSelector:
    matchNames:
    - longhorn-system
  endpoints:
  - port: manager
    path: /metrics
EOF
```

Verify Prometheus picked it up:

```bash
kubectl get servicemonitor -n monitoring
```

Then in Grafana → **Dashboards** → **Import** → enter ID `16888` or `22705` to get the Longhorn dashboards.

!!! note
    Allow 1-2 minutes after creating the ServiceMonitor for Prometheus to start scraping and data to appear in Grafana.

---

!!! note "Disk usage"
    Longhorn uses available free space on your NVMe SSD. With a 500GB drive and Ubuntu using ~5GB, you have ~495GB available for persistent volumes.

---

!!! success "Ready"
    Proceed to [07 — Cloudflare DNS + DDNS](07-cloudflare-ddns.md)