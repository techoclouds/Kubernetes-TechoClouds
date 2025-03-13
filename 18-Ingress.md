
# Chapter 18: Kubernetes Ingress – Routing External Traffic

## Key Objectives:
- Clearly understand Kubernetes Ingress and its role.
- Explore popular Ingress Controllers.
- Hands-on installation and configuration of Nginx Ingress Controller.
- Detailed YAML examples demonstrating host-based routing and path-based routing.
- Deep dive into `pathType`: Prefix, Exact, ImplementationSpecific.
- Configure SSL/TLS termination using Ingress.
- Learn troubleshooting techniques for Ingress resources.

---

## 1. Understanding Kubernetes Ingress

**Ingress** provides HTTP/HTTPS routing into Kubernetes clusters, enabling centralized traffic management, SSL termination, name-based virtual hosting, and flexible routing rules.

**Ingress advantages over NodePort or LoadBalancer:**
- Flexible URL-based routing.
- SSL/TLS termination at ingress level.
- Simplified external traffic management.

---

## 2. Ingress Controllers Explained

An **Ingress Controller** fulfills Ingress resources' routing rules and manages external traffic.

### Popular Ingress Controllers:

| Controller | Description                                                      |
|------------|------------------------------------------------------------------|
| **Nginx**  | Most popular, robust, easy configuration, well-supported         |
| **Traefik**| Dynamic configuration, integrates automatic SSL (Let’s Encrypt)  |
| **Istio**  | Advanced service mesh, additional features for security and monitoring |
| **HAProxy**| Reliable, high-performance, suitable for heavy workloads         |

This chapter uses **Nginx Ingress Controller** for practical demonstrations.

---

## 3. Installing Nginx Ingress Controller (Hands-On)

Deploy using official YAML manifest:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.6/deploy/static/provider/cloud/deploy.yaml
```

Verify installation:

```bash
kubectl get pods -n ingress-nginx
```

---

## 4. Hands-On: Host-Based Routing Example

Host-based routing directs traffic to services based on the domain requested.

### YAML Definition:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-ingress
spec:
  rules:
    - host: app1.example.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app1-service
                port:
                  number: 80
    - host: app2.example.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app2-service
                port:
                  number: 80
```

Apply and verify:

```bash
kubectl apply -f host-based-ingress.yaml

curl -H "Host: app1.example.local" http://<Ingress_Controller_IP>
curl -H "Host: app2.example.local" http://<Ingress_Controller_IP>
```

---

## 5. Hands-On: Path-Based Routing with Different `pathType`

Kubernetes supports three `pathType` matching options:

- **Prefix**: Matches URLs beginning with specified path.
- **Exact**: Matches exact URL.
- **ImplementationSpecific**: Depends on Ingress controller; typically prefix-based.

### YAML Demonstration:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-type-demo-ingress
spec:
  rules:
    - host: example.local
      http:
        paths:
          - path: /app
            pathType: Prefix
            backend:
              service:
                name: prefix-service
                port:
                  number: 80

          - path: /exactmatch
            pathType: Exact
            backend:
              service:
                name: exact-service
                port:
                  number: 80

          - path: /special
            pathType: ImplementationSpecific
            backend:
              service:
                name: special-service
                port:
                  number: 80
```

Apply YAML and test paths:

```bash
kubectl apply -f path-type-demo-ingress.yaml
```

---

## 6. YAML Deep Dive: Key Parameters & Examples

| Parameter          | Purpose                                                     |
|--------------------|-------------------------------------------------------------|
| **rules**          | Routing rules (host/path based)                             |
| **host**           | Domain name matching                                        |
| **paths**          | URL paths within a host                                     |
| **pathType**       | Matching method (`Prefix`, `Exact`, `ImplementationSpecific`)|
| **backend**        | Destination service                                         |
| **defaultBackend** | Handles traffic not matching defined rules                  |

### Default Backend Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: default-backend-ingress
spec:
  defaultBackend:
    service:
      name: fallback-service
      port:
        number: 80
```

---

## 7. SSL/TLS Termination with Ingress (Hands-On)

Generate self-signed certificates:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=example.local/O=example.local"

kubectl create secret tls example-tls --key tls.key --cert tls.crt
```

Ingress YAML with TLS enabled:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
spec:
  tls:
    - hosts:
        - example.local
      secretName: example-tls
  rules:
    - host: example.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: secure-service
                port:
                  number: 80
```

Apply and test TLS:

```bash
kubectl apply -f secure-ingress.yaml
curl -k https://example.local -H "Host: example.local"
```


## 8. Troubleshooting Kubernetes Ingress

**Ingress details & events:**

```bash
kubectl describe ingress <ingress-name>
```

**Check Ingress Controller logs:**

```bash
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller
```

**Verify backend services:**

```bash
kubectl get svc
kubectl get endpoints <service-name>
```

### Common Issues:

- Incorrect backend service names or ports.
- Mismatch in host or path rules.
- Certificate problems for TLS.
```
