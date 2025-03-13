## Chapter 8: Kubernetes Services

**Key Objectives:**
- Master the art of service creation and management in Kubernetes.
- Explore different service types and their unique use cases.
- Understand Kubernetes DNS and service discovery mechanisms.
- Learn about headless services and their use cases.
- Gain hands-on experience connecting your applications to the world.
- Learn common troubleshooting methods for diagnosing service-related issues.

---

## 1. Understanding Services

*Think of services as gatekeepers for your containerized applications.* They provide a stable, reliable endpoint to ephemeral pods, simplifying communication and external access. Key features include:

- **Pod Abstraction:** Access pods via stable service names rather than individual IPs.
- **Load Balancing:** Automatically distributes traffic across pod replicas.
- **Service Discovery:** Kubernetes automatically generates DNS entries for easy service location.
- **Health Checks:** Ensures only healthy pods receive traffic.

---

## 2. Kubernetes Service Types

| Type             | Description                                        | Use Case                               |
|------------------|----------------------------------------------------|----------------------------------------|
| **ClusterIP**    | Internal-only access                               | Microservice communication             |
| **NodePort**     | Exposes service externally via Node IP and Port    | Simple external access                 |
| **LoadBalancer** | Cloud-provider external load balancer              | Public-facing applications             |
| **ExternalName** | Maps to external DNS name (no proxying)            | External service access                |
| **Headless**     | Direct pod IP discovery (no ClusterIP)             | Stateful apps needing direct pod access|

---

## 3. Creating Services

**YAML Manifest Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  labels:
    app: redis
spec:
  selector:
    app: redis
  ports:
    - port: 6379
      targetPort: 6379
  type: ClusterIP
```

Apply with:
```bash
kubectl apply -f redis-service.yaml
```

**Using `kubectl` CLI:**
```bash
kubectl create service clusterip redis --selector=app=redis --port=6379
```

### Hands-on: Creating a NodePort Service

**YAML Manifest Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30080
```

Apply with:
```bash
kubectl apply -f web-nodeport.yaml
```

Test external access:
```bash
curl http://<NodeIP>:30080
```

### Hands-on: Creating a ClusterIP Service

**YAML Manifest Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  type: ClusterIP
  selector:
    app: api
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
```

Apply with:
```bash
kubectl apply -f api-clusterip.yaml
```

Test internal access:
```bash
kubectl exec -it <pod-name> -- curl http://api-service:5000
```

---

## 4. Common Parameters in Kubernetes Services

**Detailed YAML Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service          # Unique name
  namespace: default        # Optional
  labels:
    app: myapp              # Organizational labels
spec:
  type: NodePort            # ClusterIP, NodePort, LoadBalancer, ExternalName
  selector:
    app: myapp              # Matches pod labels
  ports:
    - protocol: TCP         # TCP/UDP
      port: 80              # Service Port
      targetPort: 8080      # Pod Port
      nodePort: 30080       # External NodePort (NodePort only)
  clusterIP: None           # None for headless; otherwise auto-assigned
```

---

## 5. Headless Services

A **headless service** exposes pods directly without proxying, useful for stateful apps.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-db
spec:
  selector:
    app: database
  clusterIP: None
  ports:
    - port: 5432
      targetPort: 5432
```

- **Pod DNS:**
```
<pod-hostname>.<service-name>.<namespace>.svc.cluster.local
```

Example:
```
db-0.headless-db.default.svc.cluster.local
```

---

## 6. Kubernetes DNS & Service Discovery

Automatically generated DNS entries:

- **Service DNS:**
```
<service-name>.<namespace>.svc.cluster.local
```
Example:
```
redis-cache.default.svc.cluster.local
```

- **Headless Service DNS (Pods):**
```
<pod-hostname>.<service-name>.<namespace>.svc.cluster.local
```

---

## 7. Managing & Inspecting Services

- List services:
```bash
kubectl get services
```

- Describe service:
```bash
kubectl describe service <service-name>
```

- Delete service:
```bash
kubectl delete service <service-name>
```

---

## 8. Troubleshooting Kubernetes Services

Common issues and fixes:

| Problem                                    | Solution                                                             |
|--------------------------------------------|-------------------------------------------------------------------|
| NodePort not accessible externally         | Check firewall/security groups allow node port                      |
| Pods not receiving traffic                 | Verify pod readiness/liveness probes                                |
| LoadBalancer IP not provisioned            | Check cloud provider events, validate service configuration         |
| DNS resolution problems                   | Check `coredns` pod status and configuration                        |

---

## 9. Advanced Concepts

### Accessing Services from Inside a Pod

Pods can access services internally through Kubernetes DNS.

**Hands-on Example:**

Deploy a debugging pod:
```bash
kubectl run debug-pod --image=busybox --restart=Never -- sleep 3600
```

Check service access from the pod:
```bash
kubectl exec -it debug-pod -- curl http://api-service:5000
```

**Use Cases:**
- Validate service connectivity from application pods.
- Troubleshoot connectivity issues within the cluster.
- Confirm service discovery is working correctly.

- **Network Policies:** Restrict pod-to-pod communication for enhanced security.

---

## 10. Chapter Summary

Services simplify pod communication and discovery by providing stable endpoints. Understanding the various service types and their appropriate use cases helps manage and expose applications effectively. Hands-on practice creating and troubleshooting services will solidify your Kubernetes expertise.
