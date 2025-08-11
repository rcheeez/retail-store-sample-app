# Branch Structure Fixes Applied

## Issues Identified and Fixed

Based on the README.md and BRANCHING_STRATEGY.md documentation, the following issues were identified and resolved:

### 🔧 **Issue 1: Branch Target Mismatch**
**Problem**: Individual ArgoCD applications were pointing to `main` branch instead of `gitops` branch
**Fix**: Updated all individual ArgoCD applications in gitops branch to use `targetRevision: gitops`

### 🔧 **Issue 2: Missing Umbrella Chart Application**
**Problem**: Main branch was missing the umbrella chart application (`retail-store-app.yaml`)
**Fix**: Created `argocd/applications/retail-store-app.yaml` pointing to `src/app/chart` with `targetRevision: main`

### 🔧 **Issue 3: GitHub Actions in Wrong Branch**
**Problem**: GitHub Actions workflow existed in main branch but should only be in gitops branch
**Fix**: 
- Removed `.github/workflows/` from main branch
- Restored complete GitHub Actions workflow in gitops branch

### 🔧 **Issue 4: Individual Apps in Main Branch**
**Problem**: Main branch had individual ArgoCD applications but should only have umbrella chart
**Fix**: Removed all individual application files from main branch, kept only `retail-store-app.yaml`

## Current Branch Structure (Fixed)

### 🌐 **Main Branch** (Public Application)
```
✅ ArgoCD: retail-store-app.yaml (umbrella chart)
✅ Target: src/app/chart
✅ Branch: main
✅ Images: Public ECR (stable versions)
✅ Workflows: None
✅ Purpose: Simple deployment, demos, learning
```

### 🏭 **GitOps Branch** (Production)
```
✅ ArgoCD: Individual applications (5 services)
✅ Target: src/{service}/chart
✅ Branch: gitops
✅ Images: Private ECR (auto-updated)
✅ Workflows: .github/workflows/deploy.yml
✅ Purpose: Production, automated CI/CD
```

## Deployment Instructions

### **For Public Application (Main Branch)**
```bash
# Use umbrella chart application
kubectl apply -f argocd/applications/retail-store-app.yaml -n argocd
```

### **For Production (GitOps Branch)**
```bash
# Use individual applications
kubectl apply -f argocd/applications/ -n argocd
```

### **Switching Between Branches**
```bash
# To switch from Production to Public Application
kubectl delete -f argocd/applications/ -n argocd  # Remove individual apps
git checkout main
kubectl apply -f argocd/applications/retail-store-app.yaml -n argocd

# To switch from Public Application to Production
kubectl delete application retail-store-app -n argocd  # Remove umbrella app
git checkout gitops
kubectl apply -f argocd/applications/ -n argocd  # Apply individual apps
```

## Verification

### **Main Branch Verification**
```bash
git checkout main
ls argocd/applications/  # Should show only retail-store-app.yaml
ls .github/  # Should not exist
```

### **GitOps Branch Verification**
```bash
git checkout gitops
ls argocd/applications/  # Should show 5 individual service applications
ls .github/workflows/  # Should show deploy.yml
grep "targetRevision: gitops" argocd/applications/*.yaml  # All should point to gitops
```

## Next Steps

1. **Push Changes**: Push both branches to remote repository
2. **Configure Secrets**: Set up GitHub Actions secrets for production workflow
3. **Deploy**: Choose appropriate branch based on your deployment needs
4. **Monitor**: Use ArgoCD UI to monitor application deployments

## GitHub Actions Secrets Required (GitOps Branch Only)

| Secret Name | Description |
|-------------|-------------|
| `AWS_ACCESS_KEY_ID` | AWS Access Key for ECR/EKS access |
| `AWS_SECRET_ACCESS_KEY` | AWS Secret Key |
| `AWS_REGION` | AWS Region (e.g., us-west-2) |
| `AWS_ACCOUNT_ID` | AWS Account ID for ECR URLs |

The branch structure now correctly implements the dual-branch GitOps strategy as documented in BRANCHING_STRATEGY.md.
