
### Chapter 13: Namespaces

#### Objective
Understand the concept of namespaces to manage resources in a multi-tenant environment effectively.

#### Introduction to Namespaces
- **Definition:** Namespaces are logical partitions in a Kubernetes cluster used to divide cluster resources between multiple users.
- **Purpose:** They help isolate resources, provide a unique scope for naming, and simplify resource management in environments with many users spread across multiple teams or projects.

#### Default vs. Custom Namespaces
- **Default Namespaces:**
  - `default`: Used for objects without a specific namespace.
  - `kube-system`: For system-generated objects and services.
  - `kube-public`: Universally accessible across the cluster.
- **Custom Namespaces:**
  - **Benefits:** Help organize resources by project, team, or development stage; limit resource visibility and access.
  - **CLI Command:**
    ```bash
    kubectl create namespace development
    ```
  - **Manifest File (development-namespace.yaml):**
    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: development
    ```
  - **Deploy using:**
    ```bash
    kubectl apply -f development-namespace.yaml
    ```

#### Managing Resources Across Namespaces
- **Resource Isolation:** Namespaces are key to isolating resources such as pods, services, and deployments, preventing conflicts between different teams or projects.
- **Resource Sharing:** Certain cluster resources, like Persistent Volume Claims, can be shared across namespaces depending on permissions and configurations.

### Hands-on Examples

#### 1. Creating a Namespace
- **Goal:** Create a new namespace called `test-namespace`.
- **CLI Command:**
  ```bash
  kubectl create namespace test-namespace
  ```
- **Manifest File (test-namespace.yaml):**
  ```yaml
  apiVersion: v1
    kind: Namespace
    metadata:
      name: test-namespace
  ```
- **Deploy using:**
  ```bash
  kubectl apply -f test-namespace.yaml
  ```

#### 2. Deploying Pods in Different Namespaces
- **Goal:** Deploy a simple Nginx server in both the `default` and `test-namespace`.
- **CLI Commands:**
  ```bash
  kubectl run nginx-default --image=nginx --namespace=default
  kubectl run nginx-test --image=nginx --namespace=test-namespace
  ```
- **Manifest Code (nginx-deployment.yaml):**
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
    namespace: test-namespace
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:latest
          ports:
          - containerPort: 80
  ```
- **Deploy using:**
  ```bash
  kubectl apply -f nginx-deployment.yaml
  ```

#### 3. Accessing Services Across Namespaces
- **Goal:** Create services in both namespaces and access the service in `test-namespace` from the `default` namespace.
- **CLI Commands for Exposing Deployments:**
  ```bash
  kubectl expose deployment nginx-default --port=80 --type=ClusterIP --namespace=default
  kubectl expose deployment nginx-test --port=80 --type=ClusterIP --namespace=test-namespace
  ```
- **Service Manifest (nginx-service.yaml):**
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
    namespace: test-namespace
  spec:
    type: ClusterIP
    ports:
    - port: 80
      protocol: TCP
      targetPort: 80
    selector:
      app: nginx
  ```
- **Deploy using:**
  ```bash
  kubectl apply -f nginx-service.yaml
  ```
- **Access the Service Using CLI:**
  ```bash
  # Run a temporary busybox pod in the default namespace to access the service across namespaces
  kubectl run busybox --image=busybox --rm -it --namespace=default -- wget -qO- http://nginx-service.test-namespace.svc.cluster.local
  ```

#### Summary
This chapter introduces Kubernetes namespaces, providing a foundation for resource management in a multi-tenant environment. We explored creating namespaces, deploying resources, and accessing services across namespace boundaries using both CLI commands and manifest files.

### Conclusion
Namespaces are essential for effective resource management and organizational structuring within Kubernetes, ensuring efficient isolation and sharing of resources across various projects or teams in the same cluster.
