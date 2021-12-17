---
path: "/docs/install-with-kurl/removing-minio"
date: "2021-12-17"
weight: 21
linktitle: "Removing Minio"
title: "Removing Minio from your Cluster"
---

As of [v2021.12.20-0](https://kurl.sh/release-notes/v2021.12.20-0), kURL clusters can be installed without addons having a peer dependency on object storage. 
There is also a migration path for clusters to remove object storage, such as is the case with removing Minio.

# New Installations

The following addons can be installed without object storage as noted:
1. **Registry**: without an object store in the installer spec, a Persistent Volume (PV) will be used for storage. Note, that if the Rook  addon is installed, the Registry will ALWAYS use this as an object store.
1. **Velero**: without object storage in the spec, no `default` BackupStorageLocation will be created. If the `disableS3` flag is set to `true` for the KOTS addon, a PV-backed storage location will be created as the `default` location using the [Local-Volume-Provider](https://github.com/replicatedhq/local-volume-provider) plugin.
1. **KOTS**: to allow KOTS to be deployed without an object store, the flag `kotsadm.disableS3` must be set to `true` in the installer. This will deploy KOTS as a statefulset using a Persistent Volume (PV) for storage. It will also disable the use of Minio for hostpath and NFS storage destinations.

This installer spec is an example of deploying a new cluster without using any object storage.
```yaml
apiVersion: cluster.kurl.sh/v1beta1
kind: Installer
metadata:
  name: no-object-storage
spec:
  kubernetes:
    version: 1.21.x
  containerd:
    version: 1.4.x
  weave:
    version: 2.6.5
  longhorn:
    version: 1.2.x
  registry:
    version: 2.5.7
  velero:
    version: 1.7.x
  kotsadm:
    version: 1.58.x
    disableS3: true
```

# Existing Cluster Migrations

There are two migration paths for removing object storage components.

## Enabling `disableS3` in the KOTS Addon

Object storage can be removed from some of the addons in the cluster by simply setting the `kotsadm.disableS3` flag in the KOTS addon to `true`.
See documentation in the [KOTS addon](/docs/add-ons/kotsadm) for more information on the `disableS3` flag.

When you re-install or upgrade using the updated installer spec (see the previous section for a sample), you should expect:
1. **Registry**: Nothing will change about the deployment. The registry will continue to use the same object storage.
1. **Velero**: the `default` BackupStorageLocation will be updated to point at an attached Persistent Volume using the [Local-Volume-Provider](https://github.com/replicatedhq/local-volume-provider) plugin. A migration will be triggered to copy from the object store into the attached PV.
1. **KOTS**: This will scale down the KOTS deployment, delete it, and re-deploy KOTS as a statefulset using a Persistent Volume (PV) for storage. It will also disable the use of Minio for hostpath and NFS storage destinations. A migration will be triggered to copy from the object store into the attached PV.

## Removing the Existing Provider

To fully remove object storage from the cluster, the current provider must be removed from your installer spec.
In the case of Minio, it is a straightforward removal of the addon.
For clusters using the Rook adddon, another CSI such as Longhorn or OpenEBS is required for storage. 
Data will be migrated between the storage providers automatically per existing [CSI Migrations](/docs/install-with-kurl/migrating-csi).
Any object store data used by addons will also be migrated to PVs.
You are also expected to set the `kotsadm.disableS3` flag to `true` in the installer to explicitly enable migrations for KOTS.
See documentation in the [KOTS addon](/docs/add-ons/kotsadm) for more information on the `disableS3` flag.

When you re-install or upgrade using the new updated spec (see the previous section for a sample), you should expect:
1. **Registry**: a Persistent Volume (PV) will be added for storage. A migration will be triggered to copy from the object store into the attached PV.
1. **Velero**: the `default` BackupStorageLocation will be updated to point at an attached Persistent Volume using the [Local-Volume-Provider](https://github.com/replicatedhq/local-volume-provider) plugin. A migration will be triggered to copy from the object store into the attached PV.
1. **KOTS**: This will scale down the KOTS deployment, delete it, and re-deploy KOTS as a statefulset using a Persistent Volume (PV) for storage. It will also disable the use of Minio for hostpath and NFS storage destinations. A migration will be triggered to copy from the object store into the attached PV.

