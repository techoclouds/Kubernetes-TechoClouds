# Chapter 10: ConfigMaps in Kubernetes

## Introduction

ConfigMaps are a crucial part of managing configuration data in Kubernetes, offering a flexible and efficient way to store key-value pairs. This chapter explores the fundamentals of ConfigMaps, demonstrates their usage with a hands-on example using Minikube, delves into advanced topics like mounting ConfigMaps as volumes, discusses best practices, and concludes with a tutorial on creating ConfigMaps using kubectl.

## What are ConfigMaps?

ConfigMaps in Kubernetes serve as a mechanism for decoupling configuration artifacts from containerized applications. They are designed to store configuration data in key-value pairs, providing a convenient way to manage and update application settings without modifying the application itself. ConfigMaps offer several advantages:

- **Portability:** By separating configuration from the application, the Docker image becomes more portable across different environments.
- **Flexibility:** Updates to configuration can be made dynamically without the need to rebuild the application image.
- **Security:** Sensitive information can be kept separate from application code, enhancing security practices.

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
kubectl create -f configmap.yaml
```

### Verifying the ConfigMap

To ensure the ConfigMap has been created successfully, view its details:

```bash
kubectl get configmap my-app-config -o yaml
```

### Deploying an Application

Reference the ConfigMap in your deployment manifest by using environment variables. For instance:

```yaml
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

### Accessing the ConfigMap in Your Application

Check the application pod logs to see the environment variables populated with values from the ConfigMap:

```bash
kubectl logs -f pods <pod_name>
```

You should observe the application using configuration values from the ConfigMap.

### Advanced Topics

#### Mounting ConfigMaps as Volumes

To access ConfigMap content as files, mount the ConfigMap as a volume:

```yaml
volumes:
- name: config-volume
  configMap:
    name: my-app-config
```

Mount the volume inside your container:

```yaml
containers:
- name: my-app-container
  ...
  volumeMounts:
  - name: config-volume
    mountPath: /app/config
```

Now, your application can access configuration files at `/app/config`.

#### Using ConfigMaps with Different Data Types

ConfigMaps can store various data types, including Base64-encoded binary data and JSON or YAML for structured configurations. Remember to encode binary data as Base64 before adding it to the ConfigMap, and parse JSON or YAML within your application.

## Best Practices

When working with ConfigMaps, it's essential to follow best practices:

- **Avoid sensitive data:** For passwords or sensitive information, use Kubernetes Secrets.
- **Multiple ConfigMaps:** Consider creating separate ConfigMaps for different environments (development, staging, production).
- **Mount specific keys:** Only mount specific keys to avoid exposing unnecessary data.

## kubectl create configmap Tutorial

The `kubectl create configmap` command allows you to create ConfigMaps from various data sources. Here's a tutorial on creating ConfigMaps using different methods:

### Creating from Literal Values

```bash
kubectl create configmap <configmap-name> --from-literal=<key1>=<value1> --from-literal=<key2>=<value2>
```

### Creating from Files

```bash
kubectl create configmap <configmap-name> --from-file=<file1> --from-file=<file2>
```

### Creating from Directories

```bash
kubectl create configmap <configmap-name> --from-file=<directory-path>
```

### Specifying Data Types

```bash
kubectl create configmap <configmap-name> --from-file=<file1> --from-literal=<key1>=<value1>
```

This tutorial covers creating ConfigMaps from literal values, files, and directories while specifying data types as needed.

In the next chapters, we'll explore more advanced use cases and optimization techniques for managing configuration in Kubernetes.
