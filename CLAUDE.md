# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a GitOps demonstration repository using Flux CD with a Node.js demo application. It showcases modern GitOps practices with automated CI/CD pipelines and Kubernetes deployments.

## Repository Structure

```
├── flux-system/          # Flux CD system configuration
├── clusters/dev/         # Environment-specific configurations  
├── apps/demo-app/        # Kubernetes manifests for demo application
├── demo-app/             # Node.js Express application source code
└── .github/workflows/    # GitHub Actions CI/CD pipeline
```

## Development Commands

**Demo Application (Node.js)**:
```bash
cd demo-app
npm install          # Install dependencies
npm start           # Run application (port 3000)
npm run dev         # Run in development mode
```

**Docker Commands**:
```bash
cd demo-app
docker build -t gitops-demo-app:latest .
docker run -p 3000:3000 gitops-demo-app:latest
```

**Kubernetes/Flux Commands**:
```bash
# Install Flux in cluster
flux bootstrap github --owner=idvoretskyi --repository=gitops-demo --branch=main --path=./clusters/dev

# Check Flux status
flux get all

# Validate Kubernetes manifests
kubectl apply --dry-run=client -k apps/demo-app/
```

## Architecture

**GitOps Workflow**:
1. Code changes pushed to `demo-app/` directory
2. GitHub Actions builds and pushes container image to GHCR
3. Pipeline updates Kubernetes manifests with new image tag
4. Flux detects manifest changes and deploys to cluster

**Application Stack**:
- **Runtime**: Node.js 24 Alpine
- **Framework**: Express.js
- **Container Registry**: GitHub Container Registry (GHCR)
- **GitOps Tool**: Flux CD
- **Orchestration**: Kubernetes with Kustomize

## Security Features

**Code Scanning**:
- CodeQL analysis for JavaScript security vulnerabilities
- Trivy vulnerability scanning for dependencies and Docker images
- Gitleaks for secrets detection
- npm audit for Node.js dependency vulnerabilities

**Container Security**:
- Non-root user execution (UID 1000)
- Read-only root filesystem with writable volumes for /tmp and cache
- Dropped ALL capabilities including NET_RAW
- Security contexts with privilege escalation disabled
- Runtime/default seccomp profile
- Health checks built into Docker image

**Kubernetes Security**:
- SecurityContext with runAsNonRoot and high UID/GID (30000)
- No service account token mounting
- NetworkPolicy for traffic isolation
- Resource limits and requests defined
- Specific image tags with imagePullPolicy: Always
- Image digests in CI/CD for immutable deployments

**Infrastructure Security**:
- Kubesec for Kubernetes manifest security analysis
- Polaris for Kubernetes best practices validation
- Checkov for Infrastructure as Code security scanning

**Automated Security Updates**:
- Dependabot for dependency updates (npm, Docker, GitHub Actions)
- Weekly scheduled security scans

## Key Files

- `demo-app/src/app.js` - Main application entry point
- `apps/demo-app/deployment.yaml` - Local development deployment (tag-based)
- `apps/demo-app/deployment-production.yaml` - Production deployment (digest-based)
- `flux-system/gotk-components.yaml` - Flux system configuration
- `.github/workflows/build-and-deploy.yml` - CI/CD pipeline with security scanning
- `.github/workflows/codeql.yml` - CodeQL security analysis
- `.github/workflows/security.yml` - Comprehensive security scanning
- `.github/dependabot.yml` - Automated dependency updates
- `SECURITY.md` - Comprehensive security implementation guide
- `IMAGE-DIGEST-SOLUTION.md` - Image digest compliance solution