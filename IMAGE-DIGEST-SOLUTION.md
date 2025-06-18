# Image Digest Implementation for CKV_K8S_43 Compliance

## Security Requirement: Image Should Use Digest

**Policy**: `CKV_K8S_43` - Images should use digest instead of tags for immutable deployments.

## Problem
The security scanner flagged the use of image tags (`gitops-demo-app:v1.0.1`) instead of digests (`gitops-demo-app@sha256:...`), which could allow image content to change unexpectedly.

## Solution Architecture

### 1. Production Deployment (Digest-based)
```yaml
# apps/demo-app/deployment-production.yaml
containers:
- name: demo-app
  image: ghcr.io/idvoretskyi/gitops-demo/demo-app@sha256:de9f52436a01f88fbda38fe092d48589d89f26517001b8ceb06a6f08140f61ef
  imagePullPolicy: Always
```

### 2. Local Development (Tag-based)
```yaml
# apps/demo-app/deployment.yaml
containers:
- name: demo-app
  image: gitops-demo-app:v1.0.1
  imagePullPolicy: Always
```

## Implementation Steps

### For Production Environments

1. **Push images to container registry**:
   ```bash
   docker build -t ghcr.io/idvoretskyi/gitops-demo/demo-app:v1.0.1 demo-app/
   docker push ghcr.io/idvoretskyi/gitops-demo/demo-app:v1.0.1
   ```

2. **Get image digest**:
   ```bash
   docker inspect ghcr.io/idvoretskyi/gitops-demo/demo-app:v1.0.1 \
     --format='{{index .RepoDigests 0}}'
   ```

3. **Update deployment with digest**:
   ```yaml
   image: ghcr.io/idvoretskyi/gitops-demo/demo-app@sha256:abc123...
   ```

### For Local Development

1. **Build local image**:
   ```bash
   docker build -t gitops-demo-app:v1.0.1 demo-app/
   ```

2. **Use tag in deployment**:
   ```yaml
   image: gitops-demo-app:v1.0.1
   imagePullPolicy: Always
   ```

## CI/CD Pipeline Integration

### Enhanced Workflow (Automatic Digest Update)

```yaml
- name: Update image with digest in deployment
  run: |
    # Get the image digest from the pushed image
    IMAGE_DIGEST=$(docker inspect ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:main-${{ github.sha }} \
      --format='{{index .RepoDigests 0}}')
    
    # Update production deployment with digest
    sed -i "s|image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}@sha256:.*|image: ${IMAGE_DIGEST}|g" \
      apps/demo-app/deployment-production.yaml
```

## Security Benefits

### ✅ Digest Advantages
- **Immutable**: Content cannot change once deployed
- **Integrity**: Cryptographic verification of image content
- **Auditability**: Exact image version is always known
- **Security**: Prevents tag-based attacks

### ⚠️ Tag Limitations
- **Mutable**: Tag can point to different content
- **Unpredictable**: `latest` or version tags may change
- **Risk**: Potential for supply chain attacks

## Environment-Specific Deployments

### Development Environment
```bash
# Use local deployment with tags
kubectl apply -k apps/demo-app/
```

### Production Environment
```bash
# Use production deployment with digests
kubectl apply -f apps/demo-app/deployment-production.yaml
kubectl apply -f apps/demo-app/service.yaml
kubectl apply -f apps/demo-app/namespace.yaml
kubectl apply -f apps/demo-app/network-policy.yaml
```

## Verification Commands

### Check Current Image Reference
```bash
kubectl get deployment demo-app -n demo-app -o jsonpath='{.spec.template.spec.containers[0].image}'
```

### Verify Image Digest
```bash
# For registry images
docker inspect <image-reference> --format='{{index .RepoDigests 0}}'

# For local images
docker inspect <image-reference> --format='{{.Id}}'
```

## Compliance Status

| Environment | Image Reference | CKV_K8S_43 Compliance |
|-------------|----------------|------------------------|
| Local Dev   | Tag-based      | ❌ (For development only) |
| Production  | Digest-based   | ✅ (Fully compliant) |

## Next Steps

1. **For Production**: Use `deployment-production.yaml` with registry digests
2. **For CI/CD**: Implement automatic digest updates in pipeline
3. **For Security**: Always use digests in production environments
4. **For Development**: Tags are acceptable for local testing

This approach balances security compliance with development practicality.