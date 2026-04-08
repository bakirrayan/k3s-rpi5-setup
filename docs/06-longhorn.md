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

!!! note "Disk usage"
    Longhorn uses available free space on your NVMe SSD. With a 500GB drive and Ubuntu using ~5GB, you have ~495GB available for persistent volumes.

---

!!! success "Ready"
    Proceed to [07 — Cloudflare DNS + DDNS](07-cloudflare-ddns.md)
