# Chapter 7: Deploying and exposing Your First Containerized App - Hello, Nginx in Kubernetes!

## Preparing for Deployment

- Ensure your Minikube cluster is running (from Chapter 3).
- While `kubectl` is optional, it's recommended for easier interactions.

## Deployment Steps

1. Create a deployment manifest:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-hello
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-hello
  template:
    metadata:
      labels:
        app: nginx-hello
    spec:
      containers:
      - name: nginx
        image: nginx:latest  # Using the official Nginx image
        ports:
        - containerPort: 80
```

2. Apply the manifest:

```
kubectl apply -f deployment.yaml
```

## Exposing Your Application

1. Create a service:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-hello-service
spec:
  selector:
    app: nginx-hello
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # Optional for external access
  type: NodePort  # Optional for external access
```

2. Apply the service manifest:

```
kubectl apply -f service.yaml
```

## Accessing Nginx

1. Get the service endpoint:

```
kubectl get service nginx-hello-service
```

Note the "EXTERNAL-IP" or "NODE-PORT" address.

2. Open the endpoint in your browser:

- If using NodePort: Access `http://[NODE-IP]:30080` (replace with actual Node IP).
- If using LoadBalancer (if available): Access the EXTERNAL-IP.

You should see the default Nginx welcome page!

## Congratulations!

You've successfully deployed a pre-built Nginx container on Kubernetes!

## Remember

- Explore scaling and managing resources in future chapters.
- Experiment with different container images and deployment configurations.
- Keep learning and building your Kubernetes expertise!

