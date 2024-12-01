## Different ways to install ArgoCD

ArgoCD can be installed in multiple ways, depending on your environment and requirements. The main methods include:

### 1. Using kubectl (Manifest Installation)

- Download the YAML manifests for ArgoCD and apply them directly to your Kubernetes cluster.
- Simple and quick.
- No additional tools required.
- Limited customization during installation.

Example:
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/  install.yaml
```

### 2. Using Helm (Recommended for Customization)

- Install ArgoCD using the Helm package manager.
- Highly customizable.
- Integrates well with CI/CD pipelines.
- Requires Helm installed.

Example:
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd -n argocd --create-namespace
```

### 3. Using Kustomize

- Clone the ArgoCD repository and customize the installation using kustomize.
- Ideal for advanced users needing a declarative setup.
- Slightly more complex than ```kubectl```.

Example:
```bash
git clone https://github.com/argoproj/argo-cd.git
cd argo-cd
kubectl apply -k overlays/stable
```

### 4. Terraform

- Use tools like Terraform to provision ArgoCD alongside other Kubernetes resources
- Infrastructure as Code (IaC) for better repeatability.
- Requires Terraform knowledge.

Example:
```hcl
resource "helm_release" "argocd" {
  name       = "argocd"
  repository = "https://argoproj.github.io/argo-helm"
  chart      = "argo-cd"
}
```
