# Documentation

## Users

- Container Build Workflow
- Container Validation Workflow

## Contributors

- Platform Architecture
- Workflow Development

---

# Platform Overview

The CI Platform provides reusable GitHub Actions workflows for:

* Building container images
* Publishing OCI images
* Running runtime smoke tests
* Validating published container artifacts

The platform maintains a small set of repository-owned supporting container images where required.

Current validation tooling:

```text
container-validator
```

is the platform-owned validation runtime image used for OCI image validation workflows.

---

# Documentation Map

## Container Build Workflow

See:

```text
docs/container-build-workflow.md
```

Covers:

* Build workflow usage
* BuildKit configuration
* Registry caching
* Image tagging
* Workflow inputs and outputs

---

## Container Validation Workflow

See:

```text
docs/container-validation-workflow.md
```

Covers:

* Runtime smoke testing
* OCI image validation
* Registry verification
* Validation workflow usage
* Validation runtime image configuration

---

## Architecture

See:

```text
docs/architecture.md
```

Covers:

* Platform design
* Workflow responsibilities
* Build architecture
* Validation architecture
* Supporting container images

---

## Workflow Development

See:

```text
docs/workflow-development.md
```

Covers:

* Creating reusable workflows
* Interface design
* Testing changes
* Maintaining platform components

---

# Supporting Container Images

The platform maintains supporting images required for CI workflows.

Current repository-owned images:

```text
images/
├── test/
│   └── Dockerfile
│
└── validation/
    └── Dockerfile
```

## Test Image

The test image provides a minimal runtime image used by platform validation workflows.

Purpose:

* Verify container build workflows
* Test runtime execution
* Confirm image publishing behavior

---

## Validation Image

The validation image provides the runtime environment used by OCI validation workflows.

Purpose:

* Provide a consistent validation environment
* Bundle required validation tooling
* Avoid depending on external validation images

The validation image includes:

* POSIX shell
* crane OCI registry tooling
* certificate bundles
* required runtime dependencies

The image is built by:

```text
.github/workflows/build-validation-image.yaml
```

and consumed by:

```text
.github/workflows/validate-container.yaml
```

---

# Platform Design Principles

The platform prefers:

* reusable workflows
* explicit workflow interfaces
* centralized CI logic
* reproducible tooling environments
* minimal consumer configuration

Supporting container images are maintained only when they provide platform-specific functionality that is not appropriate to depend on externally.

Benefits:

* consistent execution environments
* controlled tool versions
* predictable validation behavior
* easier platform maintenance