# GitOps CI/CD Pipeline

A complete GitOps workflow demonstrating GitHub Actions, Docker builds, security scanning with Trivy, and Argo CD deployments to Kubernetes.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        GitHub                                │
│                                                              │
│  ┌──────────────┐    ┌──────────────────┐                  │
│  │   Push/PR    │───▶│ GitHub Actions   │                  │
│  └──────────────┘    │  - Build         │                  │
│                      │  - Test          │                  │
│                      │  - Docker Build  │                  │
│                      │  - Trivy Scan    │                  │
│                      │  - Push Image    │                  │
│                      └────────┬─────────┘                  │
│                               │                            │
│                      ┌────────▼─────────┐                   │
│                      │  Docker Registry │                   │
│                      │  (GHCR/Docker Hub)│                  │
│                      └────────┬─────────┘                   │
│                               │                            │
│                      ┌────────▼─────────┐                   │
│                      │   Update Manifest│                   │
│                      │   (GitOps Repo)  │                   │
│                      └────────┬─────────┘                   │
│                               │                            │
│                      ┌────────▼─────────┐                   │
│                      │     Argo CD      │                   │
│                      │  (Sync to K8s)    │                   │
│                      └────────┬─────────┘                   │
│                               │                            │
│                      ┌────────▼─────────┐                   │
│                      │   Kubernetes     │                   │
│                      │  (k3d Cluster)   │                   │
│                      └──────────────────┘                   │
└─────────────────────────────────────────────────────────────┘
```

## Prerequisites

- GitHub account with repository
- Docker installed locally
- k3d Kubernetes cluster
- Argo CD installed
- kubectl configured

## Setup

### 1. Fork/Clone Repository
```bash
git clone <your-repo-url>
cd gitops-cicd
```

### 2. Configure GitHub Secrets
Add these secrets to your GitHub repository:
- `DOCKER_USERNAME`: Your Docker Hub username
- `DOCKER_PASSWORD`: Your Docker Hub password/token
- `ARGOCD_SERVER`: Argo CD server address
- `ARGOCD_AUTH_TOKEN`: Argo CD auth token

### 3. Install Argo CD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 4. Access Argo CD UI
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Open http://localhost:8080
# Default password: argocd-server (first login, change it)
```

### 5. Create Argo CD Application
```bash
kubectl apply -f argocd/application.yaml
```

## Workflow

1. **Developer pushes code** to GitHub
2. **GitHub Actions triggers**:
   - Builds Docker image
   - Runs Trivy security scan
   - Pushes image to registry
   - Updates Kubernetes manifests with new image tag
3. **Argo CD detects** manifest changes
4. **Argo CD syncs** changes to Kubernetes cluster
5. **Application deploys** automatically

## Components

### GitHub Actions
- `.github/workflows/ci-cd.yaml`: Main CI/CD pipeline
- Automated testing, building, and deployment

### Docker
- `Dockerfile`: Multi-stage build for optimized images
- Security best practices implemented

### Trivy
- Container vulnerability scanning
- Integrated into CI/CD pipeline
- Blocks deployment on high-severity vulnerabilities

### Argo CD
- GitOps operator for Kubernetes
- Continuous synchronization
- Rollback capabilities
- Application health monitoring

## Verification

```bash
# Check GitHub Actions workflow
# Visit: https://github.com/<username>/<repo>/actions

# Check Argo CD application status
argocd app get sample-app

# Check Kubernetes deployment
kubectl get deployments -n default

# Check pods
kubectl get pods -n default
```

## Security Features

- **Trivy Scanning**: Automated vulnerability detection
- **Image Signing**: Content trust (optional)
- **RBAC**: Argo CD with restricted permissions
- **Secrets Management**: Kubernetes secrets for sensitive data
- **Policy Enforcement**: OPA Gatekeeper (optional extension)
