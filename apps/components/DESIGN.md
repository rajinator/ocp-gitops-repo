# GitOps Components Design

## Why Two Separate Components for GitOps?

You might notice we have TWO components for managing OpenShift GitOps:

1. `openshift-gitops-custom` - ArgoCD instance configuration
2. `openshift-gitops-self-managed` - Operator resources (Subscription, etc.)

### Why Not Combine Them?

**Separation of Concerns:**
- **Operator Layer**: Subscription, InstallPlans, RBAC (managed by OLM)
- **Instance Layer**: ArgoCD CR configuration (managed by Operator)

These operate at different levels and have different sync behaviors:

| Aspect | Operator (self-managed) | Instance (custom) |
|--------|------------------------|-------------------|
| Manages | Subscription, RBAC | ArgoCD CR |
| Controlled by | OLM | GitOps Operator |
| Namespace | `openshift-gitops-operator` | `openshift-gitops` |
| Update flow | CSV → InstallPlan → Approval | Direct patch to CR |
| Restart impact | Operator pod | ArgoCD pods |

**Practical Benefits:**
1. **Clear boundaries**: Operator vs. instance configuration
2. **Independent updates**: Change ArgoCD config without touching operator
3. **Easier troubleshooting**: Know which layer has issues
4. **Flexibility**: Use default ArgoCD config OR custom overlay

### When to Use Each

```yaml
# Minimal setup (default ArgoCD config)
# apps/dev-cluster/kustomization.yaml
components:
  - ../components/openshift-gitops-self-managed

# Custom setup (OLM health checks, SSO, etc.)
# apps/dev-cluster/kustomization.yaml
components:
  - ../components/openshift-gitops-custom      # ← Add this
  - ../components/openshift-gitops-self-managed
```

### Alternative: All-in-One Component?

We could create a combined component, but you'd lose flexibility:

```yaml
# Hypothetical combined component
components:
  - ../components/openshift-gitops-all-in-one  # Operator + custom config

# Problem: Can't use just operator with default config
# Problem: Can't swap custom overlays easily
```

## Conclusion

Two components = More verbose but more flexible and clear.

Think of it like Kubernetes itself:
- **Install**: `kubectl` (like our operator component)
- **Configure**: `kubeconfig` (like our custom component)

Both needed, but separate concerns.

