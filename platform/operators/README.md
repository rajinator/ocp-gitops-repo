# Platform Operators

This directory contains ArgoCD Application manifests for deploying **OLM Operators** to the OpenShift cluster.

## Sync Waves

All operators use sync waves **1-5** to ensure proper deployment order:

- **Wave 1**: `installplan-approver` - Enables automated InstallPlan approval for all operators
- **Wave 0, 3**: `openshift-gitops` - ArgoCD self-management and configuration
- **Wave 5**: All other operators (cert-manager, gitlab-runner, etc.)

## Operators Included

### installplan-approver
**Purpose:** Automates OLM InstallPlan approval across multiple namespaces  
**Namespace:** `iplan-approver-system`  
**Source:** [k8s-apps-repo](https://github.com/rajinator/k8s-apps-repo)

### cert-manager
**Purpose:** Red Hat cert-manager Operator for certificate management  
**Namespace:** `cert-manager-operator`  
**Source:** [private-k8s-apps-repo](https://github.com/rajinator/private-k8s-apps-repo)  
**Notes:** Uses recursive DNS nameservers overlay

### gitlab-runner
**Purpose:** GitLab Runner Operator for CI/CD  
**Namespace:** `gitlab-runner-operator`  
**Source:** [private-k8s-apps-repo](https://github.com/rajinator/private-k8s-apps-repo)

### openshift-gitops
**Purpose:** ArgoCD/OpenShift GitOps with custom configuration  
**Namespace:** `openshift-gitops`  
**Source:** In-cluster configurations  
**Notes:** Includes self-management with centralized InstallPlan approval

## Adding New Operators

1. Create a new directory: `platform/operators/<operator-name>/`
2. Add ArgoCD Application YAML: `<operator-name>-operator.yaml`
3. Create `kustomization.yaml`:
   ```yaml
   apiVersion: kustomize.config.k8s.io/v1alpha1
   kind: Component
   resources:
     - <operator-name>-operator.yaml
   ```
4. Add reference in `bootstrap/<cluster>/kustomization.yaml`
5. Set appropriate sync-wave (typically wave 5 for new operators)

## Source Manifests

The actual Kubernetes manifests (Namespace, OperatorGroup, Subscription) are stored in:
- **Public operators**: `k8s-apps-repo/ocp/<operator-name>/`
- **Private configs**: `private-k8s-apps-repo/ocp/<operator-name>/`

This repo only contains the **ArgoCD Application wrappers**.

