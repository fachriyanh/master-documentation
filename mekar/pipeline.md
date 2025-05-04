# **Subscriber Disbursement CI/CD Pipeline Documentation**

This documentation outlines the **CI/CD pipeline** and **Kubernetes deployment** for the **subscriber-disbursement** service. The pipeline automates tasks like linting, building the Docker image, and deploying it to **Google Kubernetes Engine (GKE)**. Here’s an in-depth breakdown of how everything works.

---

## **CI/CD Pipeline Overview (`gitlab-ci.yaml`)**

The CI/CD pipeline is defined in the `gitlab-ci.yaml` file and consists of three stages:

1. **Pre Stage** (`validate_subs`): Linting and testing the code.
2. **Build Stage** (`build_subs`): Building the Docker image.
3. **Deploy Stage** (`deploy_subs`): Deploying the Docker image to GKE using Helm.

---

## **File Breakdown**

### **1. `gitlab-ci.yaml` - CI/CD Pipeline**

This file defines the pipeline stages in GitLab CI.

#### `validate_subs` Stage

```yaml
validate_subs:
  image: golang:1.24
  script:
    - chmod +x validate_sub.sh
    - ./validate_sub.sh
  rules:
    - if: '$CI_COMMIT_BRANCH != "main"'
```

**Explanation:**

* **Purpose**: Runs linting and tests to ensure that the Go code is correct.
* **Image**: Uses the `golang:1.24` image to run the Go tools.
* **Script**: The `validate_sub.sh` script is executed to format the code (`go fmt`), check for code issues (`go vet`), and run tests (`go test`).
* **Rules**: The stage runs for any branch except `main`.

#### `build_subs` Stage

```yaml
build_subs:
  image: gcr.io/kaniko-project/executor:debug
  script:
    - chmod +x build_subs.sh
    - ./build_subs.sh
  only:
    - staging
```

**Explanation:**

* **Purpose**: Builds the Docker image for the service.
* **Image**: Uses `gcr.io/kaniko-project/executor:debug`, a Kaniko image to build Docker images in a Kubernetes environment.
* **Script**: Runs `build_subs.sh` to create the Docker image and push it to Google Container Registry (GCR).
* **Only**: This stage is triggered only when changes are pushed to the `staging` branch.

#### `deploy_subs` Stage

```yaml
deploy_subs:
  image: google/cloud-sdk:latest
  script:
    - chmod +x deploy_sub.sh
    - ./deploy_sub.sh
  only:
    - staging
```

**Explanation:**

* **Purpose**: Deploys the service to GKE using Helm.
* **Image**: Uses `google/cloud-sdk` to interact with Google Cloud.
* **Script**: Executes the `deploy_sub.sh` script to deploy the service using Helm, applying configurations from the Kubernetes templates.
* **Only**: This stage is triggered only when changes are pushed to the `staging` branch.

---

### **2. `build_subs.sh` - Building the Docker Image**

```bash
#!/bin/sh
set -e

echo "$GCP_SA_KEY_REGISTRY" > /kaniko/gcp-key.json

cat > /kaniko/.docker/config.json <<EOF
{
  "credHelpers": {
    "asia-southeast2-docker.pkg.dev": "gcr"
  }
}
EOF

export GOOGLE_APPLICATION_CREDENTIALS=/kaniko/gcp-key.json

/kaniko/executor \
  --context subscriber \
  --dockerfile Dockerfile \
  --destination "$IMAGE_TAG" \
  --destination "$IMAGE_LATEST"
```

**Explanation:**

* **Purpose**: Builds the Docker image using Kaniko (a tool for building Docker images in Kubernetes).
* **Steps**:

  1. **Google Service Account Key**: The script writes the service account key (`GCP_SA_KEY_REGISTRY`) into a file at `/kaniko/gcp-key.json`.
  2. **Docker Config**: Creates a Docker configuration file with credential helpers for GCR.
  3. **Build Image**: The `kaniko/executor` is used to build the Docker image from the `Dockerfile` and push it to the specified destination (`$IMAGE_TAG` and `$IMAGE_LATEST`).

**Important Configuration:**

* **GCP Service Account Key**: Set the `$GCP_SA_KEY_REGISTRY` environment variable in the CI/CD pipeline settings to the base64-encoded service account key JSON.
* **Kaniko**: Kaniko requires a service account key that grants access to GCR. Make sure that this account has the required permissions.

---

### **3. `deploy_sub.sh` - Deploying to GKE**

