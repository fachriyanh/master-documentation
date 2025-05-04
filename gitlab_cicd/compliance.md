# Compliance in CI/CD Pipelines

This section discusses how to use **GitLab CI/CD** features to ensure compliance with internal and external audit requirements. We will cover important GitLab features that help you implement security controls, such as **approval pipelines**, **protected branches**, and **signed commits**, to meet compliance and regulatory standards.

## 1. **GitLab CI/CD for Audit and Compliance**

GitLab offers several features that help organizations maintain compliance with security policies, standards, and regulations (such as SOC 2, GDPR, and ISO 27001). These features provide controls over code changes, deployments, and workflows.

### 1.1 **Approval Pipelines**

Approval pipelines ensure that key steps of your pipeline are manually approved by a designated person or team before proceeding. This can be useful for enforcing additional checks and balances in your deployment process.

#### How to Enable Approval Pipelines

To create an approval pipeline in GitLab, you can use the `manual` job and the `when: manual` condition. This will pause the pipeline at a specific stage, requiring manual intervention to continue the process.

Example `.gitlab-ci.yml` configuration:

```yaml
stages:
  - test
  - approval
  - deploy

approve_deployment:
  stage: approval
  script:
    - echo "Approval required before deployment."
  when: manual
  allow_failure: false
```

In this example, the `approve_deployment` job will stop the pipeline and wait for a manual approval to proceed. A user with the appropriate permissions must approve it through the GitLab UI.

You can also enforce approval by specifying a **code review** or **security check** for additional compliance requirements.

### 1.2 **Protected Branches**

Protected branches allow you to control who can push to or merge into critical branches like `main`, `master`, or `release`. These branches are often subject to stricter compliance requirements and need to be carefully managed.

#### How to Set Up Protected Branches

1. Go to your **GitLab project** and navigate to **Settings > Repository > Protected Branches**.
2. Select the branch you want to protect (e.g., `main`).
3. Define the level of protection:
   - **Who can push**: Set this to **Maintainers** or a specific role.
   - **Who can merge**: Limit merges to certain roles, such as **Maintainers** or specific users.
   - **Require approvals**: You can enforce merge request approvals before allowing merges.

You can also configure automatic merge request approvals for added compliance. This ensures that only authorized personnel can alter critical branches and all changes are reviewed.

### 1.3 **Signed Commits**

Signed commits ensure that the person making the commit is authenticated using their GPG key. This adds an additional layer of security by verifying the identity of contributors, which is crucial for compliance in many regulated industries.

#### How to Enable Signed Commits

1. **Generate GPG keys**: Developers must generate GPG keys for signing their commits. The following commands can be used to generate GPG keys:

   ```bash
   gpg --full-generate-key
   gpg --list-secret-keys --keyid-format LONG
   ```

2. **Add the GPG key to GitLab**:
   - Go to **GitLab > User Settings > GPG Keys**.
   - Add your GPG public key here.

3. **Enable signed commits**:
   Developers should configure Git to automatically sign their commits with the following command:

   ```bash
   git config --global commit.gpgSign true
   ```

4. **Verify signed commits**:
   After the commit is made, GitLab will automatically verify and display whether the commit is signed or not in the repository view.

GitLab also supports signing commits in **merge requests**. If you require signed commits as part of your compliance workflow, this feature ensures that all commits are properly verified and tracked.

---

## 2. **GitLab Compliance Features**

### 2.1 **Audit Logs**

GitLab provides **Audit Logs** to track and record all user activity within the platform. This is essential for maintaining compliance and auditing purposes.

You can access **Audit Logs** from the **Admin Area > Monitoring > Audit Events**. These logs record actions such as:

- Changes to repository settings.
- Pipeline executions.
- User logins.
- Merge actions.

Audit logs can be exported for external compliance reviews and are critical for providing traceability and accountability in your CI/CD pipeline.

### 2.2 **Compliance Pipelines and Security Features**

GitLab supports compliance features in pipelines that can ensure you meet regulatory requirements. Here are some options:

- **Static Analysis**: Automatically run security scans such as **SAST** and **DAST** for code and deployed applications.
- **Dependency Scanning**: Automatically scan dependencies for vulnerabilities.
- **Container Scanning**: Scan Docker images for security issues.
- **Manual Approval**: Ensure no deployment happens without manual approval, adding an extra layer of security for critical environments.

---

## 3. **Summary of Compliance Features**

| **Feature**                    | **Description**                                                                 |
|---------------------------------|---------------------------------------------------------------------------------|
| **Approval Pipelines**          | Manual approval steps in the pipeline to enforce control over deployments.      |
| **Protected Branches**          | Restrict who can push and merge to critical branches like `main` and `release`. |
| **Signed Commits**              | Ensures all commits are signed and verified to authenticate the author.         |
| **Audit Logs**                  | Tracks all user actions within GitLab for compliance and auditing purposes.     |
| **Compliance Pipelines**        | Use security scans, manual approvals, and protected branches to meet compliance standards. |

---

## Example GitLab CI/CD Configuration for Compliance

```yaml
stages:
  - test
  - approval
  - deploy

# Approval pipeline for deployment
approve_deployment:
  stage: approval
  script:
    - echo "Approval required before deployment."
  when: manual
  allow_failure: false

# Security and compliance checks (SAST, Dependency Scanning, etc.)
security_check:
  stage: test
  script:
    - echo "Running security checks for compliance"
  include:
    - template: Security/SAST.gitlab-ci.yml
    - template: Security/Dependency-Scanning.gitlab-ci.yml
```

---

This configuration helps ensure that your GitLab CI/CD pipeline is aligned with compliance requirements, such as **manual approval workflows**, **protected branches**, and **signed commits**. You can further extend these configurations based on your specific compliance needs.