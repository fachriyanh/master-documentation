# Security and Compliance

This section explains how to secure your Kubernetes environment on GKE and integrate vulnerability scanning within GitLab CI/CD.

---

## Kubernetes RBAC (Role-Based Access Control)

**RBAC** in Kubernetes controls who can access what resources in the cluster.

### Key Concepts:
- **Role**: Defines a set of permissions within a namespace.
- **ClusterRole**: Defines permissions across the entire cluster.
- **RoleBinding**: Grants a Role to a user or group within a namespace.
- **ClusterRoleBinding**: Grants a ClusterRole to a user or group cluster-wide.

### Example of a Role and RoleBinding:

```
- apiVersion: rbac.authorization.k8s.io/v1
- kind: Role
- metadata:
  - namespace: default
  - name: pod-reader
- rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]

- apiVersion: rbac.authorization.k8s.io/v1
- kind: RoleBinding
- metadata:
  - name: read-pods
  - namespace: default
- subjects:
  - kind: User
    name: "example-user"
    apiGroup: rbac.authorization.k8s.io
- roleRef:
  - kind: Role
    name: pod-reader
    apiGroup: rbac.authorization.k8s.io
```

---

## Pod Security Policies

**Pod Security Policies (PSPs)** control the security settings of Pods.

### Goals:
- Prevent privileged containers unless absolutely necessary.
- Enforce usage of read-only root filesystem.
- Restrict volume types.
- Limit container capabilities.

**Note**: PSPs are deprecated in Kubernetes 1.21 and removed in 1.25. Alternatives include **PodSecurity Admission** and third-party solutions like OPA Gatekeeper.

### Example PSP YAML:

```
- apiVersion: policy/v1beta1
- kind: PodSecurityPolicy
- metadata:
  - name: restricted
- spec:
  - privileged: false
  - runAsUser:
    - rule: MustRunAsNonRoot
  - seLinux:
    - rule: RunAsAny
  - fsGroup:
    - rule: MustRunAs
    - ranges:
      - min: 1
        max: 65535
  - volumes:
    - configMap
    - secret
    - persistentVolumeClaim
```

---

## Network Policies

**Network Policies** control traffic between Pods and to/from the cluster.

### Goals:
- Restrict pod-to-pod communication.
- Control ingress and egress traffic for enhanced security.

### Example NetworkPolicy:

```
- apiVersion: networking.k8s.io/v1
- kind: NetworkPolicy
- metadata:
  - name: allow-app-traffic
  - namespace: default
- spec:
  - podSelector:
      matchLabels:
        app: my-app
  - ingress:
    - from:
      - podSelector:
          matchLabels:
            app: my-frontend
```

### Important:
- GKE must be configured to use a network plugin that supports network policies (e.g., Calico).

---

## Vulnerability Scanning (Container Scanning with GitLab CI)

Perform container scanning automatically inside your GitLab CI/CD pipelines to catch vulnerabilities early.

### Setup Steps:
1. Use GitLab’s built-in Container Scanning template.
2. Configure the pipeline to scan Docker images after build.
3. Set up vulnerability thresholds to block deployments if critical issues are found.

**Example GitLab CI Job:**
```
- container_scanning:
  - stage: test
  - image: docker:stable
  - services:
    - docker:dind
  - variables:
    - DOCKER_DRIVER: overlay2
  - script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker scan $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  - only:
    - merge_requests
    - main
```

**Alternative**: Use GitLab’s `container-scanning` predefined template for more automated workflows.

---

# Summary

- Use **Kubernetes RBAC** to tightly control user and service account access.
- Apply **Pod Security Policies** or **PodSecurity Admission** to enforce container security standards.
- Configure **Network Policies** to restrict Pod communication.
- Integrate **Container Scanning** into GitLab CI to detect vulnerabilities early in the pipeline.

---