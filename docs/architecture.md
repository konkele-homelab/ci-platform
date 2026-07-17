# CI Platform Architecture

The CI Platform provides reusable GitHub Actions workflows and supporting container tooling for standardized container build and validation pipelines.

The platform separates container lifecycle responsibilities into independent workflows that can be consumed by multiple repositories.

---

# Platform Goals

The platform is designed to provide:

* Consistent container image builds
* Shared CI/CD automation
* Centralized BuildKit resources
* Repeatable validation patterns
* Reduced duplicated workflow configuration
* Standardized image publishing practices
* Controlled OCI image validation environments

---

# High-Level Architecture

```text
                         Application Repository

                                  |
                                  |
                                  v

                       GitHub Actions Workflow

                                  |
                                  |
                                  v

                     Reusable CI Platform Workflows

             +--------------------+--------------------+
             |                                         |
             v                                         v

    build-container.yaml                    validation workflows
             |                                         |
             |                                         |
             v                                         v

       Remote BuildKit                    smoke-container.yaml
             |                            validate-container.yaml
             |                                         |
             v                                         v

      Container Registry              container-validator image
                                                  |
                                                  v
                                      OCI Validation Runtime
```

---

# Workflow Responsibilities

The platform uses separate workflows for separate responsibilities.

## Build Workflow

Workflow:

```text
.github/workflows/build-container.yaml
```

Purpose:

* Build container images
* Generate image metadata
* Apply image tags
* Push images
* Manage build cache

The build workflow should not perform application-specific validation.

---

## Validation Image Build Workflow

Workflow:

```text
.github/workflows/build-validation-image.yaml
```

Purpose:

* Build the platform validation runtime image
* Publish the validation image
* Maintain a controlled validation environment

The validation image is built from:

```text
images/validation/Dockerfile
```

The image provides the tooling required by validation workflows.

---

## Runtime Validation Workflow

Workflow:

```text
.github/workflows/smoke-container.yaml
```

Purpose:

* Start published containers
* Verify runtime behavior
* Execute validation commands
* Verify expected files

This validates that the image works as a runtime artifact.

---

## OCI Image Validation Workflow

Workflow:

```text
.github/workflows/validate-container.yaml
```

Purpose:

* Verify registry availability
* Inspect OCI image metadata
* Confirm manifests exist
* Validate published artifacts

This workflow executes inside the platform validation runtime image:

```text
container-validator
```

The validation image is maintained by this repository and provides the required OCI inspection tooling.

---

# Build Architecture

The build workflow uses Docker Buildx with a remote BuildKit daemon.

```text
GitHub Actions Runner

        |
        |
        v

     Docker Buildx

        |
        |
        v

 Remote BuildKit

        |
        +----------------+
        |                |
        v                v

 Registry Cache     Container Registry
```

---

# Remote BuildKit

Remote BuildKit provides:

* Centralized build resources
* Shared build cache
* Consistent build environment
* Reduced runner dependencies

Default endpoint:

```text
tcp://buildkitd.buildkit.svc.cluster.local:1234
```

The endpoint can be overridden through workflow inputs.

---

# Container Registry Integration

The platform uses the registry for:

## Image Storage

Published container images are stored in the configured registry namespace.

Example:

```text
registry.example.com/applications/my-image:latest
```

---

## Build Cache Storage

BuildKit cache is stored remotely:

```text
<registry>/<cache_namespace>/<image>:buildcache-<platform>
```

Benefits:

* Cache persistence
* Shared builds across runners
* Faster rebuilds

---

# Runner Architecture

The platform is designed for GitHub Actions Runner Controller (ARC).

Typical flow:

```text
GitHub Actions

      |

      v

ARC Runner

      |

      v

Workflow Container

      |

      v

Build / Validation Tooling
```

---

# Image Lifecycle

The standard lifecycle is:

```text
Developer Change

      |
      v

Container Build

      |
      v

Registry Publish

      |
      +----------------+
      |                |
      v                v

Runtime Test      OCI Validation
      |                |
      |                |
      v                v

smoke-container   validate-container
                       |
                       v
              container-validator image
```

---

# Supporting Images

Supporting images are stored under:

```text
images/
```

Current repository-managed images:

```text
images/
├── test/
│   └── Dockerfile
│
└── validation/
    └── Dockerfile
```

---

## Test Image

The test image is used for platform workflow testing.

Purpose:

* Verify image building
* Verify runtime execution
* Validate workflow behavior

---

## Validation Image

The validation image provides the runtime environment for OCI validation.

Purpose:

* Provide consistent validation tooling
* Control validation dependencies
* Avoid relying on external runtime images

The validation image includes:

* POSIX shell
* crane OCI registry tooling
* certificate bundles
* required runtime utilities

The image lifecycle is managed by:

```text
.github/workflows/build-validation-image.yaml
```

---

# Design Principles

## Reusable First

Workflows should be reusable across repositories.

Prefer:

```yaml
uses: organization/platform/.github/workflows/workflow.yaml@v1
```

over duplicated workflow files.

---

## Separation of Responsibilities

Each workflow should have a clear purpose.

Examples:

* Building is separate from testing
* Testing is separate from publishing validation
* Platform validation is separate from application validation

---

## Explicit Interfaces

Reusable workflows should clearly define:

* inputs
* secrets
* outputs
* expected behavior

---

## Controlled Tooling Environments

Platform-owned container images should only be created when they provide value over external dependencies.

The validation image exists because it provides:

* reproducibility
* controlled tool versions
* predictable workflow execution
* reduced external dependency risk