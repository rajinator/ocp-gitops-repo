# OpenShift GitOps Repository

GitOps repository for managing OpenShift clusters using ArgoCD Application-of-Apps pattern.

Assisted by: Cursor with claude-4.5-sonnet

## Repository Structure

```
ocp-gitops-repo/
├── bootstrap/                    # Cluster bootstrap configurations
│   └── ocp4-d2/                 # ocp4.d2.homelab.bapu.cloud
│       ├── kustomization.yaml   # App-of-Apps entrypoint
│       └── README.md
│
├── platform/                     # Platform-level components (admin-managed)
│   ├── operators/               # OLM Operators (sync-wave 1-5)
│   │   ├── installplan-approver/
│   │   ├── cert-manager/
│   │   ├── gitlab-runner/
│   │   ├── openshift-gitops/
│   │   └── README.md
│   │
│   └── configs/                 # Platform configurations (sync-wave 10-20)
│       ├── installplan-approver/
│       ├── cert-manager/
│       └── README.md
│
└── apps/                        # Application workloads (sync-wave 30+)
    └── README.md                # Currently empty, reserved for future apps
```

## GitOps Pattern

This repository uses the **App-of-Apps** pattern:
- Each cluster has a **bootstrap Application** in `bootstrap/<cluster>/`
- Bootstrap references **platform operators** and **platform configs**
- ArgoCD deploys everything in **dependency order** using sync-waves

## Sync Wave Strategy

| Wave | Category | Purpose | Examples |
|------|----------|---------|----------|
| 0-5 | Platform Operators | Install OLM operators | InstallPlan Approver, Cert-Manager, GitLab Runner |
| 10-20 | Platform Configs | Configure operators | ClusterIssuers, InstallPlanApprover CRs |
| 30+ | Applications | Deploy workloads | Future applications |

## Cluster Deployment

### Initial Bootstrap

1. **Install OpenShift GitOps Operator** (manual, one-time):
   ```bash
   kubectl apply -k bootstrap/openshift-gitops-operator/base
   ```

2. **Create Bootstrap Application**:
   ```bash
   kubectl apply -f - <<EOF
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: ocp4-d2-bootstrap
     namespace: openshift-gitops
   spec:
     project: default
     source:
       repoURL: https://github.com/rajinator/ocp-gitops-repo
       targetRevision: main
       path: bootstrap/ocp4-d2
     destination:
       name: in-cluster
       namespace: openshift-gitops
     syncPolicy:
       automated:
         prune: false
         selfHeal: true
   EOF
   ```

3. **Watch Deployment**:
   ```bash
   kubectl get applications -n openshift-gitops
   argocd app list
   ```

## Source Repositories

This repository contains **only ArgoCD Application manifests** (pointers). Actual Kubernetes resources are in:

- **Public operators/configs**: [k8s-apps-repo](https://github.com/rajinator/k8s-apps-repo)
- **Private configs/secrets**: [private-k8s-apps-repo](https://github.com/rajinator/private-k8s-apps-repo)

## Adding New Clusters

1. Create new directory: `bootstrap/<cluster-name>/`
2. Copy and customize `bootstrap/ocp4-d2/kustomization.yaml`
3. Adjust cluster-specific labels and patches
4. Apply bootstrap Application pointing to new path

## Key Features

- ✅ **Centralized InstallPlan Approval** - Automated operator upgrades
- ✅ **Cert-Manager Integration** - Automated TLS certificate management
- ✅ **GitOps Self-Management** - ArgoCD manages itself
- ✅ **Sync Wave Orchestration** - Proper dependency ordering
- ✅ **Multi-Cluster Ready** - Easy to add new clusters

## Related Documentation

- [Bootstrap Documentation](./bootstrap/ocp4-d2/README.md)
- [Platform Operators](./platform/operators/README.md)
- [Platform Configs](./platform/configs/README.md)
- [Applications](./apps/README.md)

