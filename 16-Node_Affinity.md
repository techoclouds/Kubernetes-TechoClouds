Here's a structured, detailed, and fully hands-on **Chapter 16: Node Affinity and Anti-Affinity in Kubernetes**, complete with examples and clear explanations:

---

# Chapter 16: Node Affinity and Anti-Affinity in Kubernetes

**Key Objectives:**
- Understand Node Affinity and Anti-Affinity concepts.
- Gain precise control over pod scheduling on specific nodes.
- Learn to schedule pods based on node attributes like labels.
- Apply hands-on examples and troubleshooting methods.

---

## 1. Introduction to Node Affinity and Anti-Affinity

**Node Affinity** allows pods to be scheduled onto specific nodes based on **node labels** and rules. This helps in scenarios like:

- Scheduling CPU-intensive tasks to high-performance nodes.
- Ensuring region-specific or availability zone deployments.
- Dedicated nodes for special workloads (GPU workloads, database servers).

**Node Anti-Affinity** allows pods to avoid specific nodes, improving reliability, resource distribution, and workload isolation.

---

## 2. Node Affinity Explained

Node Affinity provides two methods for pod scheduling:

- **requiredDuringSchedulingIgnoredDuringExecution** (Hard Rule)
  - Pods **must** meet these criteria at scheduling time; otherwise, they won't be scheduled.

- **preferredDuringSchedulingIgnoredDuringExecution** (Soft Rule)
  - Kubernetes attempts to meet the criteria; if no nodes satisfy, pods can still schedule elsewhere.

### YAML Structure of Node Affinity

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:   # Hard rule
        nodeSelectorTerms:
        - matchExpressions:
          - key: gpu
            operator: In
            values:
            - "true"
      preferredDuringSchedulingIgnoredDuringExecution:  # Soft rule
      - weight: 1
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - "us-east-1a"
```

---

## 3. Node Affinity Operators

Common operators used in affinity:

| Operator | Description                                 | Example                      |
|----------|---------------------------------------------|------------------------------|
| In       | Node label matches any listed value         | zone in (us-east-1a,us-east-1b) |
| NotIn    | Node label must not match listed values     | zone not in (us-west-1a)     |
| Exists   | Node must have the specified key (any value)| gpu exists                   |
| DoesNotExist| Node must not have the specified key     | gpu does not exist           |
| Gt, Lt   | Greater than, Less than (numeric labels)    | cpu > 4                      |

---

## 4. Hands-on Example: Node Affinity (Complete)

### Step 1: Label Node(s)

First, label your node(s):
```bash
kubectl label nodes worker-node-1 gpu=true region=us-east
kubectl label nodes worker-node-2 region=us-west
```

Verify labels:
```bash
kubectl get nodes --show-labels
```

### Step 2: Pod YAML with Node Affinity (Required)

Create a Pod to run strictly on GPU nodes:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-workload
spec:
  containers:
  - name: cuda-container
    image: nvidia/cuda
    command: ["sleep", "3600"]
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: gpu
            operator: In
            values:
            - "true"
```

Deploy Pod:
```bash
kubectl apply -f gpu-pod.yaml
kubectl get pods -o wide
```

---

## 5. Soft Affinity (Preferred) Example

Schedule Pod preferably on nodes in a specific region (`us-east`), but allow elsewhere if not available:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: preferred-region-app
spec:
  containers:
  - name: app
    image: nginx
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: region
            operator: In
            values:
            - us-east
```

Deploy Pod:
```bash
kubectl apply -f region-preferred-pod.yaml
kubectl get pods -o wide
```

---

## 6. Node Anti-Affinity

Anti-Affinity repels pods from nodes with certain labels.

**Scenario:** Avoid scheduling pods onto maintenance nodes.

### Label Maintenance Node:
```bash
kubectl label node worker-node-3 maintenance=true
```

### Pod YAML with Anti-Affinity:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: avoid-maintenance
spec:
  containers:
  - name: app
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: maintenance
            operator: NotIn
            values:
            - "true"
```

Deploy Pod:
```bash
kubectl apply -f anti-affinity-pod.yaml
kubectl get pods -o wide
```

---

## 7. Node Affinity vs Node Selector

| **Node Affinity**                          | **Node Selector**                       |
|--------------------------------------------|-----------------------------------------|
| Complex rules with multiple operators      | Simple equality-based rules             |
| Supports both hard & soft rules            | Only hard rules                         |
| Recommended for flexible scheduling        | Simple, legacy method                   |

---

## 8. Text-Based Diagram of Node Affinity/Anti-Affinity Scheduling Flow

```
 Pod Scheduling Request
          │
          ▼
  ┌───────────────────────────────────────────┐
  │ Check requiredDuringScheduling (hard rules)│
  └───────────────┬───────────────────────────┘
                  │ Yes                        │ No
                  ▼                            ▼
          Pod scheduled               Pod remains pending
                  │
                  ▼
  ┌───────────────────────────────────────────────┐
  │ Check preferredDuringScheduling (soft rules)  │
  └───────────────┬───────────────────────────────┘
                  │ Yes                           │ No
                  ▼                               ▼
         Pod scheduled ideally             Pod scheduled elsewhere
```

---

## 9. Troubleshooting Node Affinity Issues

Common issues and how to fix:

- **Pod stuck in pending:**  
  - Check node labels:  
  ```bash
  kubectl get nodes --show-labels
  ```
  - Verify your affinity rules match labels.

- **Pod scheduling on unexpected node:**  
  - Double-check the affinity definition (operator/value mistakes).

- **No nodes matching affinity:**  
  - Revisit labels or adjust affinity rules.

---

## 10. Best Practices for Node Affinity

- Clearly document node labels and affinity rules.
- Prefer soft affinity (`preferredDuringScheduling`) for greater flexibility.
- Avoid overly complicated affinity rules to simplify cluster management.

---

## 11. Quick Reference Cheat Sheet

| Command or YAML                        | Usage Example                                 |
|----------------------------------------|-----------------------------------------------|
| Label node                             | `kubectl label node node1 gpu=true`           |
| Remove node label                      | `kubectl label node node1 gpu-`               |
| Node Affinity YAML structure           | `spec.affinity.nodeAffinity.requiredDuringScheduling...` |
| Check Node Labels                      | `kubectl get nodes --show-labels`             |

---

## 12. Chapter Summary

Node Affinity and Anti-Affinity help precisely control pod scheduling onto nodes using labels, improving resource utilization, reliability, and workload isolation. By practicing with the hands-on examples and clearly understanding affinity rules, you'll effectively manage Kubernetes workloads and achieve advanced scheduling configurations.

---
