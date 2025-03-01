# Chapter 6: Understanding Deployments - Managing Your Kubernetes Applications

## 1. Introduction to Deployments
Kubernetes **Deployments** provide a way to manage application updates, ensure high availability, and automate scaling. They abstract lower-level resources like **ReplicaSets and Pods**, making app management easier.

### Key Features of Deployments
- **Declarative updates:** Define the desired state, and Kubernetes manages the rest.
- **Rolling updates:** New versions are released gradually without downtime.
- **Rollbacks:** If something goes wrong, roll back to the previous working version.
- **Scaling:** Easily scale up/down based on workload demand.
- **Self-healing:** Automatically restarts failed Pods.

---

## 2. Deployment YAML Structure (Basic Example)
To deploy an application, we define a **Deployment YAML**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: webserver
        image: nginx:1.19
        ports:
        - containerPort: 80
```

### Explanation of Key Fields
| **Field** | **Description** |
|-----------|---------------|
| `apiVersion: apps/v1` | Uses the Kubernetes `apps/v1` API. |
| `kind: Deployment` | Defines this as a Deployment. |
| `metadata.name` | Unique name of the Deployment (`my-deployment`). |
| `spec.replicas` | Number of Pod replicas to maintain (3). |
| `selector.matchLabels` | Ensures that only Pods with `app: web` label are managed by this Deployment. |
| `template.metadata.labels` | Labels assigned to newly created Pods. |
| `template.spec.containers` | Defines the container, image (`nginx:1.19`), and exposed port (`80`). |

---

## 3. Creating and Managing Deployments
### Creating the Deployment
```sh
kubectl apply -f deployment.yaml
```

### Verifying the Deployment
```sh
kubectl get deployments
```

### Describing the Deployment
```sh
kubectl describe deployment my-deployment
```

---

## 4. Understanding `kubectl describe deployment` Output

### Key Sections Explained
| **Field** | **Description** |
|-----------|---------------|
| **Name** | The Deployment's name. |
| **Namespace** | The namespace where it exists (`default`). |
| **Selector** | Labels used to manage Pods (`app=web`). |
| **Replicas** | Desired (3), updated (3), total (3), available (3), unavailable (0). |
| **StrategyType** | Update strategy (RollingUpdate or Recreate). |
| **MinReadySeconds** | Time a Pod must be ready before considered available (0 by default). |
| **RollingUpdateStrategy** | Controls the max unavailable and max surge Pods during an update. |
| **Pod Template** | Shows the labels and container specifications. |
| **Containers** | Lists container names, images, ports. |
| **Conditions** | Displays health status (`Available`, `Progressing`). |
| **Events** | Shows important events like scaling, failures, and rollouts. |

---

## 5. ReplicaSets and How Deployments Manage Them
### Checking ReplicaSets
```sh
kubectl get replicasets
```

---

## 6. Rolling Updates and Rollbacks
### Performing a Rolling Update
```sh
kubectl set image deployment/my-deployment webserver=nginx:1.20
```
### Rolling Back to a Previous Version
```sh
kubectl rollout undo deployment/my-deployment
```

---

## 7. Scaling Deployments
### Scaling Up
```sh
kubectl scale deployment my-deployment --replicas=5
```
### Scaling Down
```sh
kubectl scale deployment my-deployment --replicas=2
```

---

## 8. Deployment → ReplicaSet → Pods Relationship (Diagram)
```
+-----------------------------+
|        Deployment           |
|-----------------------------|
|  - Manages ReplicaSets      |
|  - Handles rolling updates  |
|  - Ensures stateful scaling |
+-----------------------------+
        |
        | Creates ReplicaSets
        v
+-----------------------------+
|        ReplicaSet            |
|-----------------------------|
|  - Ensures Pod availability |
|  - Tracks rollout versions  |
+-----------------------------+
        |
        | Manages Pods
        v
+-----------------------------+
|            Pods              |
|-----------------------------|
|  - Runs application code    |
|  - Assigned unique IPs      |
+-----------------------------+
```

## Deployment menifiest Overview

Below is a simplified diagram that represents the structure of a Deployment and how it manages Pods.

```
+--------------------------------------------------+
|                 Deployment (my-deployment)       |
|--------------------------------------------------|
|  Replicas: 3                                     |
|                                                  |
|  +--------------------------------------------+  |
|  | Pod Template                               |  |
|  |  +--------------------------------------+  |  |
|  |  |   metadata:                          |  |  |
|  |  |   - labels:                          |  |  |
|  |  |     app: web                         |  |  |
|  |  +--------------------------------------+  |  |
|  |                                          |  |  |
|  |  +--------------------------------------+  |  |
|  |  |   spec:                              |  |  |
|  |  |   - containers:                      |  |  |
|  |  |     - name: webserver                |  |  |
|  |  |     - image: nginx:1.19              |  |  |
|  |  |     - ports:                         |  |  |
|  |  |       - 80                           |  |  |
|  |  +--------------------------------------+  |  |
|  +--------------------------------------------+  |
|                                                  |
|  +--------------------------------------------+  |
|  | Managed Pods (3 replicas)                  |  |
|  +--------------------------------------------+  |
|                                                  |
+--------------------------------------------------+
```

---

## 9. Summary
### What We Covered
✅ Deployment YAML structure.  
✅ Creating, updating, and managing Deployments.  
✅ `kubectl describe deployment` explained in detail.  
✅ **How Deployments interact with ReplicaSets.**  
✅ Rolling updates, history, and rollbacks.  
✅ Scaling up and down.  


