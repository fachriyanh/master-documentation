# Job Artifacts & Caching

## How to Manage Artifacts

Artifacts are files or results generated during a job (e.g., build results, coverage reports, logs, etc.). These artifacts are usually stored so that they can be shared, accessed, or analyzed in future steps of the pipeline or later in the process.

### Steps to manage artifacts:

1. **Storing Artifacts**: 
   - You should define a location where artifacts will be stored, often linked to the build or job ID.
   - Artifacts should be named and organized logically to make them easy to identify and retrieve.
   
2. **Archiving Artifacts**:
   - Once the job is complete, archives should be created for the generated artifacts.
   - Ensure the artifact storage has a clean-up policy to avoid unnecessary accumulation of old files.
   
3. **Accessing Artifacts**: 
   - Artifacts can be accessed by different users or jobs depending on the configuration.
   - Ensure secure access to sensitive artifacts, such as deployment packages or private keys.

### Example Use Case:
If you are running a CI/CD pipeline, artifacts might include build output (e.g., `.tar` files, `.zip` files) or test coverage reports. These should be archived so that they can be viewed later or referenced by downstream jobs.

---

## Cache vs Artifacts: When to Use Each

Understanding the difference between caching and artifacts is key to optimizing your pipeline and saving time or resources.

### Caching:
- **Definition**: Caching involves storing dependencies or data that are reused across multiple jobs or runs. This is done to avoid unnecessary re-computation or downloads, thus speeding up the process.
- **Best Use Case**:
   - When you have dependencies that don’t change frequently, such as libraries or precompiled binaries.
   - When the task takes significant time (e.g., dependency installation) and should be reused.
   - Caching is ideal for workflows that can be reused across multiple jobs or runs without causing discrepancies.

### Example:
If you're building a software project that installs dependencies (e.g., npm, pip), caching these dependencies would allow future runs to skip the installation step, saving time.

### Artifacts:
- **Definition**: Artifacts are files created as a result of a job (such as a compiled app, logs, test reports). These are typically unique to that job and stored for later use or analysis.
- **Best Use Case**:
   - When the files are needed for reference later in the process or need to be preserved for later analysis or use.
   - Artifacts are typically not reused between runs like caches, but they provide crucial insights (e.g., logs, reports).

### Example:
After running unit tests, the test results or coverage reports are stored as artifacts, so they can be reviewed later.

---

## Summary

- **Artifacts** are unique outputs from a job, such as build results or test reports. They are stored for later use or review.
- **Cache** stores reusable data, like dependencies, to speed up future jobs by avoiding redundant work.

Use **artifacts** when you need the job's result for further steps or analysis, and use **cache** to avoid redundant steps that can be reused across multiple jobs.
