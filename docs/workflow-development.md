# Workflow Development Guide

This guide describes how to develop, modify, and maintain workflows in the CI Platform repository.

The goal is to keep workflows reusable, predictable, and safe for consumption by other repositories.

---

# Adding a New Workflow

Reusable workflows should be created under:

```text
.github/workflows/
```

A reusable workflow must define:

```yaml
on:
  workflow_call:
```

Example:

```yaml
name: Example Workflow

on:
  workflow_call:

    inputs:
      example:
        required: true
        type: string

    secrets:
      EXAMPLE_SECRET:
        required: true
```

---

# Workflow Design Guidelines

## Keep Workflows Generic

Workflows should provide reusable automation.

Avoid:

* repository-specific paths
* application-specific commands
* hardcoded environments

Prefer:

```yaml
inputs:
  command:
    required: false
    type: string
```

over:

```yaml
run: ./my-application-test.sh
```

---

# Document Workflow Interfaces

Every reusable workflow should document:

## Inputs

Example:

```yaml
inputs:
  image:
    description: Container image reference
    required: true
    type: string
```

---

## Secrets

Example:

```yaml
secrets:
  REGISTRY_PASSWORD:
    required: true
```

---

## Outputs

Example:

```yaml
outputs:
  image:
    description: Published image
    value: ${{ jobs.build.outputs.image }}
```

---

# Testing Workflow Changes

Changes should be tested using platform workflows.

Available validation:

```text
.github/workflows/test-platform.yaml
```

Testing should verify:

* workflow syntax
* image building
* image publishing
* runtime validation
* OCI registry validation

---

# Supporting Images

Supporting images are stored under:

```text
images/
```

Supporting images should:

* have minimal dependencies
* be reproducible
* document their purpose
* be tested through workflows

The platform maintains custom images only when they provide platform-specific value.

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

# Platform Test Image

The test image provides a minimal runtime artifact used to validate container workflow behavior.

Purpose:

* Verify image builds
* Verify runtime execution
* Confirm smoke testing behavior

Location:

```text
images/test/Dockerfile
```

---

# Validation Runtime Image

The validation runtime image provides the execution environment for OCI image validation workflows.

Location:

```text
images/validation/Dockerfile
```

The validation image includes:

* POSIX shell
* crane OCI registry tooling
* certificate bundles
* required runtime utilities

The image is built by:

```text
.github/workflows/build-validation-image.yaml
```

and consumed by:

```text
.github/workflows/validate-container.yaml
```

The validation runtime image exists to provide:

* reproducible validation execution
* controlled tooling versions
* consistent CI behavior
* reduced dependency on external images

---

# Adding or Updating Supporting Images

When modifying a supporting image:

1. Update the Dockerfile
2. Update related workflow documentation
3. Verify the image builds successfully
4. Verify consuming workflows continue to function

Example:

```text
images/validation/Dockerfile
```

changes should be validated through:

```text
.github/workflows/build-validation-image.yaml
```

---

# Validation Tooling

The validation workflow uses `crane` for OCI registry operations.

The validation runtime provides:

* registry authentication support
* image digest verification
* OCI manifest inspection
* image configuration inspection

The validation tooling should remain isolated from application build workflows.

---

# Workflow Versioning

Consumers should use version tags:

```yaml
uses: organization/platform/.github/workflows/build-container.yaml@v1
```

Avoid requiring consumers to follow:

```yaml
@main
```

Production workflows should use stable versions.

---

# Pull Request Expectations

Workflow changes should include:

## Documentation Updates

Update relevant files under:

```text
docs/
```

when changing:

* workflow inputs
* workflow outputs
* behavior
* architecture
* validation tooling
* supporting images

---

## Example Updates

Update:

```text
examples/
```

when changing:

* required inputs
* recommended usage
* workflow syntax

---

## Testing

Verify:

* workflow execution
* image publishing
* runtime validation
* OCI registry validation
* supporting image builds

---

# Adding New Platform Capabilities

When adding a new reusable workflow:

Consider:

## Purpose

Does this solve a common problem across repositories?

---

## Interface

Does it expose a simple reusable contract?

---

## Validation

Can the workflow test itself?

---

## Documentation

Can another team consume it without reading the implementation?

---

# Maintenance Principles

The CI Platform should optimize for:

* reliability
* consistency
* discoverability
* minimal consumer configuration

Prefer:

* reusable workflows over duplicated automation
* controlled platform images over unmanaged dependencies
* explicit interfaces over hidden behavior
* documented lifecycle patterns

Improve shared workflows rather than adding duplicated repository-specific automation.