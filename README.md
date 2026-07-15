# CI Platform

Reusable GitHub Actions workflows and supporting container images for the Konkele Homelab CI/CD platform.

This repository centralizes reusable GitHub Actions workflows, utility workflows, and supporting container images that can be shared across multiple repositories. Its goal is to provide consistent, reusable CI/CD automation while keeping application and infrastructure repositories focused on their primary purpose.

---

## Features

- 🚀 Reusable GitHub Actions workflows
- 🏗️ Docker Buildx with remote BuildKit
- 🌐 Multi-platform OCI image builds
- 📦 OCI metadata generation
- ⚡ Registry-backed BuildKit cache
- 🏷️ Standardized image tagging
- 🐳 Harbor-compatible OCI registry publishing
- ☸️ Designed for GitHub Actions Runner Controller (ARC)
- 🔁 Shared CI/CD automation across repositories

---

## Repository Structure

```text
.
├── .github/
│   └── workflows/      # Reusable workflows and standalone utility workflows
├── docs/               # Workflow and platform documentation
├── examples/           # Example implementations
├── images/             # Runner and utility container images
│   └── ...
└── README.md
```

---

# Available Workflows

| Workflow | Description |
|----------|-------------|
| OCI Container Build | Build and publish OCI container images using Docker Buildx and remote BuildKit. |

Detailed documentation:

- 📖 `docs/oci-container-build.md`

---

# Versioning

Consumers should reference released versions rather than branch names.

Examples:

```text
@v1
```

or

```text
@v1.0.0
```

---

# Roadmap

Planned additions include:

## Reusable Workflows

- Kubernetes deployment
- Infrastructure provisioning
- Release automation
- Repository maintenance

## Supporting Images

- Build runner
- Kubernetes runner
- Ansible runner
- Additional utility images