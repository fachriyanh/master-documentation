# Helm and GitOps

This section covers how to manage Kubernetes deployments using **Helm** and how to adopt **GitOps** practices for automated and declarative deployment workflows.

---

## Helm - Kubernetes Package Manager

**Helm** simplifies the management of Kubernetes applications by packaging them into reusable charts.

### Key Concepts:
- **Chart**: A package containing pre-configured Kubernetes resources (YAML templates).
- **Release**: An instance of a chart running in a Kubernetes cluster.
- **Repository**: A place where charts can be collected and shared.

### Common Helm Commands:
- Install a chart:
  
  helm install my-release my-chart/

- Upgrade an existing release:

  helm upgrade my-release my-chart/

- Uninstall a release:

  helm uninstall my-release

- Package a chart:

  helm package my-chart/

- Add a repository:

  helm repo add bitnami https://charts.bitnami.com/bitnami

### Using Helm in GitLab CI/CD:

Example job in `.gitlab-ci.yml`:
```
- deploy:
  - stage: deploy
  - image: alpine/helm:3.8.0
  - script:
    - helm upgrade --install my-app ./chart --namespace production --set image.tag=$CI_COMMIT_SHA
  - only:
    - main
```

---

## GitOps - Deployment by Git

**GitOps** is a way to do Continuous Deployment where Git is the source of truth for the desired state of the system.

### Key Principles:
- **Version Control**: All infrastructure and application configurations are stored in Git.
- **Automation**: Changes pushed to Git automatically trigger updates to the cluster.
- **Auditing**: Git history acts as an audit trail.

### Popular GitOps Tools:
- **ArgoCD**: Continuous delivery tool for Kubernetes using Git repositories.
- **FluxCD**: GitOps operator for Kubernetes that synchronizes Git with cluster state.

---

## GitOps Workflow Example

1. Developer updates Helm chart or Kubernetes manifest.
2. Push changes to Git repository (e.g., GitLab).
3. GitOps tool (ArgoCD or FluxCD) detects the change.
4. Tool automatically applies the updated configuration to the Kubernetes cluster.
5. Cluster state is reconciled to match the Git repository.

### ArgoCD Example Basic Installation:

- Install ArgoCD in GKE:

  kubectl create namespace argocd
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

- Access ArgoCD UI and set up your Git repository for deployment.

---

# Summary

- Use **Helm** to package, manage, and deploy Kubernetes applications efficiently.
- Adopt **GitOps** practices with tools like **ArgoCD** or **FluxCD** to automate and secure your deployments.
- Git becomes the single source of truth for your Kubernetes cluster state, enhancing traceability and control.

---