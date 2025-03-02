# Chapter 10: ConfigMaps in Kubernetes

## Introduction

ConfigMaps are a crucial part of managing configuration data in Kubernetes, offering a flexible and efficient way to store key-value pairs. This chapter explores the fundamentals of ConfigMaps, demonstrates their usage with a hands-on example using Minikube, delves into advanced topics like mounting ConfigMaps as volumes, discusses best practices, and concludes with a tutorial on creating ConfigMaps using kubectl.

## What are ConfigMaps?

ConfigMaps in Kubernetes serve as a mechanism for decoupling configuration artifacts from containerized applications. They are designed to store configuration data in key-value pairs, providing a convenient way to manage and update application settings without modifying the application itself. ConfigMaps offer several advantages:

- **Portability:** By separating configuration from the application, the Docker image becomes more portable across different environments.
- **Flexibility:** Updates to configuration can be made dynamically without the need to rebuild the application image.
- **Maintainability:** Configuration data is managed separately, making application management easier.

## Supported Data Types in ConfigMaps

ConfigMaps can store various types of data:
- **Plain Text:** Simple key-value pairs, e.g., `db_host: "127.0.0.1"`.
- **Multiline Text:** YAML or JSON content stored as a value.
- **Binary Data (Base64-encoded):** Although ConfigMaps do not natively store binary data, you can encode files using Base64 and decode them within the application.

Example of storing structured JSON:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: json-configmap
data:
  config.json: |
    {
      "db_host": "127.0.0.1",
      "db_port": "3306"
    }
```

## Hands-on Example with Minikube

### Setup

Before diving into the hands-on example, make sure you have Minikube installed and running.

### Creating a ConfigMap

In this example, let's create a ConfigMap named `my-app-config` containing essential database connection details. Save the following YAML as `configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-app-config
data:
  db_host: "127.0.0.1"
  db_port: "3306"
  username: "myapp_user"
  password: "myapp_password"
```

Create the ConfigMap using kubectl:

```bash
kubectl apply -f configmap.yaml
```

### Verifying the ConfigMap

To ensure the ConfigMap has been created successfully, view its details:

```bash
kubectl get configmap my-app-config -o yaml
```

### Using ConfigMap in Deployments

#### Using ConfigMap as Environment Variables

Reference the ConfigMap in your deployment manifest using environment variables. Create `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
defaults:
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: myapp-image
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: my-app-config
              key: db_host
        - name: DB_PORT
          valueFrom:
            configMapKeyRef:
              name: my-app-config
              key: db_port
        - name: DB_USERNAME
          valueFrom:
            configMapKeyRef:
              name: my-app-config
              key: username
```

Deploy the application:

```bash
kubectl apply -f deployment.yaml
```

### Verifying Environment Variables in Pod

Check if the environment variables are correctly set inside the running pod:

```bash
kubectl exec -it <pod-name> -- env | grep DB_
```

### Mounting ConfigMap as a Volume

To access ConfigMap content as files, mount the ConfigMap as a volume:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  containers:
  - name: my-app-container
    image: myapp-image
    volumeMounts:
    - name: config-volume
      mountPath: /app/config
  volumes:
  - name: config-volume
    configMap:
      name: my-app-config
```

Apply the pod configuration:

```bash
kubectl apply -f pod.yaml
```

Verify the mounted ConfigMap data:

```bash
kubectl exec -it my-app-pod -- ls /app/config
```

## Advanced Topics

### Handling ConfigMap Updates Without Pod Restarts

- **Environment Variables:** Pods do **not** automatically update when a ConfigMap changes. You must manually restart the pod.
- **Mounted Volumes:** If ConfigMaps are mounted as volumes, updates are automatically reflected.

### Immutable ConfigMaps

To prevent accidental modifications and improve performance, set ConfigMaps as immutable:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-app-config
immutable: true
data:
  db_host: "127.0.0.1"
```

## Namespace Considerations

ConfigMaps are namespace-scoped. To retrieve a ConfigMap from a specific namespace:

```bash
kubectl get configmap my-app-config -n my-namespace
```
## Handling Large Configuration Files

For large configuration data, store JSON/YAML directly in ConfigMaps:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-app-config
data:
  config.json: |
    {
      "db_host": "127.0.0.1",
      "db_port": "3306"
    }
```

Mount it in a pod and access `/app/config/config.json`.

## Best Practices

- **Avoid sensitive data:** Use Kubernetes Secrets for passwords and sensitive information.
- **Multiple ConfigMaps:** Create separate ConfigMaps for different environments (dev, staging, prod).
- **Mount specific keys:** Only mount necessary keys to avoid exposing unnecessary data.
- **Set defaults:** Use `optional: true` in `configMapKeyRef` to handle missing values.

## Troubleshooting ConfigMap Issues

### Common Issues & Fixes

| Issue | Cause | Solution |
|-------|-------|----------|
| ConfigMap not found | Incorrect name reference | Check `kubectl get configmap` |
| No environment variable set | Pod did not restart after ConfigMap update | Restart the pod |
| ConfigMap volume not mounted | Incorrect path reference | Use `kubectl exec` to check volume content |
## kubectl create configmap Tutorial

### Creating from Literal Values

```bash
kubectl create configmap my-config --from-literal=db_host=127.0.0.1 --from-literal=db_port=3306
```

### Creating from Files

```bash
kubectl create configmap my-config --from-file=app-config.txt
```

### Creating from Directories

```bash
kubectl create configmap my-config --from-file=/path/to/config-dir
```

### Verifying ConfigMap

```bash
kubectl describe configmap my-config
```

## Conclusion

In this chapter, we covered ConfigMaps, their use cases, hands-on examples, advanced topics, best practices, and troubleshooting techniques. In the next chapter, we will explore Kubernetes Secrets and how they differ from ConfigMaps in handling sensitive data.

