
### Chapter 14: StatefulSets

#### Objective
Explore StatefulSets for managing stateful applications, which require stable, unique network identifiers, stable persistent storage, and ordered, graceful deployment and scaling.

#### Understanding StatefulSets vs. Deployments
- **Theory:**
  - **Definition and Use Cases:** Explain what StatefulSets are and when to use them over Deployments. StatefulSets are ideal for applications like databases, key-value stores, and anything requiring a stable state or identity.
  - **Characteristics of StatefulSets:** Discuss the guarantees about ordering, uniqueness, and stability that StatefulSets provide.

#### Components of a StatefulSet
- **Headless Service:** Explain the necessity of a headless service for controlling the network domain of the StatefulSet.
- **Pod Management:** Detail how pods are named and managed systematically within a StatefulSet.

#### Lifecycle of a StatefulSet Pod
- **Creation Sequence:** Describe how pods within a StatefulSet are created sequentially and how they must be healthy before moving on to the next one.
- **Scaling and Updates:** Explain how scaling out adds pods according to a stable, predictable order and how updates are handled gracefully.

### Hands-on Examples

#### 1. Deploying a Simple Stateful Application
- **Goal:** Deploy a MongoDB StatefulSet to demonstrate the use of stable storage and ordered deployment.
- **CLI Commands:**
  ```bash
  kubectl apply -f mongodb-statefulset.yaml
  ```
- **Manifest Code (mongodb-statefulset.yaml):**
  ```yaml
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: mongo
    namespace: default
  spec:
    serviceName: "mongo"
    replicas: 3
    selector:
      matchLabels:
        app: mongo
    template:
      metadata:
        labels:
          app: mongo
      spec:
        containers:
        - name: mongo
          image: mongo:latest
          ports:
          - containerPort: 27017
          volumeMounts:
          - name: mongo-persistent-storage
            mountPath: /data/db
    volumeClaimTemplates:
    - metadata:
        name: mongo-persistent-storage
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 1Gi
  ```

#### 2. Simulating StatefulSet Scaling
- **Goal:** Scale the MongoDB StatefulSet up and down and observe how the StatefulSet handles the addition and removal of pods.
- **CLI Commands for Scaling:**
  ```bash
  kubectl scale statefulsets mongo --replicas=5
  kubectl scale statefulsets mongo --replicas=2
  ```

#### 3. Updating StatefulSet Configurations
- **Goal:** Perform a rolling update on the MongoDB StatefulSet to upgrade to a newer Mongo image.
- **CLI Commands for Update:**
  ```bash
  kubectl patch statefulset mongo -p '{"spec":{"template":{"spec":{"containers":[{"name":"mongo","image":"mongo:4.4"}]}}}}'
  ```

#### Summary
This chapter delves into StatefulSets, a key resource for managing stateful applications in Kubernetes, demonstrating how they offer unique capabilities like stable storage, predictable naming, and orderly deployment and scaling.

### Conclusion
Understanding and utilizing StatefulSets is crucial for deploying applications that require persistence, identity, and ordered execution, ensuring high availability and resilience.
