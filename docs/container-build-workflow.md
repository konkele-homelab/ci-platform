# Container Validation Workflow

The Container Validation workflows provide reusable GitHub Actions workflows for verifying that published container images are functional, accessible, and correctly published.

The validation layer is designed to run after an image build completes and provides two types of validation:

* Runtime validation
* Registry/image validation

Together, these workflows verify that a container image can be consumed successfully after publication.

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
runtime validation    image validation