# Platform Configurations

This directory contains ArgoCD Application manifests for deploying **platform-level configurations** and Custom Resources.

## Sync Waves

All configs use sync waves **10-20** to deploy after operators are installed:

- **Wave 2**: `installplan-approver` - InstallPlanApprover CRs (deployed early)
- **Wave 10**: `cert-manager` - ClusterIssuers and Certificates
- **Wave 15-20**: Other platform configurations

## Configurations Included

### installplan-approver
**Purpose:** InstallPlanApprover CRs for multi-namespace approval  
**Namespace:** `iplan-approver-system`  
**Source:** [private-k8s-apps-repo](https://github.com/rajinator/private-k8s-apps-repo)  
**Wave:** 2 (needs to be deployed right after the operator)

### cert-manager
**Purpose:** ClusterIssuers and Certificates for TLS  
**Namespace:** `cert-manager` (ClusterIssuers are cluster-scoped)  
**Source:** [k8s-apps-repo](https://github.com/rajinator/k8s-apps-repo)  
**Wave:** 10  
**Resources:**
- ClusterIssuer: `le-bapu-route53` (Let's Encrypt with Route53 DNS-01)
- Certificates: API and apps wildcard certs

## Adding New Configurations

1. Create a new directory: `platform/configs/<config-name>/`
2. Add ArgoCD Application YAML: `<config-name>-config.yaml`
3. Create `kustomization.yaml`:
   ```yaml
   apiVersion: kustomize.config.k8s.io/v1alpha1
   kind: Component
   resources:
     - <config-name>-config.yaml
   ```
4. Add reference in `bootstrap/<cluster>/kustomization.yaml`
5. Set appropriate sync-wave (typically wave 10-20)

## Dependency Order

Configs depend on their corresponding operators being installed first:

```
Operator (wave 1-5) → Config (wave 10-20)
```

For example:
- `cert-manager-operator` (wave 5) → `cert-manager-config` (wave 10)
- `installplan-approver-operator` (wave 1) → `installplan-approver-config` (wave 2)

## Source Manifests

The actual Kubernetes manifests are stored in:
- **Public configs**: `k8s-apps-repo/ocp/<config-name>/`
- **Private configs**: `private-k8s-apps-repo/ocp/<config-name>/`

This repo only contains the **ArgoCD Application wrappers**.

