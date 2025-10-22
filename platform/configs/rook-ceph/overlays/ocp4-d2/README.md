# Rook Ceph Configuration for ocp4-d2

Cluster-specific Rook Ceph storage configuration for `ocp4.d2.homelab.bapu.cloud`.

## Configuration

### Storage Backend
- **Cluster**: ocp4-d2
- **Devices**: `/dev/sdb`
- **Node Selection**: All nodes (`useAllNodes: true`)

## Customization

To modify disk selection for this cluster, edit the patch in `kustomization.yaml`:

```yaml
patches:
  - target:
      kind: CephCluster
      name: rook-ceph
    patch: |-
      - op: replace
        path: /spec/storage/devices
        value:
          - name: "sdb"
          - name: "sdc"  # Add additional disks here
```

### Use Specific Nodes

To select specific nodes instead of all nodes:

```yaml
patches:
  - target:
      kind: CephCluster
      name: rook-ceph
    patch: |-
      - op: replace
        path: /spec/storage/useAllNodes
        value: false
      - op: add
        path: /spec/storage/nodes
        value:
          - name: "worker1.ocp4.d2.homelab.bapu.cloud"
            devices:
              - name: "sdb"
          - name: "worker2.ocp4.d2.homelab.bapu.cloud"
            devices:
              - name: "sdb"
```

## Deployment

This configuration is deployed via ArgoCD Application in `platform/configs/rook-ceph/rook-ceph-cluster.yaml`.

The Application references this overlay directory, allowing GitOps-driven storage configuration per cluster.

