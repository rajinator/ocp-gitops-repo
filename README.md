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

# 2. Wait for operator ready
oc wait --for=condition=Ready pod -l name=openshift-gitops-operator -n openshift-gitops-operator --timeout=300s

# 3. Manually approve the OpenShift GitOps operator's InstallPlan (one-time only)
oc patch installplan $(oc get installplan -n openshift-gitops-operator -o name | head -1) \
  -n openshift-gitops-operator --type merge --patch '{"spec":{"approved":true}}'

# 4. Wait for ArgoCD instance to be ready
oc wait --for=condition=Ready argocd/openshift-gitops -n openshift-gitops --timeout=300s

# 5. Bootstrap App-of-Apps (includes centralized approval for all operators)
oc apply -f bootstrap/dev-cluster.yaml
```

**How It Works:**
1. OpenShift GitOps operator is bootstrapped manually (one-time setup)
2. App-of-Apps deploys:
   - **InstallPlan Approver Operator + CR** (sync-wave 1)
   - **OpenShift GitOps self-management** (sync-wave 3)
   - **All other operators** (sync-wave 5+)
3. All subsequent InstallPlans are automatically approved by the centralized operator

**Benefits:**
- ✅ **Event-driven:** No polling, instant approval
- ✅ **Version control:** Only approves if CSV matches Subscription's startingCSV
- ✅ **Centralized:** One operator handles all namespaces
- ✅ **No CronJobs:** No race conditions or timing issues

## ArgoCD Access

```bash
# Get route
oc get route openshift-gitops-server -n openshift-gitops

# Get admin password
oc get secret openshift-gitops-cluster -n openshift-gitops -o jsonpath='{.data.admin\.password}' | base64 -d
```

## Available Apps

### Infrastructure
- **installplan-approver-operator** - Centralized OLM InstallPlan approval (operator + multi-namespace CR)

### GitOps Platform
- **openshift-gitops-custom** - Custom ArgoCD config with version-pinning aware health checks
- **openshift-gitops-with-auto-approval** - Operator self-management (uses centralized approval)

### OLM Operators
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

