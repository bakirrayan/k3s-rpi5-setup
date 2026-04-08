# 02 — K3s Installation

Install K3s with a production-grade configuration — etcd backend, no built-in Traefik or ServiceLB.

---

## Why these flags

| Flag | Reason |
|---|---|
| `--disable traefik` | Replaced by ingress-nginx |
| `--disable servicelb` | Replaced by MetalLB |
| `--cluster-init` | Uses etcd instead of SQLite — more robust, expandable |
| `--write-kubeconfig-mode 644` | Allows kubectl without sudo |

---

## Step 1 — Install K3s

```bash
curl -sfL https://get.k3s.io | sh -s - \
  --disable traefik \
  --disable servicelb \
  --write-kubeconfig-mode 644 \
  --cluster-init
```

---

## Step 2 — Configure kubectl

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
echo 'export KUBECONFIG=~/.kube/config' >> ~/.bashrc
source ~/.bashrc
```

---

## Step 3 — Install Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## Step 4 — Verify

```bash
kubectl get nodes
```

Expected output:

```
NAME        STATUS   ROLES                       AGE   VERSION
your-rpi5   Ready    control-plane,etcd,master   1m    v1.x.x+k3s1
```

```bash
kubectl get pods -A
```

All system pods should be `Running`.

---

!!! success "Ready"
    Proceed to [03 — MetalLB](03-metallb.md)
