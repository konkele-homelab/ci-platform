# CI Platform

Reusable GitHub Actions workflows and container tooling for standardized CI/CD pipelines.

This repository provides shared automation for building, testing, validating, and publishing container images across multiple repositories.

The goal is to reduce duplicated CI configuration while providing consistent build workflows, image lifecycle management, validation, and reusable automation patterns.

---

# Overview

The CI Platform provides reusable workflows for:

* Building container images
* Publishing images to container registries
* Using remote BuildKit builders
* Generating standardized image metadata
* Sharing registry-backed build cache
* Running container runtime validation
* Validating published images

The platform is designed for:

* GitHub Actions reusable workflows
* GitHub Actions Runner Controller (ARC)
* Container registries such as Harbor
* Docker Buildx
* Remote BuildKit deployments

---

# Repository Structure

The repository is organized into a few primary areas:

```text
.
├── .github/
│   └── workflows/     # Reusable workflows and automation pipelines
│
├── docs/              # Platform documentation and workflow guides
│
├── examples/          # Consumer workflow examples
│
├── images/            # Supporting container images used by workflows
│
└── README.md          # Platform overview
```

The repository intentionally separates:

* workflow implementation
* documentation
* consumer examples
* supporting images

This allows consuming repositories to use the workflows without needing to understand the internal platform implementation.

---

# Quick Start

A consuming repository can build a container image by calling the reusable workflow:

```yaml
name: Build Container Image

on:
  workflow_dispatch:

jobs:
  build:
    uses: your-org/ci-platform/.github/workflows/build-container.yaml@v1

    with:
      image_name: my-application
      dockerfile: Dockerfile

    secrets: inherit
```

Minimal example:

```text
examples/build-example-minimal.yaml
```

Advanced configuration:

```text
examples/build-example-complete.yaml
```

---

# CI Workflow Architecture

The platform uses a container image lifecycle pipeline:

```text
                 Source Repository
                        |
                        |
                        v
              build-container.yaml
                        |
                        |
                        v
              Container Registry
                        |
             +----------+----------+
             |                     |
             v                     v
  smoke-container.yaml    validate-container.yaml
       runtime test          image validation
```

The build workflow creates and publishes images.

The validation workflows verify that published images:

* start correctly
* contain expected content
* have valid registry metadata
* can be consumed successfully

---

# Available Workflows

| Workflow                  | Purpose                                       |
| ------------------------- | --------------------------------------------- |
| `build-container.yaml`    | Build and publish container images            |
| `smoke-container.yaml`    | Verify container startup and runtime behavior |
| `validate-container.yaml` | Validate published images and metadata        |
| `test-platform.yaml`      | Verify the CI platform itself                 |

---

# Container Build Features

The build workflow provides:

## Build System

* Docker Buildx
* Remote BuildKit
* Multi-platform builds
* Registry-backed cache
* Build argument support
* Custom Dockerfile locations
* Custom build contexts

---

## Image Metadata

Generated automatically:

* Image title
* Source repository
* Documentation reference
* Revision
* Version
* Creation timestamp

The metadata follows standard container image labeling conventions.

---

## Image Tags

Generated automatically:

* Branch tags
* Pull request tags
* Commit SHA tags
* Semantic version tags
* Latest tags for releases/default branch

Additional tags can be added with:

```yaml
extra_tags: |
  type=raw,value=testing
```

---

# Required Configuration

## GitHub Variables

The calling repository must provide:

| Variable                | Description                 |
| ----------------------- | --------------------------- |
| `CI_REGISTRY`           | Container registry hostname |
| `CI_REGISTRY_NAMESPACE` | Registry namespace/project  |

Example:

```text
CI_REGISTRY=harbor.example.com
CI_REGISTRY_NAMESPACE=applications
```

---

## GitHub Secrets

Required secrets:

| Secret                 | Description                 |
| ---------------------- | --------------------------- |
| `CI_REGISTRY_USERNAME` | Container registry username |
| `CI_REGISTRY_PASSWORD` | Container registry password |

Recommended consumer pattern:

```yaml
secrets: inherit
```

or explicitly:

```yaml
secrets:
  CI_REGISTRY_USERNAME: ${{ secrets.CI_REGISTRY_USERNAME }}
  CI_REGISTRY_PASSWORD: ${{ secrets.CI_REGISTRY_PASSWORD }}
```

---

# Build Cache

BuildKit cache is stored in the container registry:

```text
<registry>/<cache_namespace>/<image>:buildcache-<platform>
```

Example:

```text
harbor.example.com/ci-cache/my-app:buildcache-linux-amd64
```

Benefits:

* Faster rebuilds
* Shared cache between runners
* No dependency on runner-local storage

---

# Consumer Repository Requirements

Repositories using this platform require:

## Container Build Definition

A standard Dockerfile:

```text
Dockerfile
```

or a custom location:

```yaml
dockerfile: containers/app/Dockerfile
```

---

## Registry Access

The workflow requires:

* registry hostname
* registry namespace
* push credentials

---

## Runner

The default runner is:

```text
ci-build
```

Override when needed:

```yaml
runner: custom-runner
```

---

# Development and Testing

The platform includes supporting images and validation workflows used to verify platform behavior.

These validate:

* workflow execution
* image publishing
* container startup
* registry access
* image metadata

---

# Versioning

Consumers should reference released versions.

Recommended:

```yaml
uses: your-org/ci-platform/.github/workflows/build-container.yaml@v1
```

Pinned release:

```yaml
uses: your-org/ci-platform/.github/workflows/build-container.yaml@v1.0.0
```

Avoid consuming:

```yaml
@main
```

for production pipelines.

---

# Documentation

Detailed workflow documentation:

```text
docs/container-build-workflow.md
```

Examples:

```text
examples/
```