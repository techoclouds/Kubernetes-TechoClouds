# Chapter 11: Secrets in Kubernetes

## Introduction

In the Kubernetes ecosystem, managing sensitive information is paramount to ensure the security of applications. Kubernetes provides a dedicated resource called **Secrets** for handling confidential data such as passwords, API keys, and TLS certificates. This chapter delves into the fundamentals of Secrets, explores their use cases, and provides practical examples to secure sensitive information within your Kubernetes cluster.

## What are Secrets?

Secrets in Kubernetes are resources designed to store sensitive information securely. They act as a safer alternative to ConfigMaps when dealing with confidential data. Secrets are encoded in Base64, adding a layer of obfuscation, though they are not encrypted. Common use cases for Secrets include storing database passwords, API keys, and TLS certificates.

## Understanding Base64 Encoding in Secrets

Kubernetes Secrets require values to be Base64-encoded before storage. However, **Base64 is not encryption**; it is just an encoding mechanism.

To manually encode a password before adding it to a Secret:

```bash
echo -n 'mypassword' | base64
```

To decode a Secret value:

```bash
echo 'bXlwYXNzd29yZA==' | base64 --decode
```

## Types of Secrets in Kubernetes

In Kubernetes, the `type` field of a Secret specifies the type of secret and influences how the secret data is used. The most common types are `Opaque`, `kubernetes.io/service-account-token`, and `kubernetes.io/dockerconfigjson`.

### Opaque

- **Description:** The `Opaque` type is a generic type that can be used to store arbitrary data. It is the default type for Secrets.
- **Use Case:** Opaque secrets are suitable for any kind of confidential information that does not require special handling, such as passwords or API keys.

**Example:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-app-secret
type: Opaque
data:
  db_password: <base64-encoded-password>
```

### kubernetes.io/service-account-token

- **Description:** This type is automatically used when a service account token is created. It is specifically designed for storing Kubernetes service account tokens.
- **Use Case:** Typically used to mount service account tokens into pods for authentication with the Kubernetes API server.

### kubernetes.io/dockerconfigjson

- **Description:** This type is used for storing Docker registry authentication credentials in a JSON format. It is often used with private container registries.
- **Use Case:** Useful when pulling images from a private Docker registry that requires authentication.

## Creating a Secret

Let's walk through creating a Secret named `my-app-secret` containing a database password. Save the following YAML as `secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-app-secret
type: Opaque
data:
  db_password: <base64-encoded-password>
```

Replace `<base64-encoded-password>` with the Base64-encoded representation of your actual database password. Create the Secret using:

```bash
kubectl apply -f secret.yaml
```

## Verifying the Secret

To ensure the Secret has been created successfully, inspect its details:

```bash
kubectl get secret my-app-secret
```

## Using Secrets in Pods

### Using Secrets as Environment Variables

Reference the Secret in your pod's configuration by adding the following environment variable:

```yaml
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: my-app-secret
      key: db_password
```

### Mounting Secrets as Volumes

Secrets can also be mounted as files within a pod:

```yaml
volumes:
  - name: secret-volume
    secret:
      secretName: my-app-secret
volumeMounts:
  - name: secret-volume
    mountPath: "/etc/secrets"
    readOnly: true
```

Applications can now read secrets from `/etc/secrets/db_password`.

### Auto-Refreshing Secrets with Projected Volumes

Secrets do not update automatically in running pods unless **projected volumes** are used. The following example demonstrates how to enable automatic refresh of Secrets:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: auto-refresh-pod
spec:
  containers:
  - name: app-container
    image: my-app-image
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/secrets"
      readOnly: true
  volumes:
  - name: secret-volume
    projected:
      sources:
      - secret:
          name: my-app-secret
      - configMap:
          name: my-config
    periodSeconds: 30
```

This ensures that secrets are **refreshed every 30 seconds** inside the container.

## Best Practices for Secrets

- **Restrict Access:** Limit access to secrets using RBAC.
- **Rotate Secrets:** Regularly update and rotate sensitive data.
- **Use TLS for Communication:** Encrypt data in transit.
- **Enable Encryption at Rest:** Configure Kubernetes to encrypt secrets stored in etcd.
- **Use Immutable Secrets:** Prevent accidental modifications using:

```yaml
immutable: true
```

## Troubleshooting Secret Issues

### Common Issues & Fixes

| Issue | Cause | Solution |
|-------|-------|----------|
| Secret not found | Incorrect name reference | Check `kubectl get secret` |
| Secret not updating | Pod is not restarted | Use projected volumes or restart pod |
| Access denied | RBAC restrictions | Review role bindings |

## kubectl create secret Tutorial

### Creating from Literal Values

```bash
kubectl create secret generic <secret-name> --from-literal=<key1>=<value1> --from-literal=<key2>=<value2>
```

### Creating from Files

```bash
kubectl create secret generic <secret-name> --from-file=<file1> --from-file=<file2>
```

### Creating Docker-registry Secrets

```bash
kubectl create secret docker-registry <secret-name> --docker-server=<server> --docker-username=<username> --docker-password=<password> --docker-email=<email>
```

## Conclusion

In this chapter, we explored Kubernetes Secrets, their use cases, hands-on examples, best practices, troubleshooting techniques, and security measures. By properly managing secrets, you can enhance the security of your Kubernetes applications and infrastructure.