```bash
#!/bin/sh
set -e

echo "$GCP_SA_KEY_REGISTRY_64" | base64 -d > gcloud-key.json
gcloud auth activate-service-account --key-file=gcloud-key.json
gcloud config set project mekar-dev

echo "🔐 Installing GKE Auth Plugin"
apt-get update && apt-get install -y google-cloud-sdk-gke-gcloud-auth-plugin
export USE_GKE_GCLOUD_AUTH_PLUGIN=True

echo "🔗 Connecting to GKE cluster"
gcloud container clusters get-credentials mekar-reachitecture-dev \
  --region asia-southeast2 \
  --project mekar-dev

echo "📦 Installing Helm"
curl -sSL https://get.helm.sh/helm-v3.17.3-linux-amd64.tar.gz | tar -xz
mv linux-amd64/helm /usr/local/bin/helm

echo "📁 Ensuring namespace exists"
kubectl get namespace staging || kubectl create namespace staging

echo "🚀 Deploying using Helm"
helm upgrade --install subscriber-disbursement ./subscriber/deployment \
  --namespace staging \
  --set image.tag=$CI_COMMIT_SHORT_SHA
```

**Explanation:**

* **Purpose**: Deploys the Docker image to Google Kubernetes Engine (GKE) using Helm.
* **Steps**:

  1. **Activate Service Account**: The service account key (`GCP_SA_KEY_REGISTRY_64`) is decoded from base64 and used for authentication.
  2. **GKE Cluster Access**: The script uses `gcloud` to authenticate and connect to the GKE cluster.
  3. **Helm Installation**: Helm is installed on the environment if it isn’t already installed.
  4. **Namespace Creation**: Ensures that the `staging` namespace exists in the cluster.
  5. **Helm Deploy**: Deploys the application using Helm, applying the `image.tag` dynamically from the commit SHA.

**Important Configuration:**

* **Service Account Key**: Set the `$GCP_SA_KEY_REGISTRY_64` environment variable in the CI/CD pipeline settings as the base64-encoded service account key JSON.

---

### **4. `validate_sub.sh` - Code Validation**

```bash
#!/bin/sh
cd subscriber

echo "🔍 Linting & Testing"
go fmt ./...
go vet ./...
go test ./...
```

**Explanation:**

* **Purpose**: Ensures that the Go code is properly formatted, free of issues, and passes the tests.
* **Steps**:

  1. `go fmt ./...`: Formats the Go code.
  2. `go vet ./...`: Checks for potential issues in the Go code.
  3. `go test ./...`: Runs tests to ensure the code functions correctly.

---

## **Kubernetes Resources**

### **1. `deployment.yaml` - Kubernetes Deployment**

Defines how the service is deployed within the Kubernetes cluster.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.app.name }}
  namespace: {{ .Values.namespace }}
spec:
  replicas: {{ .Values.replicaCount }}
  containers:
    - name: {{ .Values.app.label }}
      image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
      ports:
        - containerPort: {{ .Values.probes.port }}
      livenessProbe:
        httpGet:
          path: /health
          port: {{ .Values.probes.port }}
```

**Explanation:**

* Defines the deployment settings like the number of replicas, the Docker image to use, and the health probes.
* **Important**: This template will pull the image with the tag set by the pipeline.

### **2. `hpa.yaml` - Horizontal Pod Autoscaler**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ .Values.app.name }}-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ .Values.app.name }}
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }}
```

**Explanation**: Autoscaling configuration based on CPU usage.

### **3. `service.yaml` - Kubernetes Service**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: disbursement-service
spec:
  ports:
    - port: 80
      targetPort: http
```

**Explanation**: Defines a `ClusterIP` service that routes traffic to the deployed containers.

---

## **How Everything is Connected and Executed**

1. **GitLab CI Pipeline**:

   * **Stage 1**: Lints and tests the Go code (`validate_sub.sh`).
   * **Stage 2**: Builds the Docker image using Kaniko (`build_subs.sh`).
   * **Stage 3**: Deploys the image to GKE using Helm (`deploy_sub.sh`).

2. **Helm Templates**:

   * These files define the Kubernetes resources (`Deployment`, `HPA`, `Service`, `ServiceAccount`).
   * The `deployment.yaml`, `hpa.yaml`, `service.yaml`, and `serviceaccount.yaml` templates are applied when `helm upgrade` is run during the deploy stage.

3. **Environment Variables**:

   * Variables like `GCP_SA_KEY_REGISTRY` and `CI_COMMIT_SHORT_SHA` are configured in GitLab CI's environment variables section.

---

## **Service Account Configuration**

To create the service account for deploying to GKE:

1. Go to the **Google Cloud Console**.
2. Navigate to **IAM & Admin** → **Service Accounts**.
3. Create a new service account (`gke-app`).
4. Assign the necessary roles (e.g., Kubernetes Engine Admin, Storage Admin, etc.).
5. Download the **JSON key** and set it as an environment variable in the CI/CD pipeline.
