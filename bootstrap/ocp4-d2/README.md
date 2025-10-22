# OCP4-D2 Bootstrap

**Cluster:** `ocp4.d2.homelab.bapu.cloud`  
**Environment:** Homelab  
**GitOps Pattern:** App-of-Apps

## Overview

This is the bootstrap Application-of-Apps for the OpenShift 4 cluster in datacenter D2. It deploys all platform components and configurations needed for the cluster to be fully operational.

## Deployment Order (Sync Waves)

1. **Wave 0-5**: Platform Operators
   - InstallPlan Approver Operator (wave 1)
   - OpenShift GitOps Configuration (wave 0, 3)
   - Cert-Manager Operator (wave 5)
   - GitLab Runner Operator (wave 5)

2. **Wave 10-20**: Platform Configurations
   - InstallPlan Approver CRs (wave 2)
   - Cert-Manager ClusterIssuers & Certificates (wave 10)

3. **Wave 30+**: Applications (future)

## Bootstrap Application

Apply this as the initial ArgoCD Application:

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

## Customization

Cluster-specific patches can be added to `kustomization.yaml` in the `patches:` section.

Example:
```yaml
patches:
  - target:
      kind: Application
      name: cert-manager-operator
    patch: |-
      - op: replace
        path: /spec/source/targetRevision
        value: v1.0.1
```

## Related Documentation

- [Platform Operators](../../platform/operators/README.md)
- [Platform Configs](../../platform/configs/README.md)

