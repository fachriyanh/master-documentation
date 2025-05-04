# Advanced Topics in CI/CD Pipelines

In this section, we dive into **advanced CI/CD pipeline configurations** that help optimize and scale your workflows. These concepts allow you to split large pipelines into smaller, more manageable parts, dynamically generate child pipelines, and run builds for multiple combinations of environments (e.g., OS and versioning).

## 1. **Parent-Child Pipelines**

Parent-Child pipelines allow you to split a large pipeline into smaller, more manageable pipelines. This approach makes it easier to maintain and debug complex workflows by breaking them into discrete parts.

### 1.1 **What Are Parent-Child Pipelines?**

- **Parent pipelines** trigger one or more **child pipelines** based on the defined job and conditions. The child pipelines can run independently and asynchronously.
- This structure allows for better parallelization and isolation of different tasks.

### 1.2 **How to Configure Parent-Child Pipelines**

To create parent-child pipelines in GitLab, use the `trigger` keyword to invoke child pipelines within a parent pipeline.

```yaml
# Parent Pipeline Configuration
stages:
  - build
  - test
  - deploy

build:
  script:
    - make build

# Trigger Child Pipeline
trigger_child_pipeline:
  stage: deploy
  trigger:
    include: child-pipeline.yml   # Path to the child pipeline configuration file
    strategy: depend              # Parent pipeline waits for the child to complete

```

#### Child Pipeline (`child-pipeline.yml`)

```yaml
stages:
  - test
  - deploy

test:
  script:
    - make test

deploy:
  script:
    - make deploy
```

### Key Points:
- **Parent Pipeline**: The main pipeline that contains stages like build, test, and deploy.
- **Child Pipeline**: A separate pipeline defined in another YAML file, triggered by the parent pipeline.
- **`trigger` Keyword**: The `trigger` keyword in GitLab CI is used to invoke the child pipeline.
- **`strategy: depend`**: Ensures that the parent pipeline waits for the child pipeline to finish before proceeding.

## 2. **Dynamic Child Pipelines**

Dynamic Child Pipelines allow you to generate child pipelines dynamically during runtime based on conditions such as the branch, tag, or specific environment. This is especially useful when you need to adapt the pipeline to different conditions without hardcoding multiple pipeline definitions.

### 2.1 **What Are Dynamic Child Pipelines?**

Dynamic pipelines can be generated on the fly using conditions in the parent pipeline. This allows you to trigger different sets of jobs or pipelines depending on runtime data, like the branch or commit message.

### 2.2 **How to Configure Dynamic Child Pipelines**

You can define a dynamic child pipeline within a parent pipeline using conditional logic in the YAML configuration.

```yaml
stages:
  - build
  - test
  - deploy

build:
  script:
    - make build

deploy:
  script:
    - make deploy

dynamic_child_pipeline:
  script:
    - |
      if [ "$CI_COMMIT_REF_NAME" == "master" ]; then
        echo "Running master-specific child pipeline"
        echo "include: master-pipeline.yml" > child-pipeline.yml
      elif [ "$CI_COMMIT_REF_NAME" == "dev" ]; then
        echo "Running dev-specific child pipeline"
        echo "include: dev-pipeline.yml" > child-pipeline.yml
      fi
  trigger:
    include: child-pipeline.yml
    strategy: depend
```

#### Key Points:
- The `dynamic_child_pipeline` job generates a child pipeline based on the branch name or any other condition you define.
- The child pipeline definition is written to a temporary file (`child-pipeline.yml`), which is then used by the `trigger` to execute the appropriate pipeline.

## 3. **Matrix Builds**

Matrix builds allow you to run a set of jobs for multiple combinations of environments, such as different operating systems, language versions, or configurations. This approach allows you to efficiently run tests for a broad range of configurations in parallel, speeding up the testing process.

### 3.1 **What Are Matrix Builds?**

Matrix builds enable the simultaneous execution of jobs for different combinations of parameters like OS, language versions, and other environment variables. For instance, you can run tests for multiple versions of Python or Node.js at the same time.

### 3.2 **How to Configure Matrix Builds**

To configure matrix builds in GitLab CI/CD, use the `matrix` feature in combination with the `matrix` keyword inside your `.gitlab-ci.yml` file. This allows you to define all the different combinations of parameters for which the jobs should run.

#### Example: Matrix Build for Different Node.js Versions

```yaml
stages:
  - test

test:
  stage: test
  matrix:
    - NODE_VERSION: ["12", "14", "16"]
    - OS: ["ubuntu", "alpine"]

  script:
    - |
      echo "Testing Node.js $NODE_VERSION on $OS"
      docker run --rm node:$NODE_VERSION-$OS npm test
```

### Key Points:
- **Matrix**: The `matrix` keyword is used to define multiple variables (`NODE_VERSION`, `OS`) and their possible values.
- Each combination of `NODE_VERSION` and `OS` will create a separate job that runs in parallel.
- **Efficiency**: This reduces the time taken for testing across different environments by leveraging parallelism.

### 3.3 **Advanced Matrix Builds with Multiple Variables**

Matrix builds can include more than just one or two variables. For more complex testing scenarios, you can define a multi-variable matrix to run a wide range of combinations.

```yaml
stages:
  - test

test:
  stage: test
  matrix:
    - NODE_VERSION: ["12", "14"]
      OS: ["ubuntu", "alpine"]
      DATABASE: ["mysql", "postgres"]
      
  script:
    - |
      echo "Testing Node.js $NODE_VERSION, OS $OS, and Database $DATABASE"
      docker run --rm node:$NODE_VERSION-$OS npm test --db=$DATABASE
```

### Key Points:
- The matrix configuration enables testing across multiple OSes, Node.js versions, and databases in parallel.
- This approach can drastically reduce pipeline durations, especially when working with multiple combinations of software stacks.

---

## Summary of Key Concepts

| **Concept**                    | **Description**                                                                 |
|---------------------------------|---------------------------------------------------------------------------------|
| **Parent-Child Pipelines**      | Split large pipelines into smaller ones for better maintainability and isolation. |
| **Dynamic Child Pipelines**     | Generate child pipelines dynamically based on runtime conditions.                |
| **Matrix Builds**               | Run multiple combinations of builds in parallel for different environments, speeding up the process. |

---

## Example GitLab CI/CD Configuration for Advanced Topics

```yaml
stages:
  - build
  - test
  - deploy

# Parent Pipeline
build:
  script:
    - make build

# Dynamic Child Pipeline
dynamic_child_pipeline:
  script:
    - |
      if [ "$CI_COMMIT_REF_NAME" == "master" ]; then
        echo "include: master-pipeline.yml" > child-pipeline.yml
      else
        echo "include: dev-pipeline.yml" > child-pipeline.yml
      fi
  trigger:
    include: child-pipeline.yml
    strategy: depend

# Matrix Build
matrix_test:
  stage: test
  matrix:
    - NODE_VERSION: ["12", "14", "16"]
    - OS: ["ubuntu", "alpine"]

  script:
    - |
      echo "Running tests on Node.js $NODE_VERSION and OS $OS"
      docker run --rm node:$NODE_VERSION-$OS npm test
```

### Key Takeaways:
- **Parent-Child Pipelines**: Break down your large pipeline into smaller, independent pipelines for better maintainability.
- **Dynamic Child Pipelines**: Generate pipelines dynamically based on conditions to handle different scenarios on the fly.
- **Matrix Builds**: Test across multiple configurations and environments simultaneously to speed up your testing process.