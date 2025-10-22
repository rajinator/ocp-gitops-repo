# Applications

This directory is reserved for **application workloads** and **tenant applications** that run on the platform.

## Structure (Proposed)

```
apps/
├── infrastructure/        # Infrastructure services (Gitea, Harbor, Vault)
│   ├── gitea/
│   ├── harbor/
│   └── vault/
│
└── workloads/            # User applications
    ├── nextcloud/
    ├── plex/
    └── home-automation/
```

## Sync Waves

Applications use sync waves **30+** to deploy after all platform components are ready:

- **Wave 30**: Infrastructure applications
- **Wave 40+**: User workloads

## Currently Empty

No applications are currently deployed via this GitOps structure. Applications will be added as needed.

## Adding Applications

1. Create a new directory: `apps/<category>/<app-name>/`
2. Add ArgoCD Application YAML: `<app-name>.yaml`
3. Create `kustomization.yaml`:
   ```yaml
   apiVersion: kustomize.config.k8s.io/v1alpha1
   kind: Component
   resources:
     - <app-name>.yaml
   ```
4. Add reference in `bootstrap/<cluster>/kustomization.yaml`
5. Set appropriate sync-wave (typically wave 30+)

## Application Sources

Applications can be sourced from:
- **Helm charts** (public/private registries)
- **Kustomize overlays** (in separate app repos)
- **Raw manifests** (for simple applications)

## Related Documentation

- [Platform Operators](../platform/operators/README.md) - Required operators for apps
- [Platform Configs](../platform/configs/README.md) - Required platform configurations

