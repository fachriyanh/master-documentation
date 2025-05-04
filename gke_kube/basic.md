# Basic Kubernetes

Kubernetes is an open-source platform that automates deploying, scaling, and managing containerized applications. It is widely used in container orchestration, and understanding the fundamental components is essential for working with Kubernetes effectively.

This section will cover the core concepts such as **Pods**, **Deployments**, **Services**, **Ingress**, **Namespaces**, **ConfigMaps**, **Secrets**, **Volumes**, as well as an understanding of how `kubectl` works and how Kubernetes deployment strategies are implemented.

## 1. **Core Concepts in Kubernetes**

### 1.1 **Pods**

A **Pod** is the smallest and most basic unit in Kubernetes. A Pod can hold one or more containers that share the same network namespace and storage. Containers in the same Pod can communicate with each other using `localhost` and share storage volumes.

- **Pod Definition Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
```

### 1.2 **Deployments**

A **Deployment** in Kubernetes is a higher-level abstraction that manages the lifecycle of Pods. It provides features such as scaling, rolling updates, and rollback to previous versions.

- **Deployment Definition Example:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
      - name: example-container
        image: nginx
```

### 1.3 **Services**

A **Service** in Kubernetes is an abstraction that defines a set of Pods and a way to access them. It ensures that requests to a set of Pods are load-balanced and accessible.

- **Service Definition Example:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-service
spec:
  selector:
    app: example
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

### 1.4 **Ingress**

An **Ingress** is an API object that manages external access to services in a cluster, typically HTTP and HTTPS. Ingress allows fine-grained control over routing, SSL termination, and other configurations.

- **Ingress Definition Example:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: example-service
            port:
              number: 80
```

### 1.5 **Namespaces**

A **Namespace** is a way to divide cluster resources between multiple users or teams. It provides isolation and helps manage resource quotas, access control, and naming conventions.

- **Namespace Definition Example:**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: example-namespace
```

### 1.6 **ConfigMaps**

A **ConfigMap** is an object used to store non-sensitive configuration data in key-value pairs. It is commonly used to pass configuration settings to Pods.

- **ConfigMap Definition Example:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-configmap
data:
  key1: value1
  key2: value2
```

### 1.7 **Secrets**

A **Secret** is used to store sensitive data, such as passwords, OAuth tokens, or SSH keys. It is similar to ConfigMaps but is specifically for storing sensitive information and is usually base64-encoded.

- **Secret Definition Example:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: example-secret
data:
  password: dGVzdHBhc3M=  # base64 encoded 'testpass'
```

### 1.8 **Volumes**

A **Volume** in Kubernetes is a directory that contains data, accessible by the containers in a Pod. It allows containers to share data and persist data across container restarts.

- **Volume Definition Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    volumeMounts:
    - mountPath: /data
      name: example-volume
  volumes:
  - name: example-volume
    emptyDir: {}
```

---

## 2. **Understanding kubectl**

The `kubectl` command-line tool is used to interact with Kubernetes clusters. It allows you to deploy applications, inspect and manage resources, and view logs.

### 2.1 **Common kubectl Commands**

- **Get Cluster Information:**

```bash
kubectl cluster-info
```

- **Get All Pods in the Cluster:**

```bash
kubectl get pods --all-namespaces
```

- **Get Deployment Information:**

```bash
kubectl get deployments
```

- **Apply Configurations:**

```bash
kubectl apply -f deployment.yaml
```

- **Delete Resources:**

```bash
kubectl delete pod example-pod
```

- **Get Logs from a Pod:**

```bash
kubectl logs example-pod
```

---

## 3. **Kubernetes Deployment Strategies**

### 3.1 **Rolling Updates**

A **Rolling Update** allows you to update your application by incrementally replacing old versions of Pods with new ones. This strategy minimizes downtime and ensures that a certain number of Pods are always running.

- **Rolling Update Example:**

In your `Deployment` configuration, Kubernetes performs rolling updates by default when you change the image or other parameters.

```yaml
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

### 3.2 **Canary Releases**

A **Canary Release** involves deploying a new version of your application to a small subset of users first (the "canaries"). If the new version works well, the release can be rolled out to all users.

You can implement a Canary release using the `Deployment` strategy with **two separate deployments**:

1. Deploy the new version to a subset of Pods (Canary).
2. Gradually shift traffic to the new Pods.

This can be configured with Kubernetes `Ingress` rules or a service mesh like Istio for more advanced routing.

### 3.3 **Blue/Green Deployment**

A **Blue/Green Deployment** involves having two environments (Blue and Green). The Blue environment is the live version, while the Green environment is the new version. Once the Green environment is ready, traffic is switched from Blue to Green.

- **Blue/Green Deployment Workflow:**
  - Deploy the new version of your app (Green) without affecting the current version (Blue).
  - Switch traffic from Blue to Green by updating the service or load balancer to point to the Green environment.

This strategy is useful for ensuring zero-downtime deployments and quick rollback if something goes wrong.

---

## Summary of Kubernetes Core Concepts

| **Concept**        | **Description**                                                                  |
|--------------------|----------------------------------------------------------------------------------|
| **Pods**           | Smallest unit in Kubernetes that holds containers.                               |
| **Deployments**    | Manages the lifecycle of Pods, including scaling and rolling updates.           |
| **Services**       | Defines access to Pods and handles load balancing.                              |
| **Ingress**        | Manages external access to services, typically via HTTP/HTTPS.                   |
| **Namespaces**     | Isolates resources within a Kubernetes cluster for organizational purposes.     |
| **ConfigMaps**     | Stores non-sensitive configuration data for Pods.                               |
| **Secrets**        | Stores sensitive data like passwords and tokens.                                |
| **Volumes**        | Provides persistent storage for containers across restarts.                     |
| **kubectl**        | Command-line tool to interact with Kubernetes clusters.                         |

---

## Example Kubernetes Deployment for a Rolling Update

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      containers:
      - name: example-container
        image: nginx:v1
```

---
