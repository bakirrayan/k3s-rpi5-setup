# 15 — Security Hardening

A checklist of security measures applied to the cluster beyond the base setup.

---

## Cloudflare

- **SSL/TLS mode** — set to **Full (strict)** in Cloudflare → SSL/TLS → Overview
- **Always Use HTTPS** → On
- **Minimum TLS Version** → TLS 1.2
- **Zero Trust** — all subdomains protected by email authentication (see [11 — Cloudflare Zero Trust](11-cloudflare-zero-trust.md))

---

## SSH

- Password authentication disabled — SSH key only
- Key type: `ed25519`

```bash
sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
sudo systemctl restart sshd
```

---

## Kubernetes network policies

Namespace isolation — only ingress-nginx and monitoring can reach pods. Apply to every namespace:

```bash
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: isolate-namespace
  namespace: YOUR_NAMESPACE
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

!!! note
    Don't add egress restrictions to namespaces that need external API access (e.g. n8n). Only restrict ingress.

---

## Falco — runtime security

Falco monitors all syscalls across the cluster and alerts on suspicious behavior — unexpected shell access, privilege escalation, sensitive file reads, etc.

```bash
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
kubectl create namespace falco
helm install falco falcosecurity/falco \
  --namespace falco \
  --set falco.grpc.enabled=true \
  --set falco.grpcOutput.enabled=true \
  --set driver.kind=modern_ebpf
```

!!! note
    Use `modern_ebpf` driver on RPi5 — more compatible with the ARM64 kernel than the kernel module driver.

Verify:

```bash
kubectl get pods -n falco
kubectl logs -n falco daemonset/falco --tail=20
```

---

## Grafana

Change the default admin password after install:

```bash
kubectl exec -n monitoring deployment/kube-prometheus-stack-grafana -- \
  grafana-cli admin reset-admin-password yournewstrongpassword
```

---

## Todo / future hardening

- [ ] Longhorn PVC backups to S3-compatible storage
- [ ] Kubernetes audit logging
- [ ] Resource limits on all deployments
- [ ] Renovate bot for automatic image update PRs
- [ ] Pod Security Admission policies

---

!!! success "Done"
    Your cluster is hardened. Return to [Home](index.md) to deploy more services.