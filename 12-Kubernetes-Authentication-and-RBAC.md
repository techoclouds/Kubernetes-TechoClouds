
# Kubernetes Authentication and Role-Based Access Control (RBAC)

## Introduction

Kubernetes empowers you to orchestrate containerized applications in a secure and scalable manner. Two essential pillars of Kubernetes security are **authentication** and **authorization**:

- **Authentication** verifies the identity of users or systems trying to access the cluster.  
- **Authorization** determines the specific actions that identity can perform on cluster resources.

This guide provides a **comprehensive overview** of Kubernetes authentication and RBAC, covering **concepts, hands-on exercises, and best practices**.

---

## **Section 1: Kubernetes Authentication**

### **1.1 Kubernetes Authentication Types**

Kubernetes supports two main categories of authentication:

1. **User Accounts**  
   - Represent external identities (e.g., developers, admins).  
   - **Kubernetes does not manage user accounts** directly. You typically integrate with external authentication systems (e.g., X.509 certificates, OpenID Connect, etc.).  
   - Groups are also an external concept. If you add a user to a group, that is generally managed outside Kubernetes.

2. **Service Accounts**  
   - Internal identities used by **Pods** and **services** within Kubernetes.  
   - Managed by Kubernetes and stored as Secrets.  
   - Automatically mounted into Pods via volume at `/var/run/secrets/kubernetes.io/serviceaccount/`.

> **Important note:**  
> User and group management (creating, modifying, deleting users/groups) is **not** handled by Kubernetes. Instead, Kubernetes trusts an external identity source (like an OIDC provider, certificate authority, etc.) to tell it who you are when you interact with the cluster.

### **1.2 Authentication Flow**

1. A **User** or **Service Account** sends a request to the **API Server**.  
2. The API Server verifies the request identity using the configured authentication mechanism (e.g., certificate, bearer token).  
3. If authentication succeeds, the API Server checks **RBAC policies** to see if the identity is allowed to perform the requested action.  
4. The API Server either **allows** or **denies** the request.

#### **High-Level Diagram: Kubernetes Authentication & Authorization**

```
 ┌─────────────────────┐      ┌─────────────────────────────┐
 │     User / Pod      │      │         API Server          │
 └─────────────────────┘      └─────────────────────────────┘
             |                           │
             | 1) Request                │
             | ─────────────────────────> │
             |                           │
             |      2) Authentication    │
             | <───────────────────────── │
             |                           │
             |      3) RBAC Check        │
             | <───────────────────────── │
             |                           │
             | 4) Allowed or Denied      │
             | <───────────────────────── │
```

### **1.3 Common Authentication Mechanisms**

- **kubeconfig** files (used by `kubectl`).  
- **X.509 client certificates** (mutual TLS).  
- **Bearer tokens** (e.g., JSON Web Tokens—JWTs).  
- **External identity providers** (LDAP, OpenID Connect).

#### **Example: Creating an X.509 Certificate for a User**

```bash
# Generate a private key
openssl genrsa -out user.key 2048

# Create a CSR (Certificate Signing Request)
openssl req -new -key user.key -out user.csr -subj "/CN=myuser"
```

You would then sign `user.csr` with your cluster’s Certificate Authority (CA). The resulting user certificate allows `myuser` to authenticate, but **does not** automatically grant any permissions. Authorization is a separate step (via RBAC).

---

## **Section 2: Kubernetes RBAC**

### **2.1 What is RBAC?**

RBAC (**Role-Based Access Control**) in Kubernetes is the mechanism that **authorizes** what an authenticated identity (user, group, or service account) can do.

### **2.2 Roles vs. ClusterRoles**

Kubernetes RBAC revolves around two key concepts: **Roles** and **ClusterRoles**.

1. **Role**  
   - Defines a set of permissions (rules) **within a single namespace**.  
   - Example rules might allow listing or creating Pods in the `default` namespace.

2. **ClusterRole**  
   - Like a Role, but it applies **cluster-wide** (i.e., across all namespaces) or to **cluster-scoped** resources (e.g., Nodes).  
   - Typically used for permissions that span multiple namespaces or cluster-level objects like `nodes` or `PersistentVolumes`.

> **Key Distinction**  
> - Use a **Role** if you want to restrict actions to a specific namespace.  
> - Use a **ClusterRole** if you need those permissions to apply across all namespaces or for cluster-scoped resources.

### **2.3 RoleBindings vs. ClusterRoleBindings**

To actually assign a Role or ClusterRole to a user (or group, or service account), you create a **RoleBinding** or **ClusterRoleBinding**.

1. **RoleBinding**  
   - Binds a **Role** to a user, group, or service account **within a particular namespace**.  
   - If you bind a user to a Role in the `default` namespace, they do **not** automatically have those permissions in other namespaces.

2. **ClusterRoleBinding**  
   - Binds a **ClusterRole** to a user, group, or service account **across the entire cluster**.  
   - Grants permissions in **all namespaces** for namespaced resources, or cluster-wide for cluster-scoped resources.

---

## **Section 3: Default RBAC Roles and ClusterRoles**

Kubernetes ships with **many built-in ClusterRoles**. You can view them with:

```bash
kubectl get clusterroles
```

Some common default ClusterRoles include:

- **cluster-admin**  
  - Full admin rights across the entire cluster.  
  - Typically bound to admin users in development or single-tenant clusters. **Use with caution** in multi-tenant environments.

- **admin**  
  - Has admin rights **within a specific namespace**, including read/write access to most resources.  
  - Usually used via a RoleBinding.

- **edit**  
  - Lets you create, update, and delete most objects in a namespace.  
  - Does **not** grant access to Roles or RoleBindings (so you can’t change RBAC policies).

- **view**  
  - Read-only access to see most objects in a namespace, but cannot make changes.

### **Example: Viewing all default ClusterRoles**

```bash
kubectl get clusterroles

# Common built-ins you might see:
# NAME                              AGE
# cluster-admin                     ...
# admin                             ...
# edit                              ...
# view                              ...
# system:discovery                  ...
# system:kube-scheduler             ...
# system:node                       ...
# ...
```

You can inspect any of these with:

```bash
kubectl describe clusterrole <clusterrole-name>
```

---

## **Section 4: Practical Examples**

### **4.1 Creating a Role and RoleBinding**

Below is a simple example where we grant **list Pods** permission in the `default` namespace to a user named `john`.

```yaml
# pod-list-reader-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-list-reader
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list"]
```

```yaml
# john-pod-list-reader-bind.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: john-pod-list-reader
  namespace: default
subjects:
  - kind: User
    name: john          # This is the user identity recognized by the Auth system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-list-reader
```

Apply these objects:

```bash
kubectl apply -f pod-list-reader-role.yaml
kubectl apply -f john-pod-list-reader-bind.yaml
```

#### **Testing Permissions**

If `john` authenticates to the cluster (e.g., with a client certificate or kubeconfig set to `/CN=john`), the following commands will succeed or fail accordingly:

```bash
# john can list pods in 'default' namespace
kubectl get pods --namespace=default --as=john

# john CANNOT delete pods (will fail)
kubectl delete pod my-pod --namespace=default --as=john
```

### **4.2 Creating a ClusterRole and ClusterRoleBinding**

If you want `john` to have read-only access to all Pods in **all namespaces**, you can create a **ClusterRole** (rather than a Role) and a **ClusterRoleBinding**:

```yaml
# cluster-pod-read.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

```yaml
# cluster-pod-read-bind.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: john-cluster-pod-reader
subjects:
  - kind: User
    name: john
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-pod-reader
```

Now `john` can list Pods in every namespace, but has no other privileges (like create, update, or delete).

---

## **Section 5: Hands-On Verification & Security Enhancements**

### **5.1 Checking Current Context & Permissions**

```bash
# Show which cluster/context your kubectl is pointing to
kubectl config current-context

# Quick check: "Can I do <verb> <resource> in this namespace?"
kubectl auth can-i get pods --namespace=default
```

### **5.2 Auditing & Logging**

- **Audit Logs** capture the details of each request to the API Server.  
- Enable them (if not already) in your Kubernetes installation:
  ```bash
  # For example, in Minikube:
  minikube start --extra-config=apiserver.audit-log-path=-
  kubectl logs -n kube-system kube-apiserver-minikube | grep "authorization.k8s.io"
  ```
- Review logs for potential unauthorized attempts or role escalations.

### **5.3 Preventing Role Escalation**

When you create a Role or ClusterRole, **avoid** giving out permissions to manage RBAC objects (`roles`, `rolebindings`, etc.) unless absolutely necessary. This principle helps prevent a user from granting themselves higher privileges.

Example of a read-only Role that disallows writes:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: secure-role
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["get", "list", "watch"]  # No create/update/delete
```

---

## **Conclusion**

This revised guide outlines **Kubernetes Authentication** (covering **User** vs **Service Accounts**, and the fact that Kubernetes does not manage your users or groups) and **RBAC** (covering **Roles**, **RoleBindings**, **ClusterRoles**, and **ClusterRoleBindings**, plus some practical defaults and security recommendations).

Key takeaways:

1. **Authentication** in Kubernetes is provided by external mechanisms.  
2. **Authorization** uses RBAC (Roles & RoleBindings, or ClusterRoles & ClusterRoleBindings).  
3. **Default ClusterRoles** like `cluster-admin`, `admin`, `edit`, and `view` are helpful starting points.  
4. Always follow **least privilege** principles to secure your cluster.  

By assigning the right type of **Role** or **ClusterRole** and carefully binding it, you can ensure **each identity only has the permissions** it truly needs.
```
