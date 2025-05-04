# Integrating GitLab with Kubernetes

Integrating GitLab CI/CD with Kubernetes streamlines the process of deploying applications to a Kubernetes cluster like Google Kubernetes Engine (GKE). This guide covers how to set up GitLab CI/CD Runner on GKE, configure the runner to deploy to the Kubernetes cluster, and use `kubectl` and Helm for automated deployment.

## 1. **GitLab CI/CD Runner on GKE**

### 1.1 **Setting up GitLab Runner on Kubernetes using Helm or YAML**

To use GitLab CI/CD with Kubernetes, you need to install GitLab Runner in your GKE cluster. You can install the runner using either Helm or YAML files.

#### **Using Helm to Install GitLab Runner**

1. **Install Helm**:

If Helm is not yet installed, follow the [Helm installation guide](https://helm.sh/docs/intro/install/) to install it on your local machine.

2. **Add GitLab Helm Chart Repository**:

```bash
helm repo add gitlab https://charts.gitlab.io
helm repo update
```

3. **Install GitLab Runner using Helm**:

```bash
helm install gitlab-runner gitlab/gitlab-runner --namespace gitlab --create-namespace
```

This command installs GitLab Runner in the `gitlab` namespace. You can customize the Helm chart values as per your requirements by modifying the values file.

#### **Using YAML to Install GitLab Runner**

Alternatively, you can install GitLab Runner using a YAML configuration.

1. **Download the GitLab Runner YAML file**:

```bash
kubectl apply -f https://gitlab.com/gitlab-org/ci-cd/kubernetes.gitlab.io/raw/main/ci-cd/runner.yaml
```

2. **Check GitLab Runner Pods**:

```bash
kubectl get pods -n gitlab
```

Ensure that the GitLab Runner pods are running correctly.

### 1.2 **Configuring GitLab Runner to Deploy to Kubernetes Cluster**

Once the GitLab Runner is installed, you need to configure it to interact with your GKE cluster to deploy applications.

1. **Configure Kubernetes Access in GitLab Runner**:

To allow the GitLab Runner to interact with Kubernetes, you need to set up a service account with the necessary permissions in GKE. You can create a Kubernetes secret to store the kubeconfig for access:

```bash
kubectl create secret generic gitlab-kubeconfig \
  --from-file=kubeconfig=/path/to/kubeconfig \
  --namespace gitlab
```

2. **Set Kubernetes Executor in GitLab Runner Configuration**:

In the GitLab Runner configuration file (`config.toml`), configure the runner to use the Kubernetes executor and point it to the Kubernetes cluster:

```toml
[[runners]]
  name = "Kubernetes Runner"
  url = "https://gitlab.com/"
  token = "YOUR_REGISTRATION_TOKEN"
  executor = "kubernetes"
  [runners.kubernetes]
    image = "docker:latest"
    namespace = "gitlab"
    service_account = "gitlab-runner"
    privileged = true
```

This configuration ensures the GitLab Runner is ready to deploy to your Kubernetes cluster using the specified `kubeconfig` and service account.

---

## 2. **Auto Deployment to Kubernetes**

Once the GitLab Runner is configured, you can automate deployments to your GKE cluster directly from GitLab CI/CD pipelines.

### 2.1 **Deploying Applications Directly to Kubernetes from GitLab Pipeline**

In the `.gitlab-ci.yml` file, define jobs to deploy applications to Kubernetes using `kubectl` commands.

Here is an example of a pipeline that builds a Docker image, pushes it to Google Container Registry (GCR), and deploys it to Kubernetes:

```yaml
stages:
  - build
  - deploy

build:
  stage: build
  script:
    - docker build -t gcr.io/$CI_PROJECT_ID/my-app .
    - docker push gcr.io/$CI_PROJECT_ID/my-app

deploy:
  stage: deploy
  script:
    - kubectl apply -f k8s/deployment.yaml
    - kubectl set image deployment/my-app my-app=gcr.io/$CI_PROJECT_ID/my-app:$CI_COMMIT_SHA
  only:
    - master
```

This pipeline performs the following:
- **Builds the Docker image** for the application.
- **Pushes the image** to Google Container Registry (GCR).
- **Applies the Kubernetes deployment** and updates the container image in the Kubernetes deployment.

### 2.2 **Using kubectl in the Pipeline to Deploy/Update Applications in GKE**

You can add `kubectl` commands to your pipeline for deploying or updating Kubernetes resources such as deployments, services, or config maps.

Example to update a deployment with a new Docker image:

```yaml
deploy:
  script:
    - kubectl set image deployment/my-app my-app=gcr.io/$CI_PROJECT_ID/my-app:$CI_COMMIT_SHA
    - kubectl rollout status deployment/my-app
```

This ensures the Kubernetes deployment is updated with the new image, and the rollout status is checked to confirm successful deployment.

### 2.3 **Setting Up Helm Charts in the Pipeline for Automatic Deployment**

Helm can simplify deployment in Kubernetes by using pre-defined charts. You can automate Helm-based deployments in the GitLab pipeline.

1. **Install Helm in the GitLab Runner**:

To use Helm, ensure that Helm is installed in the GitLab CI/CD Runner by adding the following before-script:

```yaml
before_script:
  - curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

2. **Define Helm Deployment in the Pipeline**:

In the `.gitlab-ci.yml` file, you can now use Helm commands to deploy the application to Kubernetes:

```yaml
deploy:
  stage: deploy
  script:
    - helm repo add my-repo https://charts.my-repo.com
    - helm upgrade --install my-app my-repo/my-app --namespace my-namespace --set image.tag=$CI_COMMIT_SHA
```

This script:
- Adds the Helm chart repository.
- Uses `helm upgrade` to install or upgrade the application with the new image tag.

---

## Summary

### Key Points:

- **GitLab Runner on Kubernetes**: Install and configure GitLab Runner in GKE using Helm or YAML, and set it up to interact with Kubernetes clusters.
- **Auto Deployment to Kubernetes**: Automate application deployment using `kubectl` in the GitLab CI/CD pipeline.
- **Helm for Deployment**: Helm charts can be integrated into the GitLab pipeline to simplify deployments to Kubernetes.

---