# Troubleshooting GitOps Demo

## GitHub Actions Permission Issues

### Problem: Git Push Fails with 403 Forbidden

If the `update-manifests` job fails with a `403 Forbidden` error during `git push`, this indicates insufficient permissions for the GitHub Actions bot.

### Solution 1: Use Enhanced GITHUB_TOKEN (Recommended)

The current workflow is configured with enhanced GITHUB_TOKEN authentication. Ensure your repository settings allow this:

1. Go to repository **Settings** → **Actions** → **General**
2. Under **Workflow permissions**, select **"Read and write permissions"**
3. Check **"Allow GitHub Actions to create and approve pull requests"**

### Solution 2: Use Personal Access Token (Alternative)

If Solution 1 doesn't work, create a PAT:

1. **Generate PAT:**
   - Go to GitHub **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
   - Generate new token with `repo` scope
   
2. **Add as Repository Secret:**
   - Go to repository **Settings** → **Secrets and variables** → **Actions**
   - Add new secret named `GITOPS_TOKEN`
   
3. **Update Workflow:**
   Replace the checkout step in `.github/workflows/build-and-deploy.yml`:
   ```yaml
   - name: Checkout repository
     uses: actions/checkout@v4
     with:
       token: ${{ secrets.GITOPS_TOKEN }}
       fetch-depth: 0
   ```

### Solution 3: Repository Settings Check

Verify these repository settings:

- **Settings** → **Actions** → **General** → **Actions permissions**: "Allow all actions and reusable workflows"
- **Settings** → **Branches** → **Branch protection rules**: Ensure the rule doesn't block Actions from pushing

## Flux Installation Issues

### Problem: Flux Controllers Not Starting

Check if all controllers are running:
```bash
kubectl get pods -n flux-system
flux get all
```

If controllers are failing, check resource constraints or try reinstalling:
```bash
flux uninstall
flux install
```

## Demo App Issues

### Problem: Image Pull Errors

Ensure the Docker image is built and available:
```bash
docker build -t gitops-demo-app:latest demo-app/
docker images | grep gitops-demo-app
```

### Problem: Service Not Accessible

Check service and endpoints:
```bash
kubectl get svc -n demo-app
kubectl get endpoints -n demo-app
kubectl describe service demo-app-service -n demo-app
```