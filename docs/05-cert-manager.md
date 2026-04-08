# 05 — cert-manager

cert-manager automates the issuance and renewal of SSL certificates from Let's Encrypt for all your subdomains.

---

## Step 1 — Install

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.17.2/cert-manager.yaml
```

---

## Step 2 — Wait for pods

```bash
kubectl wait --namespace cert-manager \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/instance=cert-manager \
  --timeout=120s
```

---

## Step 3 — Verify pods

```bash
kubectl get pods -n cert-manager
```

Expected — all 3 pods `Running`:

```
cert-manager-xxxx              1/1   Running
cert-manager-cainjector-xxxx   1/1   Running
cert-manager-webhook-xxxx      1/1   Running
```

---

## Step 4 — Create ClusterIssuer

!!! warning
    Replace `your@email.com` with your real email. Let's Encrypt uses it for certificate expiry notices and account communication.

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
    - http01:
        ingress:
          ingressClassName: nginx
EOF
```

---

## Step 5 — Verify issuer is ready

```bash
kubectl get clusterissuer
```

Expected:

```
NAME               READY   AGE
letsencrypt-prod   True    1m
```

---

## Fix wrong email

If you applied with the wrong email, delete and reapply:

```bash
kubectl delete clusterissuer letsencrypt-prod
# reapply the manifest above with the correct email
```

---

!!! success "Ready"
    Proceed to [06 — Longhorn Storage](06-longhorn.md)
