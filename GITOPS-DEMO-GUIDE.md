# GitOps Demo Guide

This guide provides step-by-step instructions for beginners who want to learn GitOps using this demonstration repository.

## Prerequisites

- Docker Desktop installed
- kubectl configured with a Kubernetes cluster
- GitHub account

## Step 1: Understand the Application

```bash
cd demo-app
npm install
npm start
# Visit http://localhost:3000 to see the app
```

## Step 2: Build and Test Container

```bash
docker build -t gitops-demo-app:latest .
docker run -p 3000:3000 gitops-demo-app:latest
```

## Step 3: Install Flux (GitOps Tool)

```bash
# Install Flux CLI first: https://fluxcd.io/flux/installation/
flux bootstrap github --owner=YOUR_GITHUB_USERNAME --repository=gitops-demo --branch=main --path=./clusters/dev
```

## Step 4: Watch GitOps in Action

```bash
# Monitor Flux status
flux get all

# Watch deployments
kubectl get pods -w
```

## Step 5: Trigger GitOps Workflow

1. Make a change to `demo-app/src/app.js`
2. Commit and push to main branch
3. GitHub Actions builds new image automatically
4. Pipeline updates Kubernetes manifests
5. Flux detects changes and deploys automatically

## Step 6: Verify Deployment

```bash
kubectl get deployments
kubectl get services
# Access the updated application
```

## Key GitOps Concept

Your Git repository is the single source of truth - changes to code automatically trigger deployments without manual kubectl commands.