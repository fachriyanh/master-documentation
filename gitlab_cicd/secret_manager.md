# Secrets Management

Managing secrets like API keys, passwords, and tokens is critical for keeping applications secure. GitLab CI/CD offers a simple way to store secrets, but you can also integrate advanced external secret management systems like Vault, AWS Secrets Manager, and Google Secret Manager for better security.

## Using GitLab CI/CD Variables for Storing API Keys, Passwords, Tokens

### Storing Secrets in GitLab CI/CD Variables
GitLab CI/CD provides a secure way to store sensitive data such as API keys, passwords, and tokens. These variables are encrypted and hidden from logs during pipeline execution, ensuring that your secrets are not exposed.

### Steps to Store Secrets:

1. **Navigate to GitLab Settings**:
   - Go to your GitLab project or group.
   - Under **Settings**, click **CI / CD**.
   - Expand the **Variables** section.

2. **Add a New Variable**:
   - Click **Add Variable** to create a new secret.
   - Set the **Key** (e.g., `API_KEY`, `DB_PASSWORD`) and **Value** (the secret).
   - You can set the variable as **protected** (only available to protected branches) or **masked** (hidden in logs).
   - Optionally, set an **Environment Scope** to limit availability to specific environments.

### Example:

In your pipeline file, you can reference the stored secret like this:

```yaml
stages:
  - build
  - deploy

build:
  script:
    - echo $API_KEY  # Access the API_KEY variable securely
```

GitLab CI/CD variables help you securely store and access sensitive information without exposing it in the code.

---

## External Secret Managers (For Advanced Use)

For more advanced secret management, you can integrate with external secret managers like **Vault**, **AWS Secrets Manager**, and **Google Secret Manager**. These services provide enhanced security, such as better access control, auditing, and management of secrets.

### 1. **HashiCorp Vault**:
Vault is an open-source tool designed to securely store and manage secrets, such as API keys and passwords.

#### Integrating Vault with GitLab CI/CD:
1. Set up a Vault server to store your secrets.
2. Use Vault’s API to retrieve secrets during pipeline execution.

Example using Vault in GitLab CI/CD:

```yaml
stages:
  - deploy

deploy:
  script:
    - export SECRET=$(curl --header "X-Vault-Token: $VAULT_TOKEN" https://vault.example.com/v1/secret/data/myapp | jq -r '.data.data.mysecret')
    - echo $SECRET  # Use the secret in your deployment script
```

### 2. **AWS Secrets Manager**:
AWS Secrets Manager allows you to securely store and manage secrets like database credentials and API keys.

#### Integrating AWS Secrets Manager with GitLab CI/CD:
1. Store secrets in AWS Secrets Manager.
2. Set up AWS credentials in GitLab for authentication.
3. Use the AWS CLI to retrieve secrets during pipeline execution.

Example using AWS Secrets Manager in GitLab CI/CD:

```yaml
stages:
  - deploy

deploy:
  script:
    - SECRET=$(aws secretsmanager get-secret-value --secret-id mysecret --query SecretString --output text)
    - echo $SECRET  # Use the retrieved secret in your deployment script
```

### 3. **Google Secret Manager**:
Google Secret Manager helps store and manage secrets securely on Google Cloud.

#### Integrating Google Secret Manager with GitLab CI/CD:
1. Store secrets in Google Secret Manager.
2. Set up authentication with Google Cloud credentials.
3. Use the `gcloud` CLI to retrieve secrets during pipeline execution.

Example using Google Secret Manager in GitLab CI/CD:

```yaml
stages:
  - deploy

deploy:
  script:
    - SECRET=$(gcloud secrets versions access latest --secret="mysecret")
    - echo $SECRET  # Use the retrieved secret in your deployment script
```

---

## Summary

- **GitLab CI/CD Variables** are a simple and secure way to store API keys, passwords, and tokens directly in GitLab settings.
- **External Secret Managers** such as Vault, AWS Secrets Manager, and Google Secret Manager provide advanced features for managing secrets, like access control, auditing, and secret rotation.

---

### Save as a `.md` file:
To create a `.md` file, copy the content above into a text editor and save it as `secrets-management.md`.