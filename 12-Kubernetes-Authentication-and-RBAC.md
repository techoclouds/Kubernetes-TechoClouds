# Kubernetes Authentication and Role-Based Access Control (RBAC) - Part 1

## Introduction

Kubernetes empowers you to orchestrate containerized applications in a secure and scalable manner. Two essential pillars of Kubernetes security are **authentication** and **authorization**. Authentication verifies the identity of users or systems attempting to access the cluster, while authorization determines what actions they are permitted to perform on cluster resources.

This chapter focuses on the **fundamental concepts** of Kubernetes authentication and RBAC, ensuring a clear understanding of how access control works in a cluster environment.

---

## **Section 1: Kubernetes Authentication**

### **1.1 Authentication Flow**

Authentication in Kubernetes follows a structured process:

1. A user or service sends a request to the **Kubernetes API Server**.
2. The API Server verifies the identity using one of the authentication mechanisms (X.509 certificates, tokens, or external providers).
3. If authentication is successful, the API Server checks **RBAC policies** to determine what actions the user or service is allowed to perform.
4. The API Server either allows or denies the request.

#### **Flow Diagram: Kubernetes Authentication Process**

```
[ User / Service ] → [ API Server ] → [ Authentication Check ] → [ Authorization (RBAC) ] → [ Request Approved/Denied ]
```

---

### **1.2 Authentication Methods**

#### **Service Account Authentication**

- Kubernetes assigns a **Service Account** to each pod for authentication within the cluster.
- Service account credentials are mounted inside the pod at:
  ```bash
  /var/run/secrets/kubernetes.io/serviceaccount/token
  ```
- **To disable auto-mounting of service account tokens for security:**
  ```yaml
  automountServiceAccountToken: false
  ```

##### **Flow Diagram: Service Account Authentication**

```
[ Pod ] → [ Kubernetes API Server ] → [ Service Account Token Validation ] → [ Request Approved/Denied ]
```

- **Creating a Custom Service Account:**
  ```bash
  kubectl create serviceaccount custom-sa
  ```
  Assigning it to a pod:
  ```yaml
  spec:
    serviceAccountName: custom-sa
  ```

#### **kubeconfig and kubectl Authentication**

- The **kubeconfig** file stores cluster configuration, credentials, and context.
- **kubectl** uses kubeconfig for user authentication.
- **Example: Viewing Kubeconfig Settings**
  ```bash
  kubectl config view
  ```

##### **Flow Diagram: kubeconfig Authentication**

```
[ User ] → [ kubeconfig ] → [ Kubernetes API Server ] → [ Request Processed ]
```

#### **User Authentication Mechanisms**

- Kubernetes supports different user authentication methods:
  - **X.509 client certificates** (strong mutual TLS authentication)
  - **Bearer tokens** (JSON Web Tokens - JWTs)
  - **External Identity Providers** (LDAP, OpenID Connect, etc.)

##### **Example: Creating an X.509 User Certificate**

```bash
openssl genrsa -out user.key 2048
openssl req -new -key user.key -out user.csr -subj "/CN=myuser"
```

#### **Token Review API & Webhook Authentication**

- The **TokenReview API** allows external authentication systems to validate user tokens.
- Kubernetes also supports **Webhook Authentication**, where external identity providers (LDAP, OAuth2, etc.) authenticate users.

##### **Flow Diagram: Webhook Authentication**

```
[ User ] → [ Kubernetes API Server ] → [ Webhook Authentication Server ] → [ Identity Verified ]
```

#### **OIDC Authentication Setup**

- Kubernetes integrates with OpenID Connect (OIDC) for authentication.
- **Example configuration:**
  ```yaml
  --oidc-issuer-url=https://accounts.google.com
  --oidc-client-id=my-client-id
  --oidc-username-claim=email
  ```

##### **Flow Diagram: OIDC Authentication**

```
[ User ] → [ OIDC Provider ] → [ Kubernetes API Server ] → [ RBAC Authorization ] → [ Request Processed ]
```

---

## **Section 2: Kubernetes RBAC**

### **2.1 What is RBAC?**

- **RBAC (Role-Based Access Control)** allows cluster administrators to define and enforce user and service access rules.
- It ensures that users and services operate under the **principle of least privilege**.

##### **Flow Diagram: Kubernetes RBAC Process**

```
[ User / Service ] → [ API Server ] → [ Authorization Check (RBAC) ] → [ Role / RoleBinding Lookup ] → [ Allowed / Denied ]
```

### **2.2 Roles and RoleBindings**

- **Role:** Defines a set of permissions within a specific namespace.
- **RoleBinding:** Assigns a Role to a user, group, or service account in a namespace.

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

---

### **2.3 ClusterRoles and ClusterRoleBindings**

- **ClusterRole:** Provides access to resources across **all namespaces**.
- **ClusterRoleBinding:** Grants a user access at the cluster level.

