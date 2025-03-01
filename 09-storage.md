# Chapter 9: Mastering Storage in Kubernetes

## **Introduction**

Data in containerized environments like Kubernetes suffers a similar fate – pods come and go, but your precious information deserves a permanent home. Enter the realm of Kubernetes storage, where your digital treasures can find sanctuary. This chapter unveils the secrets of persistent data management, empowering you to build resilient and scalable applications.

---

## **1. Ephemeral vs. Persistent Storage**

Think of containers as rented apartments. You keep essentials like a toothbrush, but the landlord expects them gone upon checkout. That's **ephemeral data**. Conversely, **persistent data** is your family photo album – something you want to preserve beyond temporary stays.

---

## **2. Types of Storage in Kubernetes**

Kubernetes provides multiple storage options, depending on the application's requirements. Here are some common types:

### **2.1 EmptyDir (Ephemeral)**
- Temporary storage that exists as long as the Pod is running.
- When the Pod is deleted, the data is lost.

#### **Example: Pod using EmptyDir**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: [ "sh", "-c", "sleep 3600" ]
    volumeMounts:
    - mountPath: "/cache"
      name: temp-storage
  volumes:
  - name: temp-storage
    emptyDir: {}
```

---

### **2.2 HostPath (Accessing Node’s Filesystem)**
- Directly accesses the node’s file system.
- **Security risk**: Use carefully.

#### **Example: Pod using HostPath**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: [ "sh", "-c", "sleep 3600" ]
    volumeMounts:
    - mountPath: "/data"
      name: host-volume
  volumes:
  - name: host-volume
    hostPath:
      path: "/mnt/data" # Path on the node
      type: DirectoryOrCreate
```

---

### **2.3 Network Storage (Shared Storage)**
- Used for **multi-pod access across multiple nodes**.
- Examples: **NFS, GlusterFS, CephFS**.

#### **Example: Persistent Volume for NFS**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: "/exported/path"
    server: "192.168.1.100"
```

---

### **2.4 Cloud Storage (Dynamic & Scalable)**
- Managed storage solutions from **AWS, Azure, Google Cloud**.
- Supports **dynamic provisioning** using StorageClasses.

#### **Example: StorageClass for AWS EBS**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: aws-ebs-sc
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
```

---

## **3. PersistentVolumes (PVs) and PersistentVolumeClaims (PVCs)**

Imagine **PersistentVolumes (PVs)** as storage units (apartments) and **PersistentVolumeClaims (PVCs)** as lease agreements.

### **3.1 Example PersistentVolume YAML**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-persistent-volume
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/mnt/data"
```

### **3.2 Example PersistentVolumeClaim YAML**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-data-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

---

## **4. Hands-on Minikube Experiment**

### **1. Apply the PV and PVC**
```sh
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
```

### **2. Verify PVC Binding to PV**
```sh
kubectl get pv
kubectl get pvc
```

### **3. Deploy a Pod Using the PVC**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: my-container
    image: nginx
    volumeMounts:
    - name: my-data-claim
      mountPath: /data
  volumes:
  - name: my-data-claim
    persistentVolumeClaim:
      claimName: my-data-claim
```

### **4. Write Data Into the Mounted Volume**
```sh
kubectl exec -it my-app -- /bin/sh
echo "Hello, Persistent Storage!" > /data/testfile.txt
exit
```

### **5. Restart the Pod and Verify Data Persistence**
```sh
kubectl delete pod my-app
kubectl apply -f pod.yaml
kubectl exec -it my-app -- /bin/sh
cat /data/testfile.txt
```

---

## **5. StorageClasses and Dynamic Provisioning**

### **Example: AWS EBS StorageClass**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-storage-class
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Delete
```

---

## **6. Troubleshooting Persistent Storage Issues**

## **PersistentVolume (PV), PersistentVolumeClaim (PVC), and Volume Binding**

### **How Binding Works in Kubernetes**
1. **A PersistentVolume (PV)** is a cluster-wide storage resource.
2. **A PersistentVolumeClaim (PVC)** is a request for storage by a Pod.
3. **Kubernetes automatically binds PVCs to suitable PVs** based on:
   - **Access modes** (`ReadWriteOnce`, `ReadOnlyMany`, `ReadWriteMany`).
   - **Storage capacity** (`1Gi`, `10Gi`, etc.).
   - **StorageClass** (if defined).
   - **Node affinity (if applicable)**.

### **Example: PV with Binding Mode**
The `volumeBindingMode` setting in a **StorageClass** affects **when** a PVC is bound to a PV:
- `Immediate`: PVC is **bound to a PV immediately**.
- `WaitForFirstConsumer`: PVC is **not bound until a Pod requests it**, optimizing scheduling.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-binding-sc
provisioner: kubernetes.io/aws-ebs
volumeBindingMode: WaitForFirstConsumer # Delays PV binding until a Pod is scheduled
```

### **Example: PVC with a Specific StorageClass**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: delayed-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: delayed-binding-sc
```

---

| **Issue** | **Troubleshooting Command** | **Possible Fix** |
|-----------|----------------------------|------------------|
| PVC stuck in `Pending` | `kubectl describe pvc <name>` | Ensure a matching PV exists & StorageClass is correct |
| PV not binding to PVC | `kubectl get pv,pvc` | Check if PV storage class, access mode, and capacity match the PVC |
| Pod unable to mount PVC | `kubectl describe pod <name>` | Check Events for mount errors and verify the correct PVC is used |
| PV not reclaiming | `kubectl get pv` | Change reclaim policy to `Retain` |
| StorageClass not provisioning PVs | `kubectl get sc` | Ensure dynamic provisioning is supported |
| PV binding delayed | `kubectl describe pvc <name>` | Check if `volumeBindingMode: WaitForFirstConsumer` is preventing binding until Pod is scheduled |
| Node affinity issues | `kubectl describe pv <name>` | Ensure the PV's `nodeAffinity` matches where the Pod is scheduled |

---

## **7. Conclusion**

Mastering Kubernetes storage unlocks possibilities for **building resilient and scalable applications**. By leveraging **PersistentVolumes, StorageClasses, Node Affinity, and dynamic provisioning**, you can optimize storage for both performance and security.

---
