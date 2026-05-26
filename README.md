# SeaweedFS Helm Chart

Baseline Helm chart for [SeaweedFS](https://github.com/seaweedfs/seaweedfs) distributed RWX storage. This chart wraps the SeaweedFS operator, the SeaweedFS CSI driver, optional StorageClasses, Seaweed CRs, and optional pre-provisioned PVCs.

## What this baseline does

- Uses sanitized defaults with no private hostnames, node names, or environment-specific capacity assumptions.
- Defines the `seaweedfs-media` StorageClass with two total copies by default (`replication: "010"`).
- Supports a generic `persistentVolumeClaims[]` list for pre-provisioned RWX/RWO claims.
- Keeps all environment-specific details in values overrides.

## Key values

| Area | Where | Default |
|------|-------|---------|
| Operator | `seaweedfs-operator.*` | One operator replica |
| CSI filer endpoint | `csi.seaweedfsFiler` | `seaweedfs-filer-media.seaweedfs.svc.cluster.local:8888` |
| CSI topology labels | `csi.node.injectTopologyInfoFromNodeLabel.labels` | `dataCenter` and `rack` labels |
| StorageClass | `storageClasses[]` | `seaweedfs-media`, expandable, `replication: "010"` |
| Seaweed clusters | `seaweeds[]` | One example `media` cluster |
| Pre-provisioned PVCs | `persistentVolumeClaims[]` | `[]` |

## Pre-provisioned PVCs

Use `persistentVolumeClaims[]` when you want the chart to create one or more shared claims alongside SeaweedFS:

```yaml
persistentVolumeClaims:
  - enabled: true
    name: media-data
    namespace: media
    storageClass: seaweedfs-media
    accessModes:
      - ReadWriteMany
    size: 32Ti
    volumeMode: Filesystem
    annotations: {}
    labels: {}
```

## Install

From Helm repo after publishing:

```bash
helm repo add expectedbehaviors https://expectedbehaviors.github.io/seaweedfs
helm install seaweedfs expectedbehaviors/seaweedfs -f my-values.yaml -n seaweedfs --create-namespace
```

## Render & validation

```bash
helm dependency update .
helm template seaweedfs . -f values.yaml -n seaweedfs
```

## Publishing (maintainers)

This chart is published to **https://expectedbehaviors.github.io/seaweedfs** via GitHub Actions:

1. **Release seaweedfs chart on merge to main** lints and creates `seaweedfs-v<version>`.
2. **Helm chart publish** packages the chart and publishes the Helm repo to GitHub Pages.
3. Enable GitHub Pages for the repo so the chart is pullable from `https://expectedbehaviors.github.io/seaweedfs`.

## Argo CD

For private instantiations, point Argo CD at the published Helm repo and provide your private values repo as the values source. In this homelab, the values repo is `git@github.com:jd4883/homelab-seaweedfs.git`.

## Upstream documentation

| Resource | URL |
|----------|-----|
| SeaweedFS | https://github.com/seaweedfs/seaweedfs |
| Operator | https://github.com/seaweedfs/seaweedfs-operator |
| CSI driver | https://github.com/seaweedfs/seaweedfs-csi-driver |
