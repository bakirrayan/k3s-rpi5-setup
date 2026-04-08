# 03 — MetalLB

MetalLB is a bare-metal load balancer. It assigns real LAN IP addresses to your Kubernetes services — something that doesn't exist by default outside of cloud providers.

---

## Step 1 — Install via manifest

!!! note
    If Helm times out downloading from GitHub, use the manifest method below — it is more reliable on restricted networks.

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.15.3/config/manifests/metallb-native.yaml
```

---

## Step 2 — Wait for pods

```bash
kubectl wait --namespace metallb-system \
  --for=condition=ready pod \
  --selector=app=metallb \
  --timeout=90s
```

---

## Step 3 — Configure IP pool

!!! warning
    Choose an IP range that is **outside your router's DHCP pool** to avoid conflicts. Check your router settings to confirm the DHCP range before proceeding.

```bash
cat <<EOF | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.x.201-192.168.x.210   # adjust to your network
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default-pool
EOF
```

---

## Step 4 — Verify

```bash
kubectl get pods -n metallb-system
kubectl get ipaddresspool -n metallb-system
```

All pods should be `Running` and the IP pool should be listed.

---

!!! success "Ready"
    Proceed to [04 — ingress-nginx](04-ingress-nginx.md)
