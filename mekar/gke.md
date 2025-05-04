# **Preparing the GKE Environment for Deployment**

This section outlines the steps required to set up **Google Kubernetes Engine (GKE)** for deploying your application using the CI/CD pipeline.

### **1. Set Up Google Cloud Project**

Before you create the GKE cluster, make sure you have a Google Cloud project set up:

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project or use an existing one (e.g., `mekar-dev`).
3. Set your project by running the following command (replace `mekar-dev` with your project name):

```bash
gcloud config set project mekar-dev
```

---

### **2. Enable Required Google Cloud APIs**

For GKE and other Google Cloud services to work, you need to enable the required APIs. Run the following commands to enable:

```bash
gcloud services enable container.googleapis.com
gcloud services enable cloudresourcemanager.googleapis.com
gcloud services enable iam.googleapis.com
gcloud services enable cloudkms.googleapis.com
```

---

### **3. Create a Service Account for GKE Deployment**

The service account will be used by the CI/CD pipeline to authenticate with Google Cloud and deploy to GKE.

1. Go to the **Google Cloud Console** → **IAM & Admin** → **Service Accounts**.

2. Click **Create Service Account**.

3. Enter a name like `gke-app` and click **Create**.

4. Assign the following roles:

   * **Kubernetes Engine Admin**: Grants access to manage Kubernetes clusters.
   * **Storage Admin**: Allows pushing Docker images to Google Container Registry (GCR).
   * **Viewer**: Optional, for read-only access to other resources.

5. Click **Done** after assigning the roles.

6. After creating the service account, generate a **JSON key**:

   * In the Service Account details page, click **Create Key**.
   * Choose **JSON** and download the key file.

7. Store this JSON key in a secure location and use it in the CI/CD pipeline as an environment variable.

---

### **4. Set Up Google Container Registry (GCR)**

Google Container Registry (GCR) is where you'll store Docker images. To ensure your images are pushed to GCR:

1. Authenticate your local machine with Google Cloud:

```bash
gcloud auth configure-docker
```

2. Set up **image repositories**:

   * For example, you can use the project-specific path:

     * `asia-southeast2-docker.pkg.dev/mekar-dev/mekar-dev-container/subscriber-disbursement`

---

### **5. Create the GKE Cluster**

Create a GKE cluster that will run the **subscriber-disbursement** service.

1. Create the cluster using the following `gcloud` command (replace `mekar-reachitecture-dev` with your desired cluster name):

```bash
gcloud container clusters create mekar-reachitecture-dev \
  --region asia-southeast2 \
  --num-nodes=3 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=5 \
  --zone asia-southeast2-a
```

**Explanation of parameters**:

* `--region`: Specify the region for your GKE cluster.
* `--num-nodes`: Defines the initial number of nodes in the cluster.
* `--enable-autoscaling`: Enables autoscaling for the cluster.
* `--min-nodes` & `--max-nodes`: Set minimum and maximum nodes for autoscaling.

2. Once the cluster is created, get the credentials so that you can interact with the cluster:

```bash
gcloud container clusters get-credentials mekar-reachitecture-dev --region asia-southeast2
```

3. Verify your connection to the cluster by running:

```bash
kubectl cluster-info
```

---

### **6. Create the Namespace in GKE**

Namespaces in Kubernetes help separate resources within the cluster. Since the **subscriber-disbursement** service will be deployed in the `staging` environment, you can create the namespace with:

```bash
kubectl create namespace staging
```

---

### **7. Install Helm (if not installed)**

Helm is a package manager for Kubernetes that simplifies deployment. To install Helm on your machine:

1. Download and install Helm:

```bash
curl -sSL https://get.helm.sh/helm-v3.17.3-linux-amd64.tar.gz | tar -xz
mv linux-amd64/helm /usr/local/bin/helm
```

2. Verify Helm installation:

```bash
helm version
```

---

### **8. Configure IAM for GKE to Allow CI/CD Access**

You need to allow your CI/CD pipeline (e.g., GitLab CI) to authenticate with the GKE cluster.

* Ensure that the **Service Account** (`gke-app`) has permissions to interact with GKE, GCR, and deploy the application.
* If your CI/CD system uses Google Cloud as a service, you can authenticate via the `gcloud` CLI using the JSON key created earlier.

---

### **9. Configure CI/CD Environment Variables**

Add the following environment variables to your CI/CD system (e.g., GitLab CI/CD) for authentication and configuration.

1. **GCP Service Account Key (base64 encoded)**:

   * Store the **JSON service account key** in an environment variable (e.g., `GCP_SA_KEY_REGISTRY`).
   * To encode it in base64 (which is how it should be stored in the CI/CD system):

   ```bash
   base64 /path/to/your-service-account-key.json
   ```

2. **Image Tag**: The image tag (`IMAGE_TAG`) should be dynamically generated by your CI/CD pipeline using the Git commit SHA (`$CI_COMMIT_SHORT_SHA`).

3. **Other Variables**: Ensure variables like the cluster name (`mekar-reachitecture-dev`), region (`asia-southeast2`), and namespace (`staging`) are available in the CI/CD pipeline.

---

### **10. Configure Kubernetes Secrets for GCP Credentials**

For the **subscriber-disbursement** service to authenticate with Google Cloud (via GKE or GCR), you need to create a Kubernetes secret from the service account key:

1. Create the secret in Kubernetes:

```bash
kubectl create secret generic gcloud-service-account-key \
  --from-file=gcloud-key.json=/path/to/your-service-account-key.json \
  --namespace=staging
```

2. Use this secret in your deployment YAML files by referencing it in the **service account**.

---

### **11. Set Up Continuous Integration with GitLab CI**

1. In your GitLab project, go to **Settings** → **CI / CD** → **Variables**.

2. Add the following environment variables:

   * `GCP_SA_KEY_REGISTRY`: The base64-encoded service account key.
   * `IMAGE_TAG`: The Docker image tag (e.g., commit SHA).
   * `IMAGE_LATEST`: Optional, a tag for the latest image.

3. Ensure the CI/CD pipeline configuration (`gitlab-ci.yaml`) correctly references these variables.

---

### **12. Verify Deployment**

Once the setup is complete and the CI/CD pipeline is running, you can verify the deployment:

1. Check the pods in your `staging` namespace:

```bash
kubectl get pods -n staging
```

2. Verify that the service is correctly exposed by checking the service:

```bash
kubectl get svc -n staging
```

3. Check the logs of the deployed pod to ensure everything is running correctly:

```bash
kubectl logs <pod-name> -n staging
```

---

### **Conclusion**

By following these steps, you will have:

* Set up a **Google Kubernetes Engine (GKE)** cluster.
* Configured **Google Cloud IAM** for your CI/CD pipeline.
* Created the necessary **Kubernetes resources** and connected them to your **CI/CD pipeline**.
* Deployed the **subscriber-disbursement** service to **GKE** using Helm.