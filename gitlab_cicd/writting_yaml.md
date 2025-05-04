
# 📚 Complete Guide to Writing YAML

## 1. Good Practices in Writing YAML

### 1.1 Indentation
- **Always use spaces** instead of tabs. This is **mandatory** in YAML.
- Use **2 spaces** per indentation level (this is the convention).
  
**Correct Example:**
```yaml
stages:
  - build
  - test
  - deploy
```
- Avoid mixing tabs and spaces as it will lead to parsing errors.

### 1.2 Anchors and Aliases
- **Anchors (`&`)** are used to define a piece of YAML data.
- **Aliases (`*`)** allow you to reuse that data in other places, reducing duplication.

**Example:**
```yaml
default_settings: &defaults
  timeout: 30
  retries: 3

job1:
  <<: *defaults
  script:
    - echo "Job1"

job2:
  <<: *defaults
  script:
    - echo "Job2"
```

In this example:
- `&defaults` defines an anchor.
- `<<: *defaults` uses an alias to import those settings.

### 1.3 Using Scalars and Strings
- **Strings** in YAML don’t need quotes unless they include special characters (like `:`, `-`, or `#`).
  
**Example:**
```yaml
name: John Doe
description: "This is a sample description."
```

**Escaping special characters**:
- If you need to include special characters in a string, **enclose it in double quotes**.

### 1.4 Avoiding Tabulation
- **Never use tabs** for indentation. Always use **spaces**.
- A good practice is to set your editor to automatically convert tabs to spaces.

## 2. Modularizing YAML for Easier Maintenance

### 2.1 Why Modularize YAML?
- Modular YAML helps in **reuse** and **maintenance**, especially when working on complex configurations or large projects.
- It reduces redundancy and allows you to update configurations more easily without needing to modify multiple places.

### 2.2 Techniques for Modular YAML

#### 2.2.1 Using Anchors and Aliases
As discussed earlier, **anchors** (`&`) and **aliases** (`*`) are a powerful way to reuse and modularize YAML configurations.

For example:
```yaml
common_config: &common
  cpu: 2
  memory: 4GB

job1:
  <<: *common
  name: "Job 1"
  script:
    - echo "Running Job 1"

job2:
  <<: *common
  name: "Job 2"
  script:
    - echo "Running Job 2"
```
Here, the `common_config` is used in both `job1` and `job2`, making the YAML easier to maintain.

#### 2.2.2 Using External YAML Files (for Large Projects)
- **Include external YAML files** using the `!include` directive. This allows you to break up the YAML into smaller, more manageable files.

Example:
```yaml
# main.yml
include:
  - job_config.yml
  - settings.yml
```

You can include complex configurations across multiple files and avoid clutter in one large YAML file.

#### 2.2.3 Define Variables (Reusable Settings)
Another approach is to define reusable variables or configurations at the top of your YAML file and reference them throughout.

Example:
```yaml
defaults:
  retries: 3
  timeout: 60

job1:
  retries: *defaults.retries
  timeout: *defaults.timeout

job2:
  retries: *defaults.retries
  timeout: *defaults.timeout
```
This way, changing a configuration only requires changing it in the top section, making it easy to maintain.

#### 2.2.4 Using a YAML Template (for dynamic configurations)
For more complex workflows, you can use YAML templates to inject values dynamically (like environment-specific values).

Example:
```yaml
environments:
  production:
    db_host: prod-db.example.com
    db_port: 5432
  development:
    db_host: dev-db.example.com
    db_port: 5432

config:
  <<: *environments.production
```

This allows you to create different configurations for different environments and easily switch between them.

## 3. Pro Tips for YAML Writing

- **Keep it readable**: Use consistent indentation and line breaks.
- **Avoid excessive nesting**: Keep the hierarchy flat to avoid confusion.
- **Add comments**: Use comments to explain sections of the YAML.
  
**Example:**
```yaml
# This job runs on the main branch
deploy:
  stage: deploy
  script:
    - deploy_app
```

- **Validate your YAML**: Use online tools like [YAML Lint](http://www.yamllint.com/) to validate your YAML syntax.

## 📋 Pro Tips to Master Modular YAML

- **Minimize Redundancy**: Always look for opportunities to use anchors or external files to avoid repeating configuration.
- **Consistent Formatting**: Follow a consistent pattern for indentation, naming, and structure.
- **Template for Reuse**: Always look for common configurations across jobs or stages and extract them.

