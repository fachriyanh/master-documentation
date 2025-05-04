# Security Scans in CI/CD Pipelines

In this section, we cover how to integrate **security scans** into your GitLab CI/CD pipelines using built-in tools like **SAST** (Static Application Security Testing), **DAST** (Dynamic Application Security Testing), **Dependency Scanning**, and **Container Scanning**. We also walk through setting up **auto vulnerability scans** to identify and address security issues in your codebase.

## 1. **GitLab Built-in Security Tools**

GitLab provides a set of integrated tools to help secure your application at various stages of the development lifecycle. These tools help detect vulnerabilities in the codebase, dependencies, and running containers.

### 1.1 **Static Application Security Testing (SAST)**

SAST scans your source code, identifying vulnerabilities like SQL injection, XSS, and insecure deserialization, even before your application is running.

#### How to Enable SAST in GitLab CI

To enable **SAST** in your pipeline, add the following to your `.gitlab-ci.yml` file:

```yaml
stages:
  - test
  - deploy

sast:
  stage: test
  script:
    - echo "Running Static Application Security Testing (SAST)"
  include:
    - template: Security/SAST.gitlab-ci.yml
```

GitLab will automatically run SAST during the `test` stage and generate a report of vulnerabilities detected in your code.

### 1.2 **Dynamic Application Security Testing (DAST)**

DAST scans your running application, testing for vulnerabilities such as cross-site scripting (XSS), SQL injection, and other runtime issues. This is useful for testing applications after they’ve been deployed to an environment.

#### How to Enable DAST in GitLab CI

To enable **DAST**, you need to configure your `.gitlab-ci.yml` file as follows:

```yaml
stages:
  - test
  - deploy

dast:
  stage: test
  script:
    - echo "Running Dynamic Application Security Testing (DAST)"
  include:
    - template: Security/DAST.gitlab-ci.yml
```

This will trigger **DAST** during the `test` stage on the deployed application.

### 1.3 **Dependency Scanning**

Dependency Scanning identifies vulnerabilities in third-party libraries and dependencies used in your project. It checks for known vulnerabilities in libraries or packages that are part of your project’s build.

#### How to Enable Dependency Scanning in GitLab CI

To enable **Dependency Scanning**, add the following configuration:

```yaml
stages:
  - test
  - deploy

dependency_scanning:
  stage: test
  script:
    - echo "Running Dependency Scanning"
  include:
    - template: Security/Dependency-Scanning.gitlab-ci.yml
```

GitLab will automatically scan your project’s dependencies for vulnerabilities during the `test` stage.

### 1.4 **Container Scanning**

Container Scanning checks for vulnerabilities in your Docker containers, which might be missed by other security tools. This scan checks the container image for issues such as outdated software, missing patches, or misconfigurations.

#### How to Enable Container Scanning in GitLab CI

To enable **Container Scanning**, add this configuration to your `.gitlab-ci.yml`:

```yaml
stages:
  - test
  - deploy

container_scanning:
  stage: test
  script:
    - echo "Running Container Scanning"
  include:
    - template: Security/Container-Scanning.gitlab-ci.yml
```

This will automatically scan your container images for security issues during the `test` stage.

---

## 2. **Setting Up Auto Vulnerability Scans**

Auto vulnerability scans are essential for continuously monitoring your codebase, dependencies, and containers for security issues. GitLab provides built-in templates and features that automate this process, ensuring vulnerabilities are detected early and often.

### 2.1 **Auto Vulnerability Scans for Code, Dependencies, and Containers**

To set up automated vulnerability scans, you can use the built-in GitLab CI/CD templates for SAST, DAST, Dependency Scanning, and Container Scanning. These templates are designed to be easily included in your `.gitlab-ci.yml` file.

Example setup for automatic vulnerability scans:

```yaml
stages:
  - test
  - deploy

# Enable Static Application Security Testing (SAST)
sast:
  stage: test
  script:
    - echo "Running Static Application Security Testing (SAST)"
  include:
    - template: Security/SAST.gitlab-ci.yml

# Enable Dependency Scanning
dependency_scanning:
  stage: test
  script:
    - echo "Running Dependency Scanning"
  include:
    - template: Security/Dependency-Scanning.gitlab-ci.yml

# Enable Container Scanning
container_scanning:
  stage: test
  script:
    - echo "Running Container Scanning"
  include:
    - template: Security/Container-Scanning.gitlab-ci.yml

# Enable Dynamic Application Security Testing (DAST)
dast:
  stage: test
  script:
    - echo "Running Dynamic Application Security Testing (DAST)"
  include:
    - template: Security/DAST.gitlab-ci.yml
```

### 2.2 **Auto Vulnerability Scan Reports**

GitLab generates detailed reports from these scans, which can be viewed in the GitLab UI. Each report will list the vulnerabilities found, categorized by severity (low, medium, high), and provide recommendations for fixing the issues.

- **SAST report**: Displays vulnerabilities in your source code.
- **DAST report**: Displays vulnerabilities in the live application.
- **Dependency Scanning report**: Shows vulnerabilities in third-party libraries.
- **Container Scanning report**: Shows vulnerabilities in container images.

You can configure GitLab to send notifications for these vulnerabilities, allowing developers to fix them promptly.

### 2.3 **Auto-Cancel Stale Pipelines for Vulnerability Scans**

To improve efficiency, you can set up GitLab to automatically cancel stale pipelines (e.g., if a newer commit is pushed before the current pipeline finishes). This ensures that resources are not wasted on outdated or irrelevant scans.

```yaml
stages:
  - test

auto_cancel_pipelines:
  script:
    - gitlab-ci-cancel-stale-pipelines
```

---

## Summary of Key Concepts

| **Concept**                     | **Description**                                                                 |
|----------------------------------|---------------------------------------------------------------------------------|
| **SAST**                         | Scans the source code for vulnerabilities before runtime.                        |
| **DAST**                         | Scans the running application for vulnerabilities during runtime.                |
| **Dependency Scanning**          | Scans dependencies for known vulnerabilities.                                   |
| **Container Scanning**           | Scans Docker container images for security issues.                              |
| **Auto Vulnerability Scans**     | Set up automated scans in your CI pipeline to detect security issues continuously. |
| **Scan Reports**                 | GitLab generates detailed security reports from each scan (SAST, DAST, Dependency Scanning, Container Scanning). |

---

## Example GitLab CI/CD Configuration for Security Scans

```yaml
stages:
  - test
  - deploy

# SAST
sast:
  stage: test
  script:
    - echo "Running Static Application Security Testing (SAST)"
  include:
    - template: Security/SAST.gitlab-ci.yml

# Dependency Scanning
dependency_scanning:
  stage: test
  script:
    - echo "Running Dependency Scanning"
  include:
    - template: Security/Dependency-Scanning.gitlab-ci.yml

# Container Scanning
container_scanning:
  stage: test
  script:
    - echo "Running Container Scanning"
  include:
    - template: Security/Container-Scanning.gitlab-ci.yml

# DAST
dast:
  stage: test
  script:
    - echo "Running Dynamic Application Security Testing (DAST)"
  include:
    - template: Security/DAST.gitlab-ci.yml
```

---

This configuration enables **automatic vulnerability scans** using GitLab's security templates. Vulnerabilities will be automatically detected and reported, helping you maintain a secure application.
