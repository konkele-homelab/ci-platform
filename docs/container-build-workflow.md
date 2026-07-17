# Container Build Workflow

The Container Build Workflow provides a reusable GitHub Actions workflow for building and publishing OCI container images.

The workflow standardizes:

* Docker Buildx usage
* Remote BuildKit integration
* Registry authentication
* Image metadata generation
* Image tagging
* Registry-backed caching
* Multi-platform image publishing

---

# Build Lifecycle

The standard container build lifecycle:

```text
Source Repository
        |
        v
build-container.yaml
        |
        v
Remote BuildKit
        |
        v
Container Registry
        |
        v
Validation Workflows
```

The build workflow is responsible for creating and publishing images.

Validation workflows are responsible for verifying the published artifact.

---

# Workflow Location

Reusable workflow:

```text
.github/workflows/build-container.yaml
```

---

# Basic Usage

A consuming repository can build an image using:

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

---

# Workflow Inputs

| Input | Required | Default | Description |
| --- | :---: | --- | --- |
| `image_name` | Yes | — | Registry image path |
| `registry` | No | platform default | OCI registry hostname |
| `registry_namespace` | No | platform default | Registry namespace/project |
| `dockerfile` | No | `Dockerfile` | Dockerfile path |
| `context` | No | `.` | Docker build context |
| `platforms` | No | `linux/amd64` | Target build platforms |
| `runner` | No | `ci-build` | GitHub Actions runner label |
| `buildkit_host` | No | platform default | Remote BuildKit endpoint |
| `build_args` | No | empty | Docker build arguments |
| `extra_tags` | No | empty | Additional metadata-action tags |

---

# Workflow Outputs

The workflow exposes:

| Output | Description |
| --- | --- |
| `image` | Fully-qualified image name without tag |
| `reference` | Immutable SHA image reference |
| `tags` | Published image tags |

Example:

```yaml
needs.build.outputs.reference
```

returns:

```text
registry.example.com/applications/my-image:sha-abcdef123456
```

---

# Build System

The workflow uses:

* Docker Buildx
* Remote BuildKit
* Registry-backed cache

Architecture:

```text
GitHub Actions Runner

        |
        v

Docker Buildx

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

The workflow connects to a shared BuildKit service.

Default:

```text
tcp://buildkitd.buildkit.svc.cluster.local:1234
```

Benefits:

* shared build resources
* consistent build environment
* faster builds
* reduced runner dependencies

The BuildKit endpoint can be overridden:

```yaml
with:
  buildkit_host: tcp://custom-buildkit.example.com:1234
```

---

# Registry Configuration

The workflow uses:

## Variables

Required repository variables:

| Variable | Description |
| --- | --- |
| `CI_REGISTRY` | OCI registry hostname |
| `CI_REGISTRY_NAMESPACE` | Registry namespace/project |

Example:

```text
CI_REGISTRY=harbor.example.com
CI_REGISTRY_NAMESPACE=applications
```

---

## Secrets

Required secrets:

| Secret | Description |
| --- | --- |
| `CI_REGISTRY_USERNAME` | Registry username |
| `CI_REGISTRY_PASSWORD` | Registry password |

Recommended:

```yaml
secrets: inherit
```

---

# Image Metadata

The workflow automatically creates OCI labels:

```text
org.opencontainers.image.title
org.opencontainers.image.source
org.opencontainers.image.url
org.opencontainers.image.documentation
org.opencontainers.image.revision
org.opencontainers.image.version
org.opencontainers.image.created
```

These labels provide standardized image metadata for downstream consumers.

---

# Image Tags

The workflow automatically generates:

## Branch Tags

Example:

```text
main
feature-example
```

---

## Pull Request Tags

Example:

```text
pr-123
```

---

## SHA Tags

Example:

```text
sha-a1b2c3d4
```

SHA tags provide immutable references.

---

## Semantic Version Tags

For version releases:

```text
1.2.3
1
1.2
```

---

## Latest Tags

The workflow publishes:

```text
latest
```

for:

* main branch builds
* version tag releases

---

# Additional Tags

Additional tags can be added with:

```yaml
extra_tags: |
  type=raw,value=testing
```

This is useful for:

* development channels
* test images
* promotion workflows
* temporary release references

Example:

```yaml
with:
  image_name: my-application

  extra_tags: |
    type=raw,value=integration
```

---

# Build Arguments

Build arguments can be provided:

```yaml
with:
  build_args: |
    APP_VERSION=1.2.3
    FEATURE_FLAG=true
```

The workflow automatically provides:

```text
REGISTRY
REGISTRY_NAMESPACE
```

as build arguments.

---

# Build Cache

The workflow uses registry-backed BuildKit cache.

Cache location:

```text
<registry>/<cache_namespace>/<image>:buildcache-<platform>
```

Example:

```text
harbor.example.com/ci-cache/my-image:buildcache-linux-amd64
```

Benefits:

* persistent cache
* shared cache across runners
* faster rebuilds

---

# Dockerfile Locations

The default Dockerfile:

```text
Dockerfile
```

Custom Dockerfiles are supported:

```yaml
with:

  dockerfile: containers/application/Dockerfile
```

---

# Multi-Platform Builds

Multiple platforms can be specified:

```yaml
with:

  platforms: linux/amd64,linux/arm64
```

Buildx creates OCI-compatible multi-platform images.

---

# Recommended Consumer Pipeline

A complete pipeline:

```yaml
jobs:

  build:
    uses: your-org/ci-platform/.github/workflows/build-container.yaml@v1

    with:
      image_name: my-application

    secrets: inherit


  smoke:
    needs:
      - build

    uses: your-org/ci-platform/.github/workflows/smoke-container.yaml@v1

    with:
      image: ${{ needs.build.outputs.reference }}

    secrets: inherit


  validate:
    needs:
      - build
      - smoke

    uses: your-org/ci-platform/.github/workflows/validate-container.yaml@v1

    with:
      tags: |
        ${{ needs.build.outputs.reference }}

    secrets: inherit
```

Pipeline execution:

```text
Build
 |
 v
Publish
 |
 v
Runtime Smoke Test
 |
 v
OCI Image Validation
```

---

# Troubleshooting

## Build Cache Not Used

Verify:

* registry access
* cache namespace permissions
* BuildKit availability

---

## Image Not Published

Verify:

* registry credentials
* image namespace permissions
* generated image tags

---

## Invalid Dockerfile Path

Verify:

* Dockerfile exists
* build context contains referenced files

Example:

```yaml
dockerfile: images/application/Dockerfile
context: .
```

---

# Best Practices

Recommended:

* Use immutable SHA references for validation
* Use version tags for releases
* Use registry-backed cache
* Keep Dockerfiles reproducible
* Separate build and validation responsibilities

The build workflow answers:

> "Can we create and publish this container image?"