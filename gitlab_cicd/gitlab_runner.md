# Runners (Shared vs Specific)

In this document, we will discuss the installation and configuration of GitLab Runner, the available executor options, and how to connect a runner to a project or group.

## 1. Installasi dan Konfigurasi GitLab Runner Sendiri

### 1.1 Install GitLab Runner

To install GitLab Runner on your system, follow these steps:

1. **Add the GitLab repository to your package manager**:

   For **Debian/Ubuntu**:
   ```bash
   sudo apt-get install -y curl
   curl -L --output /tmp/gitlab-runner.deb https://gitlab.com/gitlab-org/gitlab-runner/-/jobs/1060156305/artifacts/download
   sudo dpkg -i /tmp/gitlab-runner.deb
   ```

   For **CentOS/RHEL**:
   ```bash
   sudo curl -L --output /etc/yum.repos.d/gitlab-runner.repo https://gitlab.com/gitlab-org/gitlab-runner/-/jobs/1060156305/artifacts/download
   sudo yum install gitlab-runner
   ```

   #### Start and Enable GitLab Runner Service
   ```bash
   sudo systemctl enable gitlab-runner
   sudo systemctl start gitlab-runner
   ```

### 1.2 Configuring GitLab Runner

1. **Register the Runner**:
   To register the GitLab Runner with your GitLab instance, run the following command:
   ```bash
   sudo gitlab-runner register
   ```

2. **Enter the GitLab Instance URL**:
   Provide your GitLab instance URL (e.g., `https://gitlab.com/`).

3. **Enter the Registration Token**:
   The registration token can be found in your GitLab project or group settings under **CI / CD** > **Runners**.

4. **Provide a Description**:
   You will be prompted for a description of the runner. This is optional.

5. **Choose the Executor**:
   Below are the available executor options that you can choose from.


## 2. Executor Options

GitLab Runner supports several types of executors, which define where and how your jobs will run. Below are the available options:

### 2.1 Shell

- **Description**: The **Shell** executor runs jobs directly on the machine where GitLab Runner is installed.
- **Use Case**: Ideal for simple, local setups where you want to run commands directly on the system.
- **Configuration**: No additional configuration is needed. It works out of the box once the runner is registered.

### 2.2 Docker

- **Description**: The **Docker** executor runs jobs in Docker containers, which provides an isolated environment for each job.
- **Use Case**: Suitable for CI/CD pipelines that require different environments or containers.
- **Configuration**: Docker must be installed on the host machine. You can specify the Docker image to use for each job in the `.gitlab-ci.yml` file.

  Example `.gitlab-ci.yml`:
  ```yaml
  job_name:
    image: node:latest
    script:
      - npm install
  ```

### 2.3 Kubernetes
- **Description**: The **Kubernetes** executor uses Kubernetes clusters to run jobs in isolated pods.
- **Use Case**: Suitable for large-scale cloud environments where jobs need to scale dynamically.
- **Configuration**: Requires a configured Kubernetes cluster and a GitLab Runner instance connected to it. The runner will run jobs in Kubernetes pods.

  Example command to register:
  ```bash
  sudo gitlab-runner register --executor kubernetes --url https://gitlab.com/ --registration-token <token>

### 2.3 Kubernetes

- **Description**: The **Kubernetes** executor uses Kubernetes clusters to run jobs in isolated pods.
- **Use Case**: Suitable for large-scale cloud environments where jobs need to scale dynamically.
- **Configuration**: Requires a configured Kubernetes cluster and a GitLab Runner instance connected to it. The runner will run jobs in Kubernetes pods.

  Example command to register:
  ```bash
  sudo gitlab-runner register --executor kubernetes --url https://gitlab.com/ --registration-token <token>
  ```

### 2.4 VirtualBox

- **Description**: The **VirtualBox** executor runs jobs inside VirtualBox virtual machines.
- **Use Case**: Useful for testing across different operating systems and providing isolated environments for each job.
- **Configuration**: VirtualBox must be installed on the host machine. You can specify the VM image and settings for the job.

### 2.5 Custom

- **Description**: The **Custom** executor allows you to define your own environment for running jobs.
- **Use Case**: Suitable for complex setups where other executors are not sufficient.
- **Configuration**: Requires manual configuration of the environment for each job, including scripting or creating custom Docker containers.

## 3. Connecting a Runner to a Project or Group

You can connect a GitLab Runner to a specific project or group. Here's how to do it:

### 3.1 Connecting the Runner to a Project

1. Go to your GitLab project.
2. Navigate to **Settings** > **CI / CD**.
3. Under the **Runners** section, click on **Expand**.
4. Copy the registration token provided.
5. On the machine where GitLab Runner is installed, run:
   ```bash
   sudo gitlab-runner register 
   ```
6. Choose the executor type and provide a description for the runner.

After registration, the runner will be linked to your project and will be able to execute jobs for that project.

### 3.2 Connecting the Runner to a Group

To register a runner for a GitLab group:

1. Navigate to your GitLab group.
2. Go to **Settings** > **CI / CD**.
3. Under the **Runners** section, click on **Expand**.
4. Copy the registration token for the group.
5. On the machine where GitLab Runner is installed, run:
   ```bash
   sudo gitlab-runner register
   ```

Paste the group token when prompted.