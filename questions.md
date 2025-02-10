## Interview Questions

## Onboarding GitOps into an Existing Kubernetes Environment:

A common challenge arises when an organization transitions to GitOps with pre-existing Kubernetes resources.

**Use Case**

- You have two applications already deployed in the Kubernetes cluster before adopting GitOps. These applications were not deployed via the GitOps pipeline.

- By default, GitOps tools may try to remove resources not defined in the Git repository, treating them as "drift" from the desired state.

- This can cause accidental deletion of critical applications.

**Solution**

- **Manually Import Existing Resources:** Export the YAML definitions of current applications and add them to the Git repository.

- **Selective Syncing:** Use GitOps configurations to exclude unmanaged namespaces or applications from reconciliation.

- **Gradual Onboarding:** Begin with a subset of resources, testing and validating the reconciliation behavior before full-scale adoption.

## GitOps Handling Admission Controller Changes:

When an admission controller modifies pod configurations (e.g., adding resource requests, limits, or annotations), GitOps tools may attempt to revert these changes to align with the repository. This can lead to issues such as deployment failures in clusters with mandatory policies.

**How GitOps Handles This ?**

- **Pre-define Admission Controller Requirements in Git:** Add required fields (e.g., resource requests, limits, labels) to the pod.yaml in the Git repository to prevent discrepancies.

- **Exclude Specific Fields from Reconciliation:** Use GitOps annotations to ignore changes made by admission controllers for certain fields.

- **Leverage Customization Tools:** Tools like Kustomize or Helm can merge dynamic admission controller configurations with the desired state during deployment.

By aligning the Git repository with mandatory cluster policies and using tools to manage overrides, GitOps ensures smoother reconciliation and compliance.

## Non-opinionated nature of ArgoCD

1. ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes. It focuses on managing Kubernetes applications using Git repositories as the source of truth, but it leaves decisions like the repository structure, deployment strategies, and management styles up to the user. The non-opinionated nature gives teams the freedom to customize how they use ArgoCD while still leveraginits powerful features for managing and deploying Kubernetes resources.

2. For example, a software development team works with a microservices-based application that needs tbe deployed across multiple environments, such as Development, Staging, and Production. Each environment has slightly different configurations (e.g., database connection strings, feature flagsetc.), and the team wants to ensure that all changes to the application are managed in a Git repository, ensuring traceability and automated deployment.

3. The team adopts a GitOps approach, where they store the Kubernetes manifest files for the pplication in a Git repository. Each environment has its own folder in the Git repo, containing thnecessary Kubernetes configurations (e.g., Deployments, Services, ConfigMaps).

    ```arduino
    ├── dev/
    │   ├── app-deployment.yaml
    │   ├── app-service.yaml
    │   └── config-map.yaml
    ├── staging/
    │   ├── app-deployment.yaml
    │   ├── app-service.yaml
    │   └── config-map.yaml
    └── production/
        ├── app-deployment.yaml
        ├── app-service.yaml
        └── config-map.yaml
    ```

4. The team sets up ArgoCD to manage the deployment of the application. ArgoCD is installed in each kubernetes cluster (Development, Staging, Production), and the application is managed using ArgoCD'declarative approach. Each environment is treated as a separate ArgoCD Application, pointing to the appropriate folder in the Git repository.

5. ArgoCD Applications are created for each environment. The Git repository URL and a directory for each environment are specified in the application configuration. For example, the ArgoCD Application for the Development environment might look like this:

    ```yaml
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: my-app-dev
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/my-app-repo
        targetRevision: HEAD
        path: dev
      destination:
        server: https://kubernetes.default.svc
        namespace: dev
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
    ```

6. ArgoCD continuously monitors the Git repository for changes. When a change is committed (e.g., a new version of the application or a configuration update), ArgoCD automatically syncs the Kubernetes resources in the cluster with the desired state defined in the Git repository. The deployment to Staging and Production environments would follow a similar approach but pointing to the staging/ and production/ directories, respectively.

7. ArgoCD's self-healing feature ensures that if any Kubernetes resource is manually modified or goes out of sync with the Git repository, it will automatically be reverted to the correct state as defined in Git. If an issue arises in any environment, the team can quickly roll back to a previous version by simply reverting the changes in the Git repository and allowing ArgoCD to sync the environment back to the previous state.

## Manifests vs Helm vs Kustomize:

When managing Kubernetes resources, there are three common approaches: manifests, Helm, and Kustomize. Each has its strengths and weaknesses, and the best choice depends on your use case.

**When to Use What?**
- *Manifests:* For straight-forward deployments or when simplicity is a priority.
- *Helm:* For complex, reusable deployments requiring dynamic values and sharing across teams.
- *Kustomize:* For customizing YAML files for different environments in a Kubernetes-native way without needing Helm’s templating.

**Manifests (Raw YAML Files):**
- Directly written Kubernetes YAML files (e.g., Deployments, Services, ConfigMaps).
- No abstraction; you manually define and maintain all configurations.
- Easy to understand and manage for small-scale projects. No extra tools or dependencies needed.
- Full control over Kubernetes resources without any abstraction.
- No additional layers of complexity; the YAML you write is what Kubernetes applies.
- High duplication across environments (e.g., different namespaces or resource limits).
- Manual changes to multiple files can lead to inconsistencies.
- Not ideal for managing complex deployments with multiple environments or dynamic values.

**Helm:**
- A package manager for Kubernetes, similar to APT or Yum.
- Uses charts, which are templates with values to generate Kubernetes manifests dynamically.
- Charts can be shared, versioned, and reused.
- Great for deploying standardized applications like MySQL or NGINX.
- Supports dynamic values using ```values.yaml``` to customize deployments.
- Thousands of pre-built charts available on repositories like Helm Hub.
- Tracks deployments as releases, making upgrades, rollbacks, and uninstallations straightforward.
- Complexity: Learning curve for Helm syntax and templating.
- Debugging templates can be challenging.
- Overhead: Adds an abstraction layer, which might be unnecessary for simple deployments.
- Tight Coupling: Changes to Helm templating can lock you into Helm-specific structures.

**Kustomize:**
- A Kubernetes-native configuration management tool.
- Works by layering or "patching" base YAML files to customize configurations for different environments.
- Declarative and Kubernetes-native: Integrated with ```kubectl``` (can be used with ```kubectl apply -k```).
- No Templates: Focuses on YAML overlays instead of template-based rendering.
- Environment-specific Customization: Uses overlays to adjust configurations without duplicating the base YAML.
- Simpler than Helm: Less abstraction, making it easier to debug.
- Less Flexible: Lacks advanced templating capabilities compared to Helm.
- Reusability: Limited to patching YAML; less powerful for creating reusable packages.
- Steeper Learning Curve for Beginners: Managing overlays and bases can feel unintuitive initially.