#### **Example: Creating a ClusterRole and ClusterRoleBinding**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-list-reader-cluster-wide
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: john-cluster-wide-reader
subjects:
- kind: User
  name: john
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pod-list-reader-cluster-wide
```

---

## **Conclusion**

Understanding Kubernetes authentication and RBAC is crucial for securing your clusters. By mastering these concepts, you can effectively manage user and service access, enforce **least privilege principles**, and protect your containerized applications.

### **Key Takeaways:**

- Authentication mechanisms include **Service Accounts, kubeconfig, X.509 certificates, and OIDC**.
- Kubernetes RBAC uses **Roles, ClusterRoles, RoleBindings, and ClusterRoleBindings** to enforce access control.
- Best practices include **audit logging, reviewing policies regularly, and restricting excessive permissions**.



# Kubernetes Authentication and Role-Based Access Control (RBAC) - Part 2

## Introduction

In **Part 1**, we covered the fundamentals of Kubernetes authentication and RBAC. Now, in **Part 2**, we will focus on **hands-on scenarios and advanced topics**, providing practical exercises that can be tested on **Minikube**.

This section includes real-world use cases, RBAC security enhancements, and advanced best practices for securing your Kubernetes cluster effectively.

---

## **Section 3: Hands-On RBAC Scenarios**

### **3.1 Checking Current User and Permissions in Minikube**
#### **Understanding Minikube Default User (Admin Privileges)**
By default, Minikube creates an administrative user that has full control over the cluster. Before modifying permissions, let's inspect the current user and its capabilities.

- **Check the current user configured in kubeconfig:**
  ```bash
  kubectl config view --minify | grep current-context
  ```
- **Verify that the current user has cluster-admin privileges:**
  ```bash
  kubectl auth can-i '*' '*' --all-namespaces
  ```
  Expected Output:
  ```bash
  yes
  ```
  This means the default user has full cluster access.

#### **Listing Available ClusterRoles**
- **View the default ClusterRoles in Kubernetes:**
  ```bash
  kubectl get clusterroles
  ```
  Look for `cluster-admin`, `edit`, `view`, and other predefined roles.

#### **Viewing Current RoleBindings**
- **Check which roles the current user is bound to:**
  ```bash
  kubectl get clusterrolebindings | grep system:masters
  ```
  This typically shows that the default user belongs to the `system:masters` group, which has `cluster-admin` access.

---

### **3.2 Creating a New User and Assigning Custom Permissions**
#### **Step 1: Generate a Certificate for a New User**
- Create a private key:
  ```bash
  openssl genrsa -out user.key 2048
  ```
- Generate a Certificate Signing Request (CSR):
  ```bash
  openssl req -new -key user.key -out user.csr -subj "/CN=newuser"
  ```
- Approve and sign the certificate using Kubernetes CA:
  ```bash
  openssl x509 -req -in user.csr -CA $(minikube ssh -- cat /var/lib/minikube/certs/ca.crt) -CAkey $(minikube ssh -- cat /var/lib/minikube/certs/ca.key) -CAcreateserial -out user.crt -days 365
  ```

#### **Step 2: Create a Kubernetes Role for the New User**
- Define a role that allows listing pods:
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
  Apply the role:
  ```bash
  kubectl apply -f pod-reader-role.yaml
  ```

#### **Step 3: Create a RoleBinding for the New User**
- Bind the new user to the `pod-reader` role:
  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    namespace: default
    name: pod-reader-binding
  subjects:
  - kind: User
    name: newuser
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: Role
    name: pod-reader
  ```
  Apply the RoleBinding:
  ```bash
  kubectl apply -f pod-reader-binding.yaml
  ```

#### **Step 4: Add the User to Kubeconfig**
- Create a new kubeconfig entry for the user:
  ```bash
  kubectl config set-credentials newuser --client-certificate=user.crt --client-key=user.key
  kubectl config set-context newuser-context --cluster=minikube --user=newuser --namespace=default
  kubectl config use-context newuser-context
  ```

#### **Step 5: Test User Permissions**
- Verify that the new user can list pods:
  ```bash
  kubectl get pods
  ```
- Attempt to delete a pod (which should fail):
  ```bash
  kubectl delete pod <pod-name>
  ```
  Expected Output:
  ```bash
  Error from server (Forbidden): pods is forbidden: User "newuser" cannot delete resource "pods" in API group "" in the namespace "default"
  ```

---

## **Section 4: Advanced RBAC Security Enhancements**

### **4.1 Enabling Kubernetes Audit Logs**
- To enable audit logs in Minikube, restart with:
  ```bash
  minikube start --extra-config=apiserver.audit-log-path=-
  ```
- View audit logs:
  ```bash
  kubectl logs -n kube-system kube-apiserver-minikube | grep "authorization.k8s.io"
  ```

---

### **4.2 Preventing Role Escalation**
- Avoid granting `create` or `update` permissions on `RoleBinding` and `ClusterRoleBinding` to regular users.
- **Example of a secure role:**
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

In **Part 2**, we covered:
- How to **check the current user’s permissions** in Minikube.
- **Creating a new user and assigning custom permissions** with X.509 certificates.
- **RBAC security enhancements**, including **audit logs** and **role escalation prevention**.

