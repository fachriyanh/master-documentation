# Docker & CI/CD in GitLab

Building and managing Docker images is an essential part of deploying applications to Kubernetes through GitLab CI/CD pipelines. This section explains how to create a `Dockerfile`, build and push images to Google Container Registry (GCR), configure auto-tagging, and optimize image builds for faster pipelines.

## 1. **Creating a Dockerfile for the Application**

A `Dockerfile` defines how your application is built into a Docker image. For applications intended to run in GKE, ensure the Dockerfile produces a lightweight, production-ready image.

Example of a basic Dockerfile (suitable for a Golang application):

```
# Build Stage
FROM golang:1.22 as builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -o main .

# Run Stage
FROM gcr.io/distroless/base-debian11
WORKDIR /root/
COPY --from=builder /app/main .
CMD ["./main"]
```

This uses a **multi-stage build**:
- First stage builds the Go binary.
- Second stage uses a minimal base image for security and performance.

---

## 2. **Building and Pushing Docker Images in GitLab CI/CD Pipeline**

In GitLab CI, you can automate building the Docker image and pushing it to GCR.

Example `.gitlab-ci.yml` for building and pushing:

```
variables:
  PROJECT_ID: your-gcp-project-id
  IMAGE_NAME: gcr.io/$PROJECT_ID/your-app

stages:
  - build
  - push

build-image:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $IMAGE_NAME:$CI_COMMIT_SHA .
    - docker push $IMAGE_NAME:$CI_COMMIT_SHA
```

Key steps:
- **docker build**: Create the Docker image from the Dockerfile.
- **docker push**: Upload the image to Google Container Registry.

You need to authenticate GitLab Runner to GCR. Typically, you use a service account key stored in GitLab CI/CD Variables.

---

## 3. **Setting up Auto-Tagging for Correct Image Versions**

Instead of only tagging by commit SHA, you can automatically tag images by branch, version, or release type.

Example extension in `.gitlab-ci.yml`:

```
before_script:
  - export TAG=${CI_COMMIT_REF_SLUG}

build-image:
  script:
    - docker build -t $IMAGE_NAME:$TAG .
    - docker push $IMAGE_NAME:$TAG
```

Some common tagging strategies:
- Tag by branch name for non-production (`develop`, `feature/xyz`).
- Tag by commit hash for precise builds (`$CI_COMMIT_SHA`).
- Tag by Git tag or version number for releases (`$CI_COMMIT_TAG`).

You can improve traceability and rollbacks by having predictable and meaningful image tags.

---

## 4. **Optimizing Image Builds (Caching and Multi-Stage Builds)**

Building Docker images can be slow if not optimized. Some optimization techniques:

### 4.1 **Docker Layer Caching**

Docker caches layers to avoid rebuilding everything from scratch.

You can set up GitLab to use Docker cache by using the same build context and optionally storing cache externally.

Example with cache:

```
build-image:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker pull $IMAGE_NAME:cache || true
    - docker build --cache-from $IMAGE_NAME:cache -t $IMAGE_NAME:$CI_COMMIT_SHA .
    - docker push $IMAGE_NAME:$CI_COMMIT_SHA
    - docker tag $IMAGE_NAME:$CI_COMMIT_SHA $IMAGE_NAME:cache
    - docker push $IMAGE_NAME:cache
```

### 4.2 **Multi-Stage Builds**

Multi-stage builds help reduce image size and separate the build environment from the runtime environment.

Advantages:
- Smaller final images.
- Reduced security risks.
- Faster deployments.

Always build your application (e.g., Go binary) in one stage and copy the final output into a clean runtime base image like `distroless` or `alpine`.

---

## Summary

### Key Points:

- **Dockerfile**: Create production-optimized Dockerfiles, especially using multi-stage builds.
- **Build and Push**: Automate Docker image builds and pushes through GitLab CI/CD pipelines.
- **Auto-Tagging**: Set up tagging strategies based on commit SHA, branch, or release version.
- **Optimization**: Use Docker cache and multi-stage builds to speed up pipelines and make images lighter.

---