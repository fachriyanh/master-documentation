# Kubernetes Secrets and ConfigMaps

In this section, we explain how to manage sensitive information (secrets) and configuration settings (config maps) for Kubernetes applications via GitLab CI/CD pipelines.

---

## Managing Secrets and Configurations in Kubernetes

### 1. Kubernetes Secrets

**Kubernetes Secrets** are objects used to store sensitive data such as passwords, API keys, OAuth tokens, and SSH keys. Secrets provide a way to pass this data securely to Pods without hardcoding sensitive information inside the application code.

**Common use cases:**
- Database credentials
- API tokens
- Private keys

**Example Secret YAML:**
```
- apiVersion: v1
- kind: Secret
- metadata:
  - name: my-secret
- type: Opaque
- data:
  - username: base64_encoded_username
  - password: base64_encoded_password
```

**How to apply via kubectl in GitLab CI pipeline:**

```
- stage: deploy
- image: google/cloud-sdk:alpine
- script:
  - echo $GCP_SERVICE_KEY | base64 -d > $HOME/gcp-key.json
  - gcloud auth activate-service-account --key-file=$HOME/gcp-key.json
  - gcloud container clusters get-credentials $GKE_CLUSTER --zone $GKE_ZONE --project $GCP_PROJECT_ID
  - echo $K8S_SECRET_YAML | base64 -d > secret.yaml
  - kubectl apply -f secret.yaml
- only:
  - main
```

**Notes:**
- It's recommended to store the secret YAML as an encoded GitLab CI/CD variable.
- Rotate secrets regularly and limit access.

---

### 2. Kubernetes ConfigMaps

**Kubernetes ConfigMaps** are used to store non-sensitive configuration data separately from application code.

**Common use cases:**
- Application settings
- Environment-specific configurations
- Feature flags

**Example ConfigMap YAML:**

```
- apiVersion: v1
- kind: ConfigMap
- metadata:
  - name: my-config
- data:
  - ENVIRONMENT: production
  - LOG_LEVEL: debug
```

**How to apply via kubectl in GitLab CI pipeline:**

```
- stage: deploy
- image: google/cloud-sdk:alpine
- script:
  - echo $K8S_CONFIGMAP_YAML | base64 -d > configmap.yaml
  - kubectl apply -f configmap.yaml
- only:
  - main
```

---

## Using Google Secret Manager with GitLab CI/CD

**Google Secret Manager** is a secure and convenient service for managing secrets outside Kubernetes.  
You can fetch secrets dynamically inside your GitLab pipeline or during deployment.

### Steps:

1. **Store secrets** in Google Secret Manager.
2. **Grant access** to the GitLab CI/CD service account.
3. **Fetch secret** in GitLab pipeline at runtime.

**Example script to fetch a secret:**

```
- stage: prepare
- image: google/cloud-sdk:alpine
- script:
  - echo $GCP_SERVICE_KEY | base64 -d > $HOME/gcp-key.json
  - gcloud auth activate-service-account --key-file=$HOME/gcp-key.json
  - SECRET_VALUE=$(gcloud secrets versions access latest --secret="my-secret-name")
  - export DB_PASSWORD=$SECRET_VALUE
- only:
  - main
```

### Benefits:
- Secrets are not hardcoded into the repository or Kubernetes manifest files.
- Centralized management of sensitive data.
- Easy rotation without pipeline change.

---

# Summary

- Use **Kubernetes Secrets** to securely pass sensitive data to your applications.
- Use **Kubernetes ConfigMaps** for application configurations.
- Integrate with **Google Secret Manager** for dynamic and secure secret management inside GitLab CI/CD pipelines.
- Always encode YAML files if stored inside GitLab CI/CD variables.
- Follow **principle of least privilege** when giving access to secrets.

---