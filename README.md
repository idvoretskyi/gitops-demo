# GitOps Demo with Flux

This repository demonstrates GitOps principles using Flux CD with a simple Node.js application.

## Structure

```
├── flux-system/          # Flux CD system configuration
├── clusters/dev/         # Environment-specific configurations
├── apps/demo-app/        # Kubernetes manifests for the demo application
├── demo-app/             # Source code for the demo application
└── .github/workflows/    # CI/CD pipeline
```

## Demo Application

A simple Node.js 24 Express application that provides:
- Health check endpoint at `/health`
- Basic API response at `/`

## GitOps Workflow

1. **Code Changes**: Push changes to the `demo-app/` directory
2. **CI Pipeline**: GitHub Actions builds and pushes container image
3. **Manifest Update**: Pipeline updates Kubernetes manifests with new image tag
4. **Flux Sync**: Flux detects changes and deploys to cluster

## Setup Instructions

### Prerequisites
- Kubernetes cluster
- Flux CLI installed
- kubectl configured

### Install Flux

```bash
# Install Flux in the cluster
flux bootstrap github \
  --owner=idvoretskyi \
  --repository=gitops-demo \
  --branch=main \
  --path=./clusters/dev
```

### Local Development

```bash
# Run the demo app locally
cd demo-app
npm install
npm start
```

### Build Docker Image

```bash
cd demo-app
docker build -t gitops-demo-app:latest .
docker run -p 3000:3000 gitops-demo-app:latest
```

## Monitoring

Access the application:
- Health check: `curl http://localhost:3000/health`
- Main endpoint: `curl http://localhost:3000/`