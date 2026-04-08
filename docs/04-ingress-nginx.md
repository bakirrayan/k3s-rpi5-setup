# 04 — ingress-nginx

ingress-nginx is the reverse proxy and ingress controller. It receives all incoming traffic and routes it to the correct pod based on the hostname (subdomain).

---

## Step 1 — Install

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.1/deploy/static/provider/cloud/deploy.yaml
```

---

## Step 2 — Wait for controller

```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

---

## Step 3 — Verify external IP

```bash
kubectl get svc -n ingress-nginx
```

Expected output:

```
NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP       PORT(S)
ingress-nginx-controller   LoadBalancer   10.43.x.x      192.168.x.201     80:xxx/TCP,443:xxx/TCP
```

!!! note
    The `EXTERNAL-IP` is assigned by MetalLB from your IP pool. This is the IP you will use for port forwarding on your router — forward ports 80 and 443 to this address.

---

## What a 404 means

Once port forwarding is set up, visiting your domain will return a `404 Not Found` from ingress-nginx. This is expected — it means traffic is flowing correctly but no apps are deployed yet.

---

!!! success "Ready"
    Proceed to [05 — cert-manager](05-cert-manager.md)
