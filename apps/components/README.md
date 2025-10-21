# GitOps Components

Optional components for cluster applications. Each cluster selects which components to deploy.

## Component Structure

```
components/
├── <component-name>/
│   ├── <component-name>.yaml   # ArgoCD Application
│   └── kustomization.yaml      # Component wrapper
```

## Available Components

### OpenShift GitOps Operator

| Component | Description | Usage | InstallPlan Approval |
|-----------|-------------|-------|---------------------|
| `openshift-gitops-custom` | Custom ArgoCD config | ✅ Include in cluster | N/A |
| `openshift-gitops-self-managed` | Self-management | ✅ Include in cluster | Manual for updates |
| `openshift-gitops-with-auto-approval` | Self-management + auto-approval | ✅ Include in cluster | CronJob auto-approves |

**Deployment Pattern:**
1. **Manual Bootstrap**: `oc apply -k k8s-apps-repo/ocp/openshift-gitops-operator/base/`
2. **App-of-Apps**: `oc apply -f bootstrap/dev-cluster.yaml`
3. **Self-Management**: ArgoCD manages operator via chosen component

**Choose ONE self-management option:**
- `openshift-gitops-self-managed` - Production (manual approval)
- `openshift-gitops-with-auto-approval` - Homelab (automatic approval)

### GitLab Runner Operator

| Component | Description |
|-----------|-------------|
| `gitlab-runner-operator` | CI/CD runner management |

## Repository Separation

### k8s-apps-repo (Source Manifests)
```
ocp/openshift-gitops-operator/
├── base/                    # Bootstrap resources
└── overlays/
    ├── custom/              # ArgoCD configuration
    ├── self-managed/        # Operator resources (no Job)
    └── with-auto-approval/  # Operator + CronJob
```

### k8s-gitops-repo (ArgoCD Applications)
```
apps/components/
├── openshift-gitops-custom/             → overlays/custom/
├── openshift-gitops-self-managed/       → overlays/self-managed/
├── openshift-gitops-with-auto-approval/ → overlays/with-auto-approval/
└── gitlab-runner-operator/              → ocp/gitlab-runner-operator/base/
```

**Separation of Concerns:**
- `k8s-apps-repo`: WHAT to deploy (manifests)
- `k8s-gitops-repo`: HOW ArgoCD deploys (Applications)

## Usage in Cluster Config

```yaml
# apps/dev-cluster/kustomization.yaml
components:
  # Custom ArgoCD configuration
  - ../components/openshift-gitops-custom
  
  # Choose ONE self-management mode:
  - ../components/openshift-gitops-self-managed          # Manual
  # OR
  # - ../components/openshift-gitops-with-auto-approval  # Automatic
  
  # Other operators
  - ../components/gitlab-runner-operator
```

**Bootstrap Workflow:**
1. **Manual bootstrap**: `oc apply -k k8s-apps-repo/ocp/openshift-gitops-operator/base/`
2. **App-of-Apps**: `oc apply -f bootstrap/dev-cluster.yaml`
3. **GitOps manages GitOps**: ArgoCD self-manages via chosen component

## Adding New Components

1. Create component directory:
   ```bash
   mkdir -p apps/components/my-app
   ```

2. Create ArgoCD Application:
   ```yaml
   # apps/components/my-app/my-app.yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: my-app
   spec:
     source:
       repoURL: <k8s-apps-repo-url>
       path: <path-to-manifests>
     destination:
       namespace: my-app
   ```

3. Create component kustomization:
   ```yaml
   # apps/components/my-app/kustomization.yaml
   apiVersion: kustomize.config.k8s.io/v1alpha1
   kind: Component
   resources:
     - my-app.yaml
   ```

4. Add to cluster configs as needed

