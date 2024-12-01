# Continuous Delivery | GitOps | ArgoCD 

## What is GitOps?

- GitOps is a modern approach to managing infrastructure and application deployments. 
- It uses Git as the single source of truth for declarative infrastructure and applications, enabling continuous delivery and operational workflows. 
- It bridges the gap between Continuous Integration (CI) and Continuous Deployment (CD), ensuring that deployment processes are version-controlled and auditable.
- With GitOps, you store the desired state of your system in a Git repository, and a GitOps tool ensures that the actual system state matches the desired state through automated synchronization.
- Argo CD and Flux are two popular GitOps controllers, mainly used with Kubernetes.
- GitOps principles are not exclusive to Kubernetes and can be extended to other infrastructures.
- While GitOps doesn't strictly require Git, its tooling and workflows are deeply aligned with Git's capabilities. Using a different VCS for GitOps is possible but might involve extra effort and customizations.

______________________________________________________________________________________________

## Key Principles of GitOps

**Declarative Configuration:**
- All system states (infrastructure, applications, etc.) are defined declaratively in code.
- Common tools: YAML manifests for Kubernetes, Terraform for infrastructure.

**Git as the Source of Truth:**
- The desired state of the system is version-controlled in Git.
- Git's version history serves as a record of changes.

**Automation and Reconciliation:**
- A GitOps operator (e.g., Flux, ArgoCD) watches the Git repository and automatically applies changes to the system.
- It continuously reconciles the actual state with the desired state.

**Security & Self-Healing Systems:**
- If the actual state deviates (e.g., manual changes or system failures), the GitOps operator brings it back to the desired state.
- Unauthorized changes to the cluster are overridden, as the system only accepts changes from Git.

___________________________________________________________________________________________

## ArgoCD Architecture

Argo CD consists of several microservices working together:

**Repo Server:**
- Interacts with Git repositories to fetch application manifests (desired state).
- Monitors changes in the Git repository.
- Performs manifest generation for configurations like Helm charts and Kustomize.

**Application Controller (Reconciliation Engine):**
- Continuously monitors the desired state defined in the Git repository and the live state of the Kubernetes cluster.
- Identifies drift (discrepancies) between the two states.
- Synchronizes the cluster state with the desired state by applying necessary changes to the cluster.

**Application CRD (Custom Resource Definition):**
- Defines an Argo CD application (Source, Target, Sync Policies)
  - Source: The Git repository and path containing the desired application state.
  - Target: The Kubernetes cluster and namespace where the application should run.
  - Sync policies: Automatic or manual synchronization options.
- Enables Kubernetes-native management of Argo CD applications.

**ApplicationSet:**
- An ApplicationSet is a custom resource in ArgoCD that extends the functionality of the ArgoCD Application resource. 
- While a typical ArgoCD Application is used to define a single deployment, an ApplicationSet allows you to define a set of applications, making it easier to manage applications that are similar but vary in certain configurations, like different environments (e.g., dev, staging, prod).

**API Server:**
- A Kubernetes-native service that exposes the Argo CD user interface (UI) and REST APIs.
- Handles user authentication and authorization (supports SSO via OIDC).
- Serves as the interaction layer between users and Argo CD components.

**Dex:**
- Provides single sign-on (SSO) and authentication integration with external identity providers like LDAP, GitHub, Google, or SAML.

**Redis:**
- Used for caching data, ensuring stateful operations and recovery in case of service restarts.

________________________________________________________________________________________________

## Common Deployment Models

Argo CD can be set up in different architectures to suit various needs. The two common deployment models are hub-spoke and standalone. Here's a comparison of the two:

#### 1. Hub-Spoke Architecture

- In the hub-spoke model, Argo CD is installed in a central "hub" cluster that manages multiple "spoke" clusters. 
- Each spoke cluster is connected to the hub cluster via a mechanism like Argo CD's Cluster API integration.
- This is particularly useful in environments where you have multiple Kubernetes clusters and want to centralize control and management.
- It's best for enterprises with complex Kubernetes setups or multi-region deployments.
- Easier governance and monitoring.
- Single point of failure: If the hub cluster becomes unavailable, managing the spoke clusters can be challenging.
- Increased complexity in setting up the communication between clusters.
- Requires networking and permissions to allow access between the hub and spoke clusters.

#### 2. Standalone Architecture

- In a standalone model, Argo CD is deployed within each cluster that you want to manage. Every Argo CD instance manages the resources within that cluster independently. 
- This setup is more decentralized compared to the hub-spoke model.
- Better for smaller setups or cases where each cluster operates independently. It's simpler and provides better fault isolation.
- High availability: If one Argo CD instance fails, it doesn't affect others.
- Simplicity: Each instance is responsible for a single cluster, reducing cross-cluster management complexity.
- Lower network dependencies since each cluster is autonomous.
- Harder to manage at scale: If you have many clusters, it becomes difficult to manage multiple Argo CD instances.
- Less centralized control: Each instance operates independently, so managing applications across clusters requires more coordination.
- Duplicate configuration: If you want the same applications in multiple clusters, you may need to maintain separate configurations for each Argo CD instance.

________________________________________________________________________________________________

## GitOps Workflow

- Store Infrastructure configurations (e.g., Kubernetes YAML manifests (node.yaml, deploy.yaml) or Helm charts, or Kustomize files or Terraform files) in a Git repository.
- Set up a GitOps operator (e.g., Flux or ArgoCD) in your cluster.
- Developer makes changes to the manifests (e.g., updates an image version).
- Creates a pull request for review.
- The pull request is reviewed and merged into the main branch.
- The GitOps operator detects the change and applies it to the target system automatically.
- The operator continuously monitors and ensures that the system matches the desired state.
- If someone directly edits the cluster without Git, the GitOps controller reverts the change.

_______________________________________________________________________________________________

## Problems GitOps Solves:

- **No tracking of changes:** When someone updates Kubernetes resources (like a node configuration) manually, there is no built-in version control or audit trail. It's hard to answer questions like:
  - Who made this change?
  - What change was made 10 days ago?
- **No versioning or immutability:** Changes made using tools like ```kubectl``` or scripts cannot be rolled back easily, and there's no history of previous configurations.
- **Lack of visibility:** Without a system like GitOps, it's challenging to determine the state of infrastructure at a given time.

**Example:** Imagine someone modifies a Kubernetes cluster directly using ```kubectl apply``` or a script. If issues arise later, there’s no easy way to track or roll back those changes. This lack of tracking and auditability poses risks.

_________________________________________________________________________________________

## Real-World Use Cases:

- **Continuous Delivery in Kubernetes:** Automating updates for microservices hosted on Kubernetes clusters.
- **Multi-Environment Management:** Managing separate repositories for dev, staging, and production environments.
- **Disaster Recovery:** Quick rollbacks to stable states in case of failed deployments.
- **Compliance and Auditing:** Git serves as an audit log for all cluster changes.

________________________________________________________________________________________

## GitOps Tools:

- **FluxCD:** A lightweight GitOps tool for Kubernetes. Watches a Git repository and applies changes automatically.
- **ArgoCD:** A more feature-rich GitOps tool, ideal for complex Kubernetes environments. Provides a web interface and integrates well with CI/CD pipelines.
- **Jenkins X:** Combines CI/CD capabilities with GitOps.
- **Spinnaker:** Primarily a deployment tool, not entirely GitOps-focused.

_________________________________________________________________________________________

## Applications Beyond Kubernetes

Although current GitOps tools primarily target Kubernetes, the principles can be applied to manage:
- Docker environments.
- AWS infrastructure.
- Other cloud resources.
____________________________________________________________________________________________

## GitOps for Applications and Infrastructure:

GitOps is not limited to application deployments. It is equally useful for managing infrastructure. For example:
- Application Delivery: Deploying services like web applications or microservices.
- Infrastructure Delivery: Managing Kubernetes nodes, admission controllers, namespaces, etc.

**Example:**
- Application Delivery: Deploying a web app using a deployment.yaml manifest.
- Infrastructure Delivery: Modifying Kubernetes node configurations using a ```node.yaml``` manifest.
_________________________________________________________________________________________

## Traditional vs GitOps-Approach to CICD Implementation

#### Traditional Approach:

1. **Continuous Integration:** Jenkins -> Build & Unit Testing -> Static Code Analysis -> SAST -> DAST -> End-to-end Testing or Regression Testing -> Docker Image -> Docker Image scanning (using Trivy or Docker Scout) ->.
2. **Continuous Delivery:** -> Shell Script or Ansible Playbook (to create k8s manifests) -> Deployment to k8s.

Here, the scripts or ansible playbooks are not capable of identifying any manual changes in the k8s cluster. For the Continuous-Integration part, we have Git as the single source of truth. But for the Continuous-Delivery part, we have no source of truth to verify any manual changes in the cluster. 

#### GitOps-Approach:

1. **Continuous Integration:** Jenkins -> Build & Unit Testing -> Static Code Analysis -> SAST -> DAST -> End-to-end Testing or Regression Testing -> Docker Image -> Docker Image scanning (using Trivy or Docker Scout) ->.
2. **Continuous Delivery:** ->  Infrastructure Configurations stored in Git repo (to create k8s manifests) -> GitOps Tool like ArgoCd (to track changes in the git repo and apply those changes to k8s cluster) -> Deployment to k8s.

Here Git is the only source of truth for both CI & CD stages.

_________________________________________________________________________________________

## Onboarding GitOps into an Existing Kubernetes Environment:

- A common challenge arises when an organization transitions to GitOps with pre-existing Kubernetes resources.
- You have two applications already deployed in the Kubernetes cluster before adopting GitOps. These applications were not deployed via the GitOps pipeline.
- By default, GitOps tools may try to remove resources not defined in the Git repository, treating them as "drift" from the desired state.
- This can cause accidental deletion of critical applications.
- **Manually Import Existing Resources:** Export the YAML definitions of current applications and add them to the Git repository.
- **Selective Syncing:** Use GitOps configurations to exclude unmanaged namespaces or applications from reconciliation.
- **Gradual Onboarding:** Begin with a subset of resources, testing and validating the reconciliation behavior before full-scale adoption.

_______________________________________________________________________________________________

## GitOps Handling Admission Controller Changes:

When an admission controller modifies pod configurations (e.g., adding resource requests, limits, or annotations), GitOps tools may attempt to revert these changes to align with the repository. This can lead to issues such as deployment failures in clusters with mandatory policies.

**How GitOps Handles This ?**
- **Pre-define Admission Controller Requirements in Git:** Add required fields (e.g., resource requests, limits, labels) to the pod.yaml in the Git repository to prevent discrepancies.
- **Exclude Specific Fields from Reconciliation:** Use GitOps annotations to ignore changes made by admission controllers for certain fields.
- **Leverage Customization Tools:** Tools like Kustomize or Helm can merge dynamic admission controller configurations with the desired state during deployment.

By aligning the Git repository with mandatory cluster policies and using tools to manage overrides, GitOps ensures smoother reconciliation and compliance.

_______________________________________________________________________________________________

## Non-opinionated nature of ArgoCD

    1. ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes. It focuses on managing Kubernetes applications using Git repositories as the source of truth, but it leaves decisions like the repository structure, deployment strategies, and management styles up to the user. This non-opinionated nature gives teams the freedom to customize how they use ArgoCD while still leveraging its powerful features for managing and deploying Kubernetes resources.
    2. For example, a software development team works with a microservices-based application that needs to be deployed across multiple environments, such as Development, Staging, and Production. Each environment has slightly different configurations (e.g., database connection strings, feature flags, etc.), and the team wants to ensure that all changes to the application are managed in a Git repository, ensuring traceability and automated deployment.
    3. The team adopts a GitOps approach, where they store the Kubernetes manifest files for the application in a Git repository. Each environment has its own folder in the Git repo, containing the necessary Kubernetes configurations (e.g., Deployments, Services, ConfigMaps).
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
    4. The team sets up ArgoCD to manage the deployment of the application. ArgoCD is installed in each Kubernetes cluster (Development, Staging, Production), and the application is managed using ArgoCD's declarative approach. Each environment is treated as a separate ArgoCD Application, pointing to the appropriate folder in the Git repository.
    5. ArgoCD Applications are created for each environment. The Git repository URL and the specific directory for each environment are specified in the application configuration.
    For example, the ArgoCD Application for the Development environment might look like this:
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

_______________________________________________________________________________________________

## Manifests vs Helm vs Kustomize:

When managing Kubernetes resources, there are three common approaches: manifests, Helm, and Kustomize. Each has its strengths and weaknesses, and the best choice depends on your use case.

**When to Use What?**
- *Manifests:* For straightforward deployments or when simplicity is a priority.
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




