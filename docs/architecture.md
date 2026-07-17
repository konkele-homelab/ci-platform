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
* Lightweight OCI image validation using upstream tooling

---

# High-Level Architecture

```text id="8r5y7a"
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

      Container Registry                    OCI Validation
```

---

# Workflow Responsibilities

The platform uses separate workflows for separate responsibilities.

## Build Workflow

Workflow:

```text id="7r4h5d"
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

## Runtime Validation Workflow

Workflow:

```text id="q8e6z1"
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

```text id="x7t1mq"
.github/workflows/validate-container.yaml
```

Purpose:

* Verify registry availability
* Inspect OCI image metadata
* Confirm manifests exist
* Validate published artifacts

This workflow uses:

```text id="5q2d0n"
alpine/crane:latest
```

as the validation runtime.

The platform does not maintain a custom validation container image.

---

# Build Architecture

The build workflow uses Docker Buildx with a remote BuildKit daemon.

```text id="0czm4p"
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

```text id="v1g4ol"
tcp://buildkitd.buildkit.svc.cluster.local:1234
```

The endpoint can be overridden through workflow inputs.

---

# Container Registry Integration

The platform uses the registry for:

## Image Storage

Published container images are stored in the configured registry namespace.

Example:

```text id="9z7h6p"
registry.example.com/applications/my-image:latest
```

---

## Build Cache Storage

BuildKit cache is stored remotely:

```text id="9gq6jl"
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

```text id="r6d2be"
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

```text id="4y4w0t"
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
      |
      v

Deployment Pipeline
```

---

# Supporting Images

Supporting images are limited to platform-specific test artifacts.

Current repository-managed images:

```text id="3k5n1h"
images/
└── test/
    └── Dockerfile
```

The platform intentionally avoids maintaining custom tooling images when upstream images provide the required functionality.

Examples:

| Purpose                 | Image                    |
| ----------------------- | ------------------------ |
| OCI registry validation | `alpine/crane:latest`    |
| Platform smoke testing  | `images/test/Dockerfile` |

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

## Prefer Upstream Tooling

The platform should use upstream images and tools when they satisfy requirements.

Benefits:

* less maintenance
* fewer custom build dependencies
* easier upgrades
* smaller platform surface area