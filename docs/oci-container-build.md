# OCI Container Build Workflow

The OCI Container Build workflow provides a standardized method for building and publishing OCI-compatible container images using Docker Buildx and a remote BuildKit daemon.

It is designed to be consumed as a reusable GitHub Actions workflow by application and infrastructure repositories.

---

# Workflow

Reusable workflow:

```text
.github/workflows/build-oci-container.yaml
```

Features:

- Docker Buildx
- Remote BuildKit
- Multi-platform builds
- OCI metadata generation
- Automatic image tagging
- Registry-backed build cache
- Harbor-compatible OCI registry publishing

---

# Example Usage

```yaml
jobs:
  build:
    uses: your-org/ci-platform/.github/workflows/build-oci-container.yaml@v1

    with:
      image_name: applications/my-app
      dockerfile: Dockerfile

    secrets: inherit
```

A more complete example is available in:

```text
examples/build-example.yaml
```

---

# Required GitHub Variables

| Variable | Example |
|----------|---------|
| CI_REGISTRY | harbor.example.com |
| CI_REGISTRY_NAMESPACE | applications |

---

# Required GitHub Secrets

| Secret |
|--------|
| CI_REGISTRY_USERNAME |
| CI_REGISTRY_PASSWORD |

---

# Workflow Inputs

| Input | Required | Default | Description |
|--------|:--------:|---------|-------------|
| registry | No | vars.CI_REGISTRY | OCI registry hostname |
| registry_namespace | No | vars.CI_REGISTRY_NAMESPACE | Registry namespace/project |
| cache_namespace | No | ci-cache | Registry namespace for BuildKit cache |
| image_name | Yes | — | Image path within the registry |
| runner | No | ci-build | ARC runner label |
| container_image | No | ghcr.io/actions/actions-runner:latest | GitHub Actions job container |
| platforms | No | linux/amd64 | Target platform(s) |
| dockerfile | No | Dockerfile | Dockerfile path |
| context | No | . | Build context |
| buildkit_host | No | tcp://buildkitd.buildkit.svc.cluster.local:1234 | Remote BuildKit endpoint |
| build_args | No | *(empty)* | Build arguments |
| extra_tags | No | *(empty)* | Additional metadata-action tags |
| push | No | true | Push resulting image |

---

# Build Process

```text
GitHub Actions
        │
        ▼
Docker Buildx
        │
        ▼
Remote BuildKit
        │
 ┌──────┴─────────┐
 ▼                ▼
Registry Cache    OCI Registry
```

---

# Image Tagging

Automatically generated:

- branch
- pull request
- commit SHA
- semantic version
- latest

Additional tags may be supplied through the `extra_tags` input.

---

# Build Cache

Registry cache format:

```text
<registry>/<cache_namespace>/<image>:buildcache-<platform>
```

Benefits:

- Faster rebuilds
- Shared cache between ARC runners
- No local persistent storage required

---

# Runner Requirements

The GitHub Actions job container must include:

- Docker CLI
- Docker Buildx
- Git
- Bash

Default container:

```text
ghcr.io/actions/actions-runner:latest
```

---

# Test Workflow

Validation workflow:

```text
.github/workflows/build-test.yaml
```

Sample image:

```text
images/test/
```

This workflow verifies:

- Docker Buildx
- Remote BuildKit
- Registry authentication
- OCI image publishing
- Registry cache
- Metadata generation

---

# Outputs

The workflow publishes:

- OCI container image
- OCI labels
- OCI tags
- BuildKit cache

---

# Future Enhancements

- SBOM generation
- Cosign signing
- Vulnerability scanning
- Multi-registry publishing
- Provenance attestation