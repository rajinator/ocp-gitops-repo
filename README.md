# OCP GitOps Repository

GitOps App Management using ArgoCD App of Apps pattern with Kustomize components.

Assisted by: Cursor and claude-4.5-sonnet

## Repository Structure

```
ocp-gitops-repo/
├── apps/
│   ├── components/            # Optional app components (à la carte)
│   │   ├── openshift-gitops-custom/
│   │   ├── openshift-gitops-self-managed/
│   │   ├── openshift-gitops-with-auto-approval/
│   │   └── gitlab-runner-operator/
│   └── dev-cluster/           # dev-cluster
│       └── kustomization.yaml
└── bootstrap/
    └── dev-cluster.yaml      # Bootstrap app-of-apps for dev-cluster
```

## How It Works

### Components (Optional Apps)

Each app is a Kustomize component - **all apps are opt-in**:

```yaml
# apps/dev-cluster/kustomization.yaml
components:
  - ../components/openshift-gitops-custom         # ✅ Include
  - ../components/openshift-gitops-self-managed   # ✅ Include
  - ../components/gitlab-runner-operator          # ✅ Include
```

### Per-Cluster Selection

Each cluster chooses exactly which apps to deploy:

```yaml
# apps/dev-cluster/kustomization.yaml
components:
  - ../components/openshift-gitops-custom
  - ../components/openshift-gitops-self-managed
  - ../components/gitlab-runner-operator
```

## Quick Start (dev-cluster)

```bash
# 1. Manual Bootstrap: Deploy OpenShift GitOps operator
oc apply -k ../k8s-apps-repo/ocp/openshift-gitops-operator/base/
oc apply -k ../k8s-apps-repo/ocp/openshift-gitops-operator/approval/bootstrap/

# 2. Wait for operator ready
oc wait --for=condition=Ready pod -l name=openshift-gitops-operator -n openshift-gitops-operator --timeout=300s

# 3. Wait for ArgoCD instance to be ready
oc wait --for=condition=Ready argocd/openshift-gitops -n openshift-gitops --timeout=300s

# 4. Bootstrap App-of-Apps (includes self-management)
oc apply -f bootstrap/dev-cluster.yaml
```

**Note:** The App-of-Apps includes self-management, so GitOps will manage itself going forward.

**InstallPlan Approval:** Other operators (like gitlab-runner) will have pending InstallPlans after initial deployment. Either:
- Pre-bootstrap them manually (see k8s-apps-repo operator docs)
- Manually approve their InstallPlans after ArgoCD deploys them
- Use the CronJob overlay for automatic approval (homelab use)

## ArgoCD Access

```bash
# Get route
oc get route openshift-gitops-server -n openshift-gitops

# Get admin password
oc get secret openshift-gitops-cluster -n openshift-gitops -o jsonpath='{.data.admin\.password}' | base64 -d
```

## Available Apps

- **openshift-gitops-custom** - Custom ArgoCD config with version-pinning aware health checks
- **openshift-gitops-self-managed** - Operator self-management (manual approval)
- **openshift-gitops-with-auto-approval** - Operator self-management (auto approval via CronJob)
- **cert-manager-operator** - Certificate management with version pinning
- **gitlab-runner-operator** - GitLab CI/CD runner management

## Add New Cluster

```bash
# 1. Copy and customize cluster config
cp -r apps/dev-cluster apps/prod-cluster
vim apps/prod-cluster/kustomization.yaml
# - Update labels
# - Choose components

# 2. Copy and customize bootstrap
cp bootstrap/dev-cluster.yaml bootstrap/prod-cluster.yaml
vim bootstrap/prod-cluster.yaml
# - Update name: prod-cluster-apps
# - Update labels.cluster: prod-cluster
# - Update source.path: apps/prod-cluster

# 3. Deploy
oc apply -f bootstrap/prod-cluster.yaml
```

## Add New App Component

```bash
# 1. Create component directory
mkdir -p apps/components/my-app
cd apps/components/my-app

# 2. Add ArgoCD Application manifest
cat > my-app.yaml <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: openshift-gitops
spec:
  source:
    repoURL: <repo-url>
    path: <path>
  destination:
    namespace: my-app
  syncPolicy:
    automated:
      prune: false
      selfHeal: true
EOF

# 3. Create component kustomization
cat > kustomization.yaml <<EOF
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component
resources:
  - my-app.yaml
EOF

# 4. Add to cluster config
vim apps/dev-cluster/kustomization.yaml
# Add: - ../components/my-app
```

## Notes

### Repository References
- **k8s-apps-repo**: Application manifests (source)
- **ocp-gitops-repo**: ArgoCD Applications (this repo)

### Design Principles
- **Optional by Default**: No app is forced on any cluster
- **Declarative**: All configs in Git
- **Composable**: Mix and match components per cluster
- **Patchable**: Override settings per cluster with Kustomize patches

