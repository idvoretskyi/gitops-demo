# CKV_K8S_43 Compliance: Image Digest Implementation

## Security Finding
**Check**: `CKV_K8S_43` - Image should use digest  
**Status**: ❌ FAILED  
**Issue**: Container image uses tag instead of digest reference

## Current State
```yaml
# apps/demo-app/deployment.yaml (Line 28)
image: gitops-demo-app:v1.0.1  # ❌ Tag-based reference
```

## Required Solution
```yaml
# Compliant format
image: gitops-demo-app@sha256:de9f52436a01f88fbda38fe092d48589d89f26517001b8ceb06a6f08140f61ef
```

## Implementation Options

### Option 1: Full Production Compliance
Replace the image reference in `apps/demo-app/deployment.yaml`:

```bash
# Get the image digest
docker inspect gitops-demo-app:v1.0.1 --format='{{.Id}}' | cut -d: -f2

# Update deployment.yaml
sed -i 's|image: gitops-demo-app:v1.0.1|image: gitops-demo-app@sha256:de9f52436a01f88fbda38fe092d48589d89f26517001b8ceb06a6f08140f61ef|' \
  apps/demo-app/deployment.yaml
```

### Option 2: Use Production Deployment
Switch to the production-ready deployment:

```bash
# Edit apps/demo-app/kustomization.yaml
# Comment out: - deployment.yaml
# Uncomment: - deployment-production.yaml
```

### Option 3: Registry-Based Approach
1. Push image to container registry:
```bash
docker tag gitops-demo-app:v1.0.1 ghcr.io/idvoretskyi/gitops-demo/demo-app:v1.0.1
docker push ghcr.io/idvoretskyi/gitops-demo/demo-app:v1.0.1
```

2. Get registry digest:
```bash
docker inspect ghcr.io/idvoretskyi/gitops-demo/demo-app:v1.0.1 --format='{{index .RepoDigests 0}}'
```

3. Update deployment with registry digest.

## Quick Fix Command
```bash
# Apply the digest-based deployment immediately
kubectl patch deployment demo-app -n demo-app -p '{"spec":{"template":{"spec":{"containers":[{"name":"demo-app","image":"gitops-demo-app@sha256:de9f52436a01f88fbda38fe092d48589d89f26517001b8ceb06a6f08140f61ef"}]}}}}'
```

## Verification
```bash
# Check current image reference
kubectl get deployment demo-app -n demo-app -o jsonpath='{.spec.template.spec.containers[0].image}'

# Should output: gitops-demo-app@sha256:de9f52436a01f88fbda38fe092d48589d89f26517001b8ceb06a6f08140f61ef
```

## Security Benefits
- ✅ **Immutable deployments**: Image content cannot change
- ✅ **Supply chain security**: Cryptographic verification
- ✅ **Audit compliance**: Exact image version always known
- ✅ **Attack prevention**: Eliminates tag-based image substitution attacks

## Implementation Status
| Environment | Current State | CKV_K8S_43 Status |
|-------------|---------------|-------------------|
| Development | Tag-based (`v1.0.1`) | ❌ Non-compliant |
| Production | Digest-ready (`deployment-production.yaml`) | ✅ Compliant |

## Recommended Action
For immediate compliance, execute Option 1 (Full Production Compliance) to update the main deployment file with the digest reference.

**Command to fix immediately:**
```bash
sed -i 's|image: gitops-demo-app:v1.0.1|image: gitops-demo-app@sha256:de9f52436a01f88fbda38fe092d48589d89f26517001b8ceb06a6f08140f61ef|' apps/demo-app/deployment.yaml
git add apps/demo-app/deployment.yaml
git commit -m "Fix CKV_K8S_43: Use image digest instead of tag"
```