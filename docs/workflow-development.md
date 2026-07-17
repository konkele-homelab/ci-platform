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
* registry validation

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

The platform avoids maintaining custom images when suitable upstream images exist.

Examples:

| Purpose                | Image                    |
| ---------------------- | ------------------------ |
| Registry validation    | `alpine/crane:latest`    |
| Platform smoke testing | `images/test/Dockerfile` |

Custom workflow tooling images should only be added when upstream alternatives do not provide the required functionality.

---

# Validation Tooling

Registry validation uses the upstream:

```text
alpine/crane:latest
```

image.

The validation workflow uses `crane` for:

* registry authentication
* image digest verification
* OCI manifest inspection
* image configuration inspection

The repository does not maintain a custom validation container image.

Benefits:

* reduced repository maintenance
* fewer custom build dependencies
* current upstream tooling
* smaller validation runtime

---

# Versioning Workflows

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
* upstream tooling over custom images
* explicit interfaces over hidden behavior
* documented lifecycle patterns

Improve shared workflows rather than adding duplicated repository-specific automation.
