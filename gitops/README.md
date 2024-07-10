# GitOps Explained! Episode 1
## GitOps Deployments
GitOps is a modern operational framework for managing infrastructure and applications, using Git as the single source of truth. It applies the principles of DevOps to infrastructure management, enabling a declarative approach to operations and continuous delivery. The core idea is to define the desired state of the system in a Git repository and use automated processes to ensure the actual state matches the desired state.

### GitOps Principles (v1.0.0, as defined by OpenGitOps)
1. **Declarative**: A system managed by GitOps must have its desired state expressed declaratively.

2. **Versioned and Immutable**: Desired state is stored in a way that enforces immutability, versioning and retains a complete version history.

3. **Pulled Automatically**: Software agents automatically pull the desired state declarations from the source.

4. **Continuously Reconciled**: Software agents continuously observe actual system state and attempt to apply the desired state.

### Argo CD
Argo CD is a continuous delivery controller for Kubernetes. Argo CD enables Kubernetes users to implement GitOps principles.

Argo CD introduces a Custom Resource Definition to Kubernetes call an `Application`. The resource defines the source for the Kubernetes manifests you want to deploy, the destination to deliver the resources to, and the sync policy to define how to apply the manifests to the cluster.

### GitOps Example with Argo CD.
To deploy the `Deployment` and `Service` resources for `my-app` to Kubernetes, you will create an Argo CD `Application` that points to the `my-app` directory of your Git repository.

Here's an example for this repo:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  destination:
    namespace: my-app
    name: in-cluster
  project: default
  source:
    path: my-app/
    repoURL: https://github.com/morey-tech/gitops-explained-1.git
    targetRevision: main
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```