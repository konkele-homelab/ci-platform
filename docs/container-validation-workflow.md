# Container Validation Workflow

The Container Validation workflows provide reusable GitHub Actions workflows for verifying that published container images are functional, accessible, and correctly published.

The validation layer runs after an image build completes and provides two types of validation:

* Runtime validation
* Registry/image validation

Together, these workflows verify that a container image can be consumed successfully after publication.

Registry validation uses the upstream `alpine/crane` image. The platform no longer maintains a custom validation container image.

---

# Validation Lifecycle

The recommended container image lifecycle:

```text
Source Repository
        |
        v
build-container.yaml
        |
        v
Container Registry
        |
        +----------------+
        |                |
        v                v
smoke-container.yaml  validate-container.yaml
runtime validation    OCI image validation
```

The build workflow creates and publishes the image.

The validation workflows confirm:

* the image can start
* the image contains expected content
* the registry contains valid OCI image data
* consumers can retrieve the published artifact

---

# Available Validation Workflows

| Workflow                  | Purpose                                       |
| ------------------------- | --------------------------------------------- |
| `smoke-container.yaml`    | Verify container startup and runtime behavior |
| `validate-container.yaml` | Verify published OCI images using crane       |

---

# Runtime Smoke Testing

## Overview

The smoke test workflow validates that a published container image can start successfully and optionally perform runtime checks.

Use this workflow when you need to verify:

* the container can launch
* the image contains expected files
* required commands execute successfully

Workflow:

```text
.github/workflows/smoke-container.yaml
```

---

# Basic Example

```yaml
jobs:
  smoke:
    uses: your-org/ci-platform/.github/workflows/smoke-container.yaml@v1

    with:
      image: ${{ needs.build.outputs.reference }}

    secrets: inherit
```

---

# Smoke Test Inputs

| Input           | Required | Default    | Description                             |
| --------------- | :------: | ---------- | --------------------------------------- |
| `image`         |    Yes   | —          | Container image reference to test       |
| `runner`        |    No    | `ci-build` | GitHub Actions runner label             |
| `expected_file` |    No    | empty      | File expected inside the container      |
| `command`       |    No    | empty      | Command to execute inside the container |

---

# File Validation Example

A common pattern is verifying that an image contains expected application files.

Example:

```yaml
jobs:
  smoke:
    uses: your-org/ci-platform/.github/workflows/smoke-container.yaml@v1

    with:
      image: ${{ needs.build.outputs.reference }}
      expected_file: /app/version.txt

    secrets: inherit
```

The workflow will:

1. Start the container
2. Verify the file exists
3. Display the file contents
4. Fail if the file is missing

---

# Command Validation Example

The workflow can execute a command inside the container.

Example:

```yaml
jobs:
  smoke:
    uses: your-org/ci-platform/.github/workflows/smoke-container.yaml@v1

    with:
      image: ${{ needs.build.outputs.reference }}
      command: app --version

    secrets: inherit
```

This is useful for validating:

* application binaries
* runtime dependencies
* CLI tools

---

# Container Authentication

Private registry images require credentials:

```yaml
secrets: inherit
```

The workflow uses:

```text
CI_REGISTRY_USERNAME
CI_REGISTRY_PASSWORD
```

to authenticate before starting the container.

---

# Image Validation

## Overview

The image validation workflow verifies that published OCI container images exist and contain valid registry metadata.

Workflow:

```text
.github/workflows/validate-container.yaml
```

The workflow runs using:

```text
alpine/crane:latest
```

The upstream image provides the `crane` CLI used to inspect OCI registry artifacts.

No custom validation image is required.

No Docker daemon is required.

---

# Basic Example

```yaml
jobs:
  validate:
    uses: your-org/ci-platform/.github/workflows/validate-container.yaml@v1

    with:
      tags: |
        ${{ needs.build.outputs.reference }}

    secrets: inherit
```

---

# Validation Inputs

| Input              | Required | Default               | Description                        |
| ------------------ | :------: | --------------------- | ---------------------------------- |
| `tags`             |    Yes   | —                     | Newline-separated image references |
| `runner`           |    No    | `ci-build`            | GitHub Actions runner label        |
| `registry`         |    No    | auto-detected         | Container registry hostname        |
| `validation_image` |    No    | `alpine/crane:latest` | OCI validation runtime image       |

---

# Validation Checks

For each image reference, the workflow verifies:

## Registry Authentication

Confirms the workflow can access the registry.

---

## Image Digest

Retrieves the published image digest.

Example:

```text
sha256:abcdef123456...
```

---

## Image Manifest

Confirms the registry contains a valid OCI image manifest.

---

## Image Configuration

Confirms the image configuration can be retrieved successfully.

---

# Recommended Pipeline

A complete repository pipeline:

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

Execution order:

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
OCI Registry Validation
```

---

# When to Use Each Workflow

## Use Smoke Testing When:

You want to verify:

* application startup
* runtime dependencies
* expected files
* command execution

Examples:

* web application container
* CLI tool image
* runtime image

---

## Use Image Validation When:

You want to verify:

* registry publishing
* image availability
* image metadata
* manifest integrity

Examples:

* release pipelines
* production image promotion
* multi-platform builds

---

# Failure Troubleshooting

## Container Does Not Start

Check:

* image entrypoint
* required environment variables
* runtime dependencies
* registry credentials

---

## Expected File Missing

Verify:

* Dockerfile COPY instructions
* build context
* image tag being tested

---

## crane Not Found

Verify:

* the validation image provides crane
* the `validation_image` input has not been overridden incorrectly

Default:

```yaml
validation_image: alpine/crane:latest
```

---

## Registry Validation Fails

Verify:

* registry credentials
* image tag exists
* image push completed successfully
* registry permissions allow read access

---

# Best Practices

Recommended:

* Run smoke tests after every build
* Run image validation before promotion
* Validate release tags before deployment
* Keep validation separate from image building
* Prefer upstream validation tooling over maintaining custom images

The build workflow answers:

> "Can we create and publish this image?"

The validation workflows answer:

> "Can users successfully consume this image?"