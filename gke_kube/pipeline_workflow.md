# CI/CD Pipeline Workflow for Kubernetes

This section explains how to build a complete CI/CD pipeline that builds, tests, pushes, and deploys applications to Kubernetes on GKE, fully automated via GitLab CI/CD.

## Pipeline Stages Overview

A typical Kubernetes CI/CD pipeline consists of the following stages:

1. **Build**: Build the application, run tests, and create a Docker image.
2. **Push**: Push the Docker image to Google Container Registry (GCR) or Artifact Registry.
3. **Deploy**: Deploy the updated application to GKE using `kubectl` or Helm.

---

## Build Stage

Tasks performed during the **Build** stage:
- Compile the application (if applicable).
- Run unit tests, integration tests, and other validations.
- Build the Docker image containing the application.

**Example Build Job:**

```
- image: docker:latest
- services:
  - docker:dind
- variables:
  - DOCKER_DRIVER: overlay2
- build:
  - stage: build
  - script:
    - docker build -t gcr.io/$GCP_PROJECT_ID/$IMAGE_NAME:$CI_COMMIT_SHORT_SHA .
  - only:
    - merge_requests
    - main
```

**Highlights:**
- Use Docker-in-Docker (`docker:dind`) for building images.
- Tag images based on Git commit SHA.

---

## Push Stage

Tasks performed during the **Push** stage:
- Authenticate with Google Cloud.
- Push the Docker image to GCR.

**Example Push Job:**

```
- push:
  - stage: push
  - script:
    - echo $GCP_SERVICE_KEY | base64 -d > $HOME/gcp-key.json
    - gcloud auth activate-service-account --key-file=$HOME/gcp-key.json
    - gcloud auth configure-docker
    - docker push gcr.io/$GCP_PROJECT_ID/$IMAGE_NAME:$CI_COMMIT_SHORT_SHA
  - only:
    - merge_requests
    - main
```

**Highlights:**
- Use base64-encoded GCP service account keys stored in GitLab variables.
- Push securely to GCR/Artifact Registry.

---

## Deploy Stage

Tasks performed during the **Deploy** stage:
- Apply Kubernetes manifests or Helm charts.
- Update the running application on the GKE cluster.

### Example Deploy Job Using kubectl:

```
- deploy:
  - stage: deploy
  - image: google/cloud-sdk:alpine
  - script:
    - echo $GCP_SERVICE_KEY | base64 -d > $HOME/gcp-key.json
    - gcloud auth activate-service-account --key-file=$HOME/gcp-key.json
    - gcloud container clusters get-credentials $GKE_CLUSTER --zone $GKE_ZONE --project $GCP_PROJECT_ID
    - kubectl set image deployment/$K8S_DEPLOYMENT_NAME $K8S_CONTAINER_NAME=gcr.io/$GCP_PROJECT_ID/$IMAGE_NAME:$CI_COMMIT_SHORT_SHA
  - only:
    - main
```

### Example Deploy Job Using Helm:

```
- deploy_helm:
  - stage: deploy
  - image: alpine/helm:3.8.2
  - script:
    - helm upgrade --install $RELEASE_NAME ./helm-chart-directory --namespace $K8S_NAMESPACE --set image.tag=$CI_COMMIT_SHORT_SHA
  - only:
    - main
```

**Highlights:**
- For `kubectl`, directly patch the Deployment's container image.
- For `helm`, upgrade or install charts dynamically based on branch/tag.

---

## Best Practices

- Use GitLab **environments** to track deployments visually.
- Store secrets (GCP keys, registry credentials) securely using **GitLab CI/CD Variables**.
- Optimize Docker images with **multi-stage builds**.
- Implement **rollback strategies** (manual approvals for production).

---

# Summary

A well-structured Kubernetes pipeline should:
- **Build** the application and Docker images efficiently.
- **Push** images securely to GCR/Artifact Registry.
- **Deploy** updates reliably into GKE clusters with minimal downtime.
- Maintain focus on **security**, **scalability**, and **automation**.

