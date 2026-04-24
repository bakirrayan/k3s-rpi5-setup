# 13 — Prometheus + Grafana

Deploy cluster monitoring using the kube-prometheus-stack Helm chart. This bundles Prometheus, Grafana, Alertmanager, node-exporter, and kube-state-metrics in a single install.

---

## What you get

- Prometheus scraping metrics from all nodes and pods
- Grafana dashboards for CPU, RAM, pod health, and more
- Alertmanager for sending alerts
- node-exporter for per-node hardware metrics
- kube-state-metrics for Kubernetes object metrics

---

## Step 1 — Add Helm repo

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

## Step 2 — Create namespace

```bash
kubectl create namespace monitoring
```

---

## Step 3 — Install kube-prometheus-stack

```bash
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set grafana.ingress.enabled=true \
  --set grafana.ingress.ingressClassName=nginx \
  --set grafana.ingress.hosts[0]=grafana.yourdomain.com \
  --set grafana.ingress.tls[0].secretName=grafana-tls \
  --set grafana.ingress.tls[0].hosts[0]=grafana.yourdomain.com \
  --set grafana.adminPassword=yourchosenpassword
```

Replace `yourchosenpassword` with a strong password.

Watch pods come up:

```bash
kubectl get pods -n monitoring -w
```

All pods should reach `Running` state within a few minutes.

---

## Step 4 — Secure Grafana

Grafana is protected by Cloudflare Zero Trust email authentication — see [11 — Cloudflare Zero Trust](11-cloudflare-zero-trust.md).

Change the default admin password after first login:

```bash
kubectl exec -n monitoring deployment/kube-prometheus-stack-grafana -- \
  grafana-cli admin reset-admin-password yournewstrongpassword
```

---

## Step 5 — Add network policy

Restrict ingress to monitoring namespace:

```bash
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: isolate-namespace
  namespace: monitoring
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ingress-nginx
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: monitoring
EOF
```

---

## Step 6 — Verify

```bash
kubectl get pods -n monitoring
kubectl get ingress -n monitoring
kubectl get certificate -n monitoring
```

All pods `Running`, certificate `READY = True`, then open `https://grafana.yourdomain.com`.

---

!!! note
    Prometheus has no ingress by default — only accessible internally. This is intentional as Prometheus has no authentication.

---

!!! success "Ready"
    Proceed to [14 — n8n](14-n8n.md)