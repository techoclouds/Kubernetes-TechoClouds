# Kubernetes Authentication and Role-Based Access Control (RBAC)

## Introduction

Kubernetes empowers you to orchestrate containerized applications in a secure and scalable manner. Two essential pillars of Kubernetes security are **authentication** and **authorization**. Authentication verifies the identity of users or systems attempting to access the cluster, while authorization determines what actions they are permitted to perform on cluster resources.

This document provides a **comprehensive guide** to Kubernetes authentication and RBAC, including **concepts, hands-on exercises, and advanced best practices**.

---

## **Section 1: Kubernetes Authentication**

### **1.1 Kubernetes Authentication Types**

Kubernetes supports two main authentication types:

1. **User Accounts**: External identities accessing the cluster (e.g., developers, admins).
   - Kubernetes does **not** manage user accounts directly.
   - Users authenticate using external mechanisms like X.509 certificates, bearer tokens, or OpenID Connect.

2. **Service Accounts**: Internal identities used by **Pods** and **services** inside Kubernetes.
   - Managed by Kubernetes and stored as secrets.
   - Used for inter-service communication within the cluster.

### **1.2 Authentication Flow**

Authentication in Kubernetes follows a structured process:

1. A user or service sends a request to the **Kubernetes API Server**.
2. The API Server verifies the identity using an authentication mechanism.
3. If authentication is successful, the API Server checks **RBAC policies** to determine allowed actions.
4. The API Server either allows or denies the request.

#### **Flow Diagram: Kubernetes Authentication Process**

```
[ User / Service ] → [ API Server ] → [ Authentication Check ] → [ Authorization (RBAC) ] → [ Request Approved/Denied ]
```

### **1.3 Authentication Methods**

#### **Service Account Authentication (Default for Pods)**

- Kubernetes assigns a **Service Account** to each pod for authentication.
- Service account credentials are mounted inside the pod at:
  ```bash
  /var/run/secrets/kubernetes.io/serviceaccount/token
  ```
- **To disable auto-mounting of service account tokens:**

  ```yaml
  automountServiceAccountToken: false
  ```

#### **User Authentication Mechanisms**

- **kubeconfig** (used by `kubectl`)
- **X.509 client certificates** (mutual TLS authentication)
- **Bearer tokens** (JSON Web Tokens - JWTs)
- **External Identity Providers** (LDAP, OpenID Connect)

##### **Example: Creating an X.509 User Certificate**

```bash
openssl genrsa -out user.key 2048
openssl req -new -key user.key -out user.csr -subj "/CN=myuser"
```

#### **OIDC Authentication Setup**

- Kubernetes integrates with OpenID Connect (OIDC) for authentication.

  ```yaml
  --oidc-issuer-url=https://accounts.google.com
  --oidc-client-id=my-client-id
  --oidc-username-claim=email
  ```

---

## **Section 2: Kubernetes RBAC**

### **2.1 What is RBAC?**

RBAC (Role-Based Access Control) defines what authenticated users **can** and **cannot** do.

##### **Flow Diagram: Kubernetes RBAC Process**

```
[ User / Service ] → [ API Server ] → [ Authorization Check (RBAC) ] → [ Role / RoleBinding Lookup ] → [ Allowed / Denied ]
```

### **2.2 Roles and RoleBindings**

- **Role**: Defines permissions within a specific namespace.
- **RoleBinding**: Assigns a Role to a user, group, or service account in a namespace.

#### **Example: Creating a Role and RoleBinding**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-list-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: default
  name: john-pod-list-reader
subjects:
- kind: User
  name: john
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-list-reader
```

### **2.3 ClusterRoles and ClusterRoleBindings**

- **ClusterRole**: Provides access across **all namespaces**.
- **ClusterRoleBinding**: Grants a user access at the **cluster level**.

---

## **Section 3: Hands-On RBAC Scenarios**

### **3.1 Checking Current User and Permissions in Minikube**

```bash
kubectl config view --minify | grep current-context
kubectl auth can-i '*' '*' --all-namespaces
```

### **3.2 Creating a New User and Assigning Custom Permissions**

#### **Step 1: Generate a Certificate for a New User**
```bash
openssl genrsa -out user.key 2048
openssl req -new -key user.key -out user.csr -subj "/CN=newuser"
```

#### **Step 2: Assign RBAC Permissions**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```
```bash
kubectl apply -f pod-reader-role.yaml
```

#### **Step 3: Test User Permissions**
```bash
kubectl get pods --as=newuser
kubectl delete pod <pod-name> --as=newuser  # Should fail
```

---

## **Section 4: Advanced RBAC Security Enhancements**

### **4.1 Enabling Kubernetes Audit Logs**
```bash
minikube start --extra-config=apiserver.audit-log-path=-
kubectl logs -n kube-system kube-apiserver-minikube | grep "authorization.k8s.io"
```

### **4.2 Preventing Role Escalation**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: secure-role
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]  # No create/update access
```

---

## **Conclusion**

This guide provided a deep dive into **Kubernetes authentication and RBAC**, including:

- **Understanding authentication mechanisms** (Users vs. Service Accounts)
- **Configuring RBAC roles and permissions**
- **Testing permissions using `kubectl auth can-i`**
- **Securing access and enabling audit logs**

By following these practices, you can **ensure least privilege access** and **protect your Kubernetes workloads effectively**.

