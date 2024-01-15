
# Chapter 2: Secrets in Kubernetes

## Introduction

In the Kubernetes ecosystem, managing sensitive information is paramount to ensure the security of applications. Kubernetes provides a dedicated resource called **Secrets** for handling confidential data such as passwords, API keys, and TLS certificates. This chapter delves into the fundamentals of Secrets, explores their use cases, and provides practical examples to secure sensitive information within your Kubernetes cluster.

## What are Secrets?

Secrets in Kubernetes are resources designed to store sensitive information securely. They act as a safer alternative to ConfigMaps when dealing with confidential data. Secrets are encoded in Base64, adding a layer of obfuscation, though they are not encrypted. Common use cases for Secrets include storing database passwords, API keys, and TLS certificates.

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
kubectl create -f secret.yaml
```

## Verifying the Secret

To ensure the Secret has been created successfully, inspect its details:

```bash
kubectl get secret my-app-secret
```

## Using Secrets in Pods

Reference the Secret in your pod's configuration by adding the following environment variable:

```yaml
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: my-app-secret
      key: db_password
```

Now, your application can securely access the database password through the `DB_PASSWORD` environment variable.

## Accessing Secrets from the Command Line

To interact with Secrets via the command line, you can use `kubectl`:

```bash
kubectl get secret my-app-secret -o jsonpath='{.data.db_password}' | base64 --decode
```

This command retrieves the Base64-encoded value of `db_password` and decodes it to reveal the actual secret.

## Best Practices for Secrets

When working with Secrets, it's crucial to follow best practices to maintain a secure environment:

- **Restrict access:** Limit the number of users and applications with access to secrets.
- **Rotate secrets:** Regularly update and rotate sensitive information to minimize the impact of potential security breaches.
- **Use TLS for communication:** Encrypt communication between pods to safeguard sensitive data in transit.
- **Avoid hardcoding secrets:** Instead of hardcoding secrets directly in manifests, use tools like Helm to manage and deploy secrets securely.

## kubectl create secret Tutorial

Creating secrets using `kubectl` is straightforward. Here's a tutorial covering different methods:

### Creating from Literal Values

```bash
kubectl create secret generic <secret-name> --from-literal=<key1>=<value1> --from-literal=<key2>=<value2>
```

### Creating from Files

```bash
kubectl create secret generic <secret-name> --from-file=<file1> --from-file=<file2>
```

### Creating from Literal Values Interactively

```bash
kubectl create secret generic <secret-name> --from-literal=<key1>=<value1> --dry-run=client -o yaml | kubectl apply -f -
```

### Creating Docker-registry Secrets

```bash
kubectl create secret docker-registry <secret-name> --docker-server=<server> --docker-username=<username> --docker-password=<password> --docker-email=<email>
```

This tutorial covers creating secrets from literal values, files, and interacting with secrets interactively.

