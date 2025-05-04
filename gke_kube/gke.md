# Google Kubernetes Engine (GKE)

Google Kubernetes Engine (GKE) is a managed Kubernetes service by Google Cloud that allows you to deploy, manage, and scale containerized applications using Kubernetes. It abstracts the complexity of Kubernetes clusters, enabling easier management and automation of deployments.

In this section, we will cover how to create a GKE cluster, set up IAM & Permissions, configure `gcloud` CLI, and set up Google Container Registry (GCR) or Artifact Registry for storing Docker images.

## 1. **How to Create Cluster GKE dan Mange Config**

### 1.1 **Create Cluster GKE**

To create a GKE cluster, you can use the Google Cloud Console, `gcloud` CLI, or Terraform for infrastructure as code. Below is how to create a GKE cluster using the `gcloud` CLI.

1. **Set Project and Region:**

First, set your Google Cloud project and region:

```bash
gcloud config set project YOUR_PROJECT_ID
gcloud config set compute/region YOUR_REGION
```

2. **Create a GKE Cluster:**

You can create a cluster using the following `gcloud` command:

```bash
gcloud container clusters create YOUR_CLUSTER_NAME \
  --zone YOUR_ZONE \
  --num-nodes 3 \
  --enable-autoscaling \
  --min-nodes 1 \
  --max-nodes 5
```

This will create a cluster with the specified name and set the node pool to scale between 1 and 5 nodes.

3. **Get Credentials to Access the Cluster:**

Once the cluster is created, you need to authenticate and configure `kubectl` to manage the cluster:

```bash
gcloud container clusters get-credentials YOUR_CLUSTER_NAME --zone YOUR_ZONE --project YOUR_PROJECT_ID
```

Now, you can use `kubectl` to interact with your GKE cluster.

### 1.2 **Manage Config GKE Cluster**

GKE clusters can be configured for different purposes (e.g., node configurations, network configurations, autoscaling). For example, you can modify the cluster’s size, enable specific features (like private clusters, IP aliases), or change the machine type.

Here is an example to update the node pool settings of a GKE cluster:

```bash
gcloud container clusters resize YOUR_CLUSTER_NAME \
  --node-pool default-pool \
  --size 4 \
  --zone YOUR_ZONE
```

This will resize the cluster’s node pool to 4 nodes.

---

## 2. **IAM & Permissions di GKE**

In Google Cloud, Identity and Access Management (IAM) roles determine who can access what resources in your environment, including your GKE clusters.

### 2.1 **Membuat Role**

Google Cloud provides predefined roles for Kubernetes Engine access, but you can also create custom roles for more granular control.

1. **Create a Custom Role:**

```bash
gcloud iam roles create customKubernetesRole \
  --project YOUR_PROJECT_ID \
  --title "Custom Kubernetes Role" \
  --permissions "container.clusters.get,container.clusters.list" \
  --stage GA
```

This custom role allows users to list and get information about GKE clusters.

### 2.2 **Assign Roles**

After defining roles, you can assign them to users or service accounts:

1. **Assign a Role to a User:**

```bash
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
  --member "user:YOUR_USER_EMAIL" \
  --role "roles/container.clusterViewer"
```

This command assigns the **Cluster Viewer** role to a specific user.

2. **Assign a Role to a Service Account:**

```bash
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
  --member "serviceAccount:YOUR_SERVICE_ACCOUNT_EMAIL" \
  --role "roles/container.clusterAdmin"
```

This assigns the **Cluster Admin** role to a service account.

### 2.3 **Kontrol Akses dengan IAM Policies**

You can control access to your GKE clusters by defining IAM policies using roles and bindings. For example, restricting who can access the Kubernetes API server or modifying cluster configuration.

---

## 3. **Google Cloud SDK dan Setup `gcloud` CLI to interact with GKE**

The Google Cloud SDK (`gcloud`) is a command-line tool that allows you to manage your Google Cloud resources, including GKE. 

### 3.1 **Install Google Cloud SDK**

To interact with GKE clusters, you need to install the Google Cloud SDK. Follow these steps:

1. **Install the SDK:**

Follow the installation instructions based on your operating system from the [Google Cloud SDK installation page](https://cloud.google.com/sdk/docs/install).

2. **Initialize the SDK:**

Once installed, initialize the SDK by running:

```bash
gcloud init
```

This command will authenticate your account, set your project, and configure other settings.

3. **Authenticate with Google Cloud:**

If you need to authenticate, you can use the following command:

```bash
gcloud auth login
```

After authentication, your session will be valid to interact with GKE.

### 3.2 **Using `kubectl` with `gcloud`**

The `gcloud` CLI can be used to authenticate and get credentials for `kubectl`:

```bash
gcloud container clusters get-credentials YOUR_CLUSTER_NAME --zone YOUR_ZONE --project YOUR_PROJECT_ID
```

Once authenticated, you can use `kubectl` to manage your GKE cluster.

---

## 4. **Configure Google Container Registry (GCR) or Artifact Registry to store Docker Image**

Google Cloud offers two main options for storing Docker images: **Google Container Registry (GCR)** and **Artifact Registry**.

### 4.1 **Google Container Registry (GCR)**

GCR is a fully managed Docker container registry that allows you to store and manage Docker images in Google Cloud.

1. **Enable the GCR API:**

```bash
gcloud services enable containerregistry.googleapis.com
```

2. **Tag Your Docker Image:**

Tag the Docker image with the GCR repository URL:

```bash
docker tag your-image gcr.io/YOUR_PROJECT_ID/your-image
```

3. **Push Docker Image to GCR:**

Push the Docker image to the GCR registry:

```bash
docker push gcr.io/YOUR_PROJECT_ID/your-image
```

4. **Pull Docker Image from GCR:**

To pull the image from GCR:

```bash
docker pull gcr.io/YOUR_PROJECT_ID/your-image
```

### 4.2 **Artifact Registry**

Artifact Registry is the newer service for managing Docker images and other artifacts. It provides more flexibility than GCR, including support for multiple types of artifacts.

1. **Enable Artifact Registry API:**

```bash
gcloud services enable artifactregistry.googleapis.com
```

2. **Create an Artifact Registry Repository:**

```bash
gcloud artifacts repositories create YOUR_REPO_NAME \
  --repository-format=docker \
  --location=YOUR_REGION
```

3. **Authenticate Docker to Artifact Registry:**

Authenticate Docker to your Artifact Registry repository:

```bash
gcloud auth configure-docker YOUR_REGION-docker.pkg.dev
```

4. **Push and Pull Docker Images:**

Tag and push Docker images to Artifact Registry in a similar way to GCR:

```bash
docker tag your-image YOUR_REGION-docker.pkg.dev/YOUR_PROJECT_ID/YOUR_REPO_NAME/your-image
docker push YOUR_REGION-docker.pkg.dev/YOUR_PROJECT_ID/YOUR_REPO_NAME/your-image
```

---

## Summary

### Key Points:

- **Creating a GKE Cluster**: Use `gcloud` CLI to create a GKE cluster, configure node pools, and authenticate using `kubectl`.
- **IAM & Permissions**: Set up custom roles, assign them to users or service accounts, and control access to GKE resources.
- **Google Cloud SDK**: Install and configure the `gcloud` CLI to manage GKE and interact with your Kubernetes clusters.
- **Storing Docker Images**: Use Google Container Registry (GCR) or Artifact Registry to store Docker images for deployment in GKE.

---