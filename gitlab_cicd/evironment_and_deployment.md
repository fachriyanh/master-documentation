# Environment & Deployment

## Setup Environments in GitLab

In GitLab CI/CD, you can define different environments such as **review apps**, **production**, **staging**, etc., to manage deployments to different stages of your application lifecycle.

### Steps to Set Up Environments:

1. **Defining Environments in `.gitlab-ci.yml`**:
   - In your GitLab CI configuration file, you can define environments for different deployment stages.
   - Example for staging and production:
     ```yaml
     deploy_staging:
       script:
         - echo "Deploying to Staging"
       environment:
         name: staging
         url: https://staging.yourapp.com
     
     deploy_production:
       script:
         - echo "Deploying to Production"
       environment:
         name: production
         url: https://yourapp.com
     ```

2. **Review Apps**:
   - GitLab allows you to automatically create environments for each pull request or merge request, called **Review Apps**.
   - Review apps are temporary environments created to preview changes before merging them.
   - Example:
     ```yaml
     review_app:
       script:
         - echo "Deploying review app"
       environment:
         name: review/$CI_COMMIT_REF_NAME
         url: https://$CI_COMMIT_REF_NAME.yourapp.com
       only:
         - merge_requests
     ```

---

## Auto Deploy to Server via SSH, Docker, Kubernetes, AWS, GCP

### SSH Deployment:

1. **Set up SSH keys**:
   - Create SSH keys and add the public key to the `~/.ssh/authorized_keys` file on your server.

2. **GitLab CI Configuration**:
   - Use `ssh` in the GitLab CI pipeline to deploy to your server.
   - Example:
     ```yaml
     deploy_via_ssh:
       script:
         - ssh user@your-server "cd /path/to/app && git pull && ./deploy.sh"
     ```

### Docker Deployment:

1. **Build Docker Image**:
   - In your `.gitlab-ci.yml`, define a job to build the Docker image.
   - Example:
     ```yaml
     build_docker_image:
       script:
         - docker build -t yourimage:latest .
         - docker push yourimage:latest
     ```

2. **Deploy Using Docker**:
   - Deploy the built Docker image to your server or cloud provider.
   - Example:
     ```yaml
     deploy_docker:
       script:
         - docker pull yourimage:latest
         - docker run -d yourimage:latest
     ```

### Kubernetes Deployment:

1. **Configure Kubernetes Cluster**:
   - Set up a Kubernetes cluster using your preferred cloud provider (AWS, GCP, etc.).
   - Add the kubeconfig to GitLab CI.

2. **Deploy to Kubernetes**:
   - In the CI pipeline, use `kubectl` to deploy your app.
   - Example:
     ```yaml
     deploy_k8s:
       script:
         - kubectl apply -f k8s/deployment.yaml
     ```

### AWS Deployment:

1. **Set up AWS credentials**:
   - Use AWS CLI or GitLab's AWS integration to set up credentials securely.

2. **Deploy Using AWS CLI**:
   - Example deployment using AWS CLI:
     ```yaml
     deploy_aws:
       script:
         - aws ecs update-service --cluster your-cluster --service your-service --force-new-deployment
     ```

### GCP Deployment:

1. **Set up GCP credentials**:
   - Add your GCP credentials to GitLab as environment variables for secure access.

2. **Deploy Using GCP CLI**:
   - Example deployment using GCP:
     ```yaml
     deploy_gcp:
       script:
         - gcloud app deploy app.yaml
     ```

---

## Dynamic Environments (Auto-Create Environments per Branch)

GitLab allows you to automatically create an environment for each branch, enabling you to preview changes dynamically in different environments.

### Steps to Create Dynamic Environments:

1. **Configure Dynamic Environments in `.gitlab-ci.yml`**:
   - Use the `CI_COMMIT_REF_NAME` variable to dynamically name environments based on the branch name.
   - Example:
     ```yaml
     deploy_dynamic_env:
       script:
         - echo "Deploying to $CI_COMMIT_REF_NAME environment"
       environment:
         name: review/$CI_COMMIT_REF_NAME
         url: https://$CI_COMMIT_REF_NAME.yourapp.com
       only:
         - branches
     ```

2. **Automatic Creation per Branch**:
   - Every new branch will automatically create a new environment, allowing you to test your code in isolation.
   - Once the branch is merged, the environment can be deleted or archived depending on your configuration.

---

## Summary

- **Setup Environments**: Define environments for different stages such as staging, production, and review apps in GitLab CI.
- **Auto Deploy**: Automate deployment to servers via SSH, Docker, Kubernetes, AWS, or GCP based on your needs.
- **Dynamic Environments**: Create an environment automatically for each branch to test code in isolation before merging.

