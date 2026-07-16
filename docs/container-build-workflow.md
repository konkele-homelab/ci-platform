# Container Build Workflow

The Container Build workflow provides a reusable GitHub Actions workflow for building, publishing, and validating container images.

The workflow standardizes container image builds across repositories by combining:

* Docker Buildx
* Remote BuildKit
* Container image metadata generation
* Registry-backed caching
* Multi-platform image builds
* Automated image tagging
* Container registry publishing

The workflow is intended to be consumed by application and infrastructure repositories through GitHub Actions reusable workflows.

---

# Workflow Location

Reusable workflow:

```text
.github/workflows/build-container.yaml
```

Consumer repositories call the workflow:

```yaml
jobs:
  build:
    uses: your-org/ci-platform/.github/workflows/build-container.yaml@v1
```

---

# Build Lifecycle

The complete image lifecycle:

```text
Developer Commit
        |
        v
GitHub Actions Workflow
        |
        v
build-container.yaml
        |
        +----------------+
        |                |
        v                v
 Remote BuildKit    Registry Cache
        |
        v
 Container Image Published
        |
        +----------------+
        |                |
        v                v
 smoke-container   validate-container
 runtime test      image validation
```

---

# Features

## Container Builds

Supported:

* Container image builds
* Dockerfile builds
* Multi-platform builds
* Build arguments
* Custom build contexts
* Custom Dockerfile locations

---

## Remote BuildKit

The workflow uses a remote BuildKit daemon.

Default:

```text
tcp://buildkitd.buildkit.svc.cluster.local:1234
```

Benefits:

* Centralized build resources
* Faster builds
* Shared caching
* Reduced runner requirements

---

## Image Metadata

Images receive standardized metadata labels:

```text
org.opencontainers.image.title
org.opencontainers.image.source
org.opencontainers.image.url
org.opencontainers.image.documentation
org.opencontainers.image.revision
org.opencontainers.image.version
org.opencontainers.image.created
```

These labels provide consistent image ownership, source tracking, and version information.

---

# Required Configuration

## GitHub Variables

The calling repository requires:

| Variable                | Description                 | Example              |
| ----------------------- | --------------------------- | -------------------- |
| `CI_REGISTRY`           | Container registry hostname | `harbor.example.com` |
| `CI_REGISTRY_NAMESPACE` | Registry namespace/project  | `applications`       |

Example:

```text
CI_REGISTRY=harbor.example.com
CI_REGISTRY_NAMESPACE=applications
```

---

## GitHub Secrets

Required:

| Secret                 | Description                 |
| ---------------------- | --------------------------- |
| `CI_REGISTRY_USERNAME` | Container registry username |
| `CI_REGISTRY_PASSWORD` | Container registry password |

Recommended usage:

```yaml
secrets: inherit
```

---

# Basic Example

```yaml
name: Build Image

on:
  workflow_dispatch:

jobs:
  build:
    uses: your-org/ci-platform/.github/workflows/build-container.yaml@v1

    with:
      image_name: example-image
      dockerfile: Dockerfile

    secrets: inherit
```

---

# Complete Example

```yaml
jobs:
  build:
    uses: your-org/ci-platform/.github/workflows/build-container.yaml@v1

    with:
      image_name: applications/example-image

      platforms: |
        linux/amd64

      dockerfile: Dockerfile

      context: .

      build_args: |
        APP_ENV=production

      extra_tags: |
        type=raw,value=stable

    secrets: inherit
```

---

# Workflow Inputs

| Input                | Required | Default                                 | Description                          |
| -------------------- | :------: | --------------------------------------- | ------------------------------------ |
| `registry`           |    No    | `vars.CI_REGISTRY`                      | Container registry hostname          |
| `registry_namespace` |    No    | `vars.CI_REGISTRY_NAMESPACE`            | Registry namespace/project           |
| `cache_namespace`    |    No    | `ci-cache`                              | Registry location for BuildKit cache |
| `image_name`         |    Yes   | —                                       | Image path                           |
| `primary_tag`        |    No    | `latest`                                | Tag returned as primary reference    |
| `runner`             |    No    | `ci-build`                              | GitHub Actions runner label          |
| `container_image`    |    No    | `ghcr.io/actions/actions-runner:latest` | Job container                        |
| `platforms`          |    No    | `linux/amd64`                           | Target platforms                     |
| `dockerfile`         |    No    | `Dockerfile`                            | Dockerfile path                      |
| `context`            |    No    | `.`                                     | Build context                        |
| `buildkit_host`      |    No    | Remote BuildKit endpoint                | BuildKit daemon                      |
| `build_args`         |    No    | empty                                   | Docker build arguments               |
| `extra_tags`         |    No    | empty                                   | Additional metadata-action tags      |

---

# Workflow Outputs

The workflow exposes:

| Output      | Description                            |
| ----------- | -------------------------------------- |
| `image`     | Fully qualified image name without tag |
| `reference` | Primary image reference                |
| `tags`      | Generated image tags                   |

Example usage:

```yaml
needs:
  - build

with:
  image: ${{ needs.build.outputs.reference }}
```

---

# Image Tagging

Tags are generated automatically using Docker metadata-action.

Generated tags include:

## Branch

Example:

```text
main
```

---

## Pull Requests

Example:

```text
pr-123
```

---

## Commit SHA

Example:

```text
sha-a1b2c3d
```

---

## Semantic Versions

For release tags:

```text
v1.2.3
```

Generated:

```text
1.2.3
1
1.2
```

---

## Latest

Generated when:

* building the default branch
* publishing version releases

---

# Custom Tags

Additional tags can be supplied:

```yaml
extra_tags: |
  type=raw,value=testing
  type=raw,value=nightly
```

These are added alongside automatically generated tags.