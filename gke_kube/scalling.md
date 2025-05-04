# Scaling and Auto-scaling in GKE

This section explains how to implement scaling strategies in Google Kubernetes Engine (GKE) using both application-level and cluster-level auto-scaling tools.

---

## Horizontal Pod Autoscaler (HPA)

**Horizontal Pod Autoscaler (HPA)** automatically scales the number of Pods in a deployment based on observed CPU utilization or other select metrics.

### Key Concepts:
- Adjusts the number of Pod replicas dynamically.
- Based on resource usage metrics like CPU, memory, or custom metrics.
- Helps ensure high availability and efficient resource usage.

### Example Configuration:
```
- apiVersion: autoscaling/v2
- kind: HorizontalPodAutoscaler
- metadata:
  - name: my-app-hpa
- spec:
  - scaleTargetRef:
    - apiVersion: apps/v1
    - kind: Deployment
    - name: my-app
  - minReplicas: 2
  - maxReplicas: 10
  - metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

**Apply in GitLab CI/CD pipeline:**
```
- stage: deploy
- image: google/cloud-sdk:alpine
- script:
  - echo $HPA_YAML | base64 -d > hpa.yaml
  - kubectl apply -f hpa.yaml
- only:
  - main
```
---

## Cluster Autoscaler

**Cluster Autoscaler** automatically adjusts the number of nodes in your GKE cluster depending on the resource requests from Pods.

### Key Concepts:
- Adds nodes when Pods are pending and cannot be scheduled.
- Removes nodes when they are underutilized and Pods are rescheduled elsewhere.
- Ensures cluster cost-efficiency and scalability.

### Setup Steps:
1. Enable **Autoscaling** when creating the GKE cluster.
2. Configure minimum and maximum number of nodes.
3. GKE automatically manages scaling without manual intervention.

**Example command to create an autoscaled cluster:**
```
gcloud container clusters create my-cluster
--zone us-central1-a
--enable-autoscaling
--min-nodes 1
--max-nodes 5
--num-nodes 3
```
---

## GKE Autopilot Mode

**GKE Autopilot** manages the entire cluster infrastructure, including node provisioning and scaling, without requiring manual configuration.

### Features:
- No manual node pool management.
- Automatic scaling based on Pod requests.
- Pay per actual usage (CPU, memory).

### Benefits:
- Simplifies operational overhead.
- Best for teams that want to focus only on application-level concerns.
- Integrated seamlessly with GitLab CI/CD for deployment and scaling.

**Deployment Flow with Autopilot:**
1. Create a GKE Autopilot cluster.
2. Deploy applications as usual with Kubernetes manifests.
3. GKE handles node provisioning and scaling behind the scenes.

**Example command to create an Autopilot cluster:**
```
gcloud container clusters create-auto my-autopilot-cluster
--region us-central1
```
---

# Summary

- Use **Horizontal Pod Autoscaler (HPA)** for dynamic Pod scaling based on metrics.
- Use **Cluster Autoscaler** to automatically adjust the number of nodes in the cluster.
- Consider **GKE Autopilot** if you want fully managed node scaling and provisioning.
- Integrate HPA YAMLs and kubectl commands inside your GitLab CI/CD pipelines for full automation.