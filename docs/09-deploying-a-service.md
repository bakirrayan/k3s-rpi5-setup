# 09 — Deploying a Service

Use this template to deploy any application with subdomain routing and automatic SSL.

---

## How SSL works

When you create an Ingress with the `cert-manager.io/cluster-issuer` annotation, cert-manager automatically:

1. Detects the new Ingress
2. Requests a certificate from Let's Encrypt
3. Stores the certificate as a Kubernetes Secret
4. Renews it automatically before expiry

---

## Ingress template

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
  namespace: my-app
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - my-app.yourdomain.com
    secretName: my-app-tls
  rules:
  - host: my-app.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app
            port:
              number: 8080
```

Replace:

- `my-app` → your app name
- `my-app.yourdomain.com` → your subdomain
- `8080` → the port your app container listens on

---

## Check certificate status

```bash
kubectl get certificate -n my-app
```

Expected after a minute or two:

```
NAME         READY   SECRET       AGE
my-app-tls   True    my-app-tls   2m
```

If `READY` stays `False`, check events:

```bash
kubectl describe certificate my-app-tls -n my-app
```

---

## Security checklist before exposing any service

- [ ] Service has authentication (login page or basic auth ingress annotation)
- [ ] Sensitive dashboards (Longhorn, Prometheus) restricted by IP or basic auth
- [ ] Cloudflare proxy (orange cloud) is ON for all DNS records
- [ ] SSL/TLS mode is **Full (strict)** in Cloudflare
- [ ] SSH port (22) is NOT forwarded on the router

---

## Add basic auth to an Ingress (optional)

For services without their own login page:

```bash
# Create credentials
sudo apt install -y apache2-utils
htpasswd -c auth admin
kubectl create secret generic basic-auth \
  --from-file=auth \
  -n my-app
```

Add to Ingress annotations:

```yaml
annotations:
  nginx.ingress.kubernetes.io/auth-type: basic
  nginx.ingress.kubernetes.io/auth-secret: basic-auth
  nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
```

---

!!! tip "Wildcard DNS"
    Since Cloudflare has a wildcard `*` A record for your domain, every new subdomain works automatically — no need to add DNS records for each new service.
