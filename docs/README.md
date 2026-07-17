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

The platform uses upstream tooling where possible to reduce repository maintenance.

Current validation tooling:

```text
alpine/crane
```

is used as the image validation runtime and is not maintained as a repository-owned image.

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