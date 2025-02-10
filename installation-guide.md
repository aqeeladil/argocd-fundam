## Installation Methods

ArgoCD can be installed in multiple ways, depending on your environment and requirements. The main methods include:

### 1. Using kubectl (Quick Start)

Best for: Development environments and quick deployments

**Advantages:**
- Fastest way to get started
- No additional tools required
- Works on any Kubernetes cluster

**Installation:**
```bash
# Create namespace
kubectl create namespace argocd

# Apply installation manifest
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**Verification:**
```bash
# Wait for all pods to be ready
kubectl wait --for=condition=Ready pod -l app.kubernetes.io/name=argocd-server -n argocd
```

### 2. Using Helm (Production Recommended)

Best for: Production environments and customized deployments

**Advantages:**
- Highly configurable through values.yaml
- Easy upgrades and rollbacks
- Version management
- Integrates well with CI/CD pipelines.
- Template-based configuration

**Installation:**
```bash
# Add ArgoCD Helm repository
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# Install ArgoCD (with default settings)
helm install argocd argo/argo-cd \
    --namespace argocd \
    --create-namespace

# Or install with custom values
helm install argocd argo/argo-cd \
    --namespace argocd \
    --create-namespace \
    -f values.yaml
```

**Example values.yaml:**
```yaml
server:
  extraArgs:
    - --insecure
  service:
    type: LoadBalancer

redis:
  resources:
    requests:
      memory: 128Mi
      cpu: 100m
```

### 3. Using Kustomize (Advanced)

Best for: GitOps workflows and environment-specific configurations

**Advantages:**
- Declarative configuration
- Environment overlays
- Resource patching capabilities
- Version control friendly

**Installation:**
```bash
# Clone the repository
git clone https://github.com/argoproj/argo-cd.git
cd argo-cd

# Apply using kustomize
kubectl apply -k manifests/cluster-install
```

**Example kustomization.yaml:**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - github.com/argoproj/argo-cd//manifests/cluster-install?ref=v2.9.0
patches:
  - path: patches/service-type.yaml
```

### 4. Using Terraform (Infrastructure as Code)

Best for: Enterprise environments and automated infrastructure provisioning

**Advantages:**
- Infrastructure as Code (IaC)
- Reproducible deployments
- State management
- Integration with other cloud resources

**Example Configuration:**
```hcl
resource "helm_release" "argocd" {
  name             = "argocd"
  repository       = "https://argoproj.github.io/argo-helm"
  chart            = "argo-cd"
  namespace        = "argocd"
  create_namespace = true
  
  set {
    name  = "server.service.type"
    value = "LoadBalancer"
  }
  
  set {
    name  = "server.ingress.enabled"
    value = "true"
  }
}
```

## Post-Installation Steps

1. **Access the ArgoCD UI:**
```bash
# Get the admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward the service
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

2. **Install ArgoCD CLI:**
```bash
# For Linux
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

## Troubleshooting

Common issues and solutions:
- If pods are not starting, check namespace events: `kubectl get events -n argocd`
- For connection issues, verify network policies
- For authentication problems, ensure secrets are properly created
