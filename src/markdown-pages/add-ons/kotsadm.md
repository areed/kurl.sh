---
path: "/docs/add-ons/kotsadm"
date: "2020-04-23"
linktitle: "KOTS Add-On"
weight: 38
title: "KOTS Add-On"
addOn: "kotsadm"
---

The [KOTS add-on](https://kots.io/kotsadm/installing/installing-a-kots-app/) installs an admin console for managing KOTS apps.

This add-on requires an S3-Compatible object store be available in the cluster by default. 
Both the Rook and the MinIO add-ons satisfy the object store requirement.
To deploy KOTS without an object store, set the `disableS3` flag in the installer to `true`. 
This will install KOTS as a statefulset using a Persistent Volume (PV) for storage.

## Advanced Install Options

```yaml
spec:
  kotsadm: 
    version: "latest"
    applicationSlug: "slug"
    uiBindPort: 8800
    hostname: "hostname"
    applicationNamespace: "kots"
    disableS3: true
```

flags-table

## Side Effects of the `disableS3` flag

Object storage is used by the KOTS addon as default behaviors in the following ways:
1. Directly, as a way for the console to store support-bundles and previous versions of your application.
1. Directly, when hostpath or NFS locations are set in the console. The console will dynamically create a Minio instance and attach to the specified source.
1. Indirectly, as part of the Registry addon, to store container images in airgap deployments.
1. Indirectly, as part of the Velero addon, to store snapshots in the cluster using the "Internal" option.

When the disableS3 flag is set to `true`, the KOTS admin console will switch to use a PV for all of the direct operations. 
It will also have the following side effects outside of the KOTS addon:
1. For new installs, the Regsistry addon behavior will not change. It will use an object store if it is available, otherwise it will use a PV. For upgrades or re-installs, a migration will be performed from an object store to a PV ONLY if the object store is removed from the installer spec.
1. For new installs, the Velero addon will use a PV for storing "Internal" snapshots instead of using object storage. For upgrades or re-installs, internal snapshots will be migrated from the object store to a PV.

Migrations will NOT be performed if the flag is not set. 

## Airgap Example

For installing KOTS apps in airgap mode, the registry add-on must also be included.

```
apiVersion: "cluster.kurl.sh/v1beta1"
kind: "Installer"
spec:
  kubernetes:
    version: latest
  containerd: 
    version: latest
  weave:
    version: latest
  rook:
    version: latest
  ekco:
    version: latest
  registry:
    version: latest
  kotsadm: 
    version: latest
```

## Online Example with MinIO

```
apiVersion: "cluster.kurl.sh/v1beta1"
kind: "Installer"
spec:
  kubernetes:
    version: latest
  containerd: 
    version: latest
  weave:
    version: latest
  openebs:
    version: latest
    isLocalPVEnabled: true
    localPVStorageClassName: default
  minio:
    version: latest
  registry:
    version: latest
  kotsadm: 
    version: latest
```

## Example without Object Storage

```
apiVersion: "cluster.kurl.sh/v1beta1"
kind: "Installer"
spec:
  kubernetes:
    version: latest
  docker: 
    version: latest
  weave:
    version: latest
  longhorn:
    version: latest
  registry:
    version: latest
  kotsadm: 
    version: latest
    disableS3: true
```
