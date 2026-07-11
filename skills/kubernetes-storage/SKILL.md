---
mode: auto
name: kubernetes-storage
description: Kubernetes storage concepts, patterns, and Storage Interface integration
tags: [kubernetes, storage, csi, persistent-volumes, storage-class]
version: 1.0.0
---

# Kubernetes Storage Concepts

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Expert knowledge of Kubernetes storage architecture, persistent volumes, storage classes, and storage interface integration.

## Technology Stack

- **Kubernetes**: 1.24+
- **OpenShift**: 4.12+
- **Storage Interface Spec**: v1.8.0
- **Storage**: Various backends (Distributed Filesystem, NFS, Ceph, etc.)

## Storage Architecture

### Storage Hierarchy
```
StorageClass
    ↓
PersistentVolume (PV)
    ↓
PersistentVolumeClaim (PVC)
    ↓
Pod (volumeMount)
```

## Persistent Volumes (PV)

### Static PV Provisioning
```yaml
# @author <AUTHOR_NAME>
apiVersion: v1
kind: PersistentVolume
metadata:
  name: filesystem-pv-static
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: storage-system-fs
  csi:
    driver: storage-system.csi.example.com
    volumeHandle: static-vol-001
    volumeAttributes:
      filesystem: "filesystem01"
```

### Dynamic PV Provisioning
```yaml
# Automatic PV creation via StorageClass
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Gi
  storageClassName: storage-system-fs
# PV automatically created by Storage Interface provisioner
```

## Access Modes

### Understanding Access Modes
```go
// @author <AUTHOR_NAME>
type AccessMode int

const (
    // ReadWriteOnce (RWO): Single node read-write
    ReadWriteOnce AccessMode = iota
    
    // ReadOnlyMany (ROX): Multiple nodes read-only
    ReadOnlyMany
    
    // ReadWriteMany (RWX): Multiple nodes read-write
    ReadWriteMany
    
    // ReadWriteOncePod (RWOP): Single pod read-write (K8s 1.27+)
    ReadWriteOncePod
)
```

### Access Mode Selection
```yaml
# File storage (NFS, Distributed Filesystem): RWX capable
spec:
  accessModes:
    - ReadWriteMany

# Block storage (iSCSI, FC): Typically RWO
spec:
  accessModes:
    - ReadWriteOnce

# Shared read-only data
spec:
  accessModes:
    - ReadOnlyMany
```

## Storage Classes

### Basic StorageClass
```yaml
# @author <AUTHOR_NAME>
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-system-fs
provisioner: storage-system.csi.example.com
parameters:
  fsType: "filesystem"
  filesystem: "filesystem01"
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true
```

### Advanced StorageClass Features

#### Volume Binding Modes
```yaml
# Immediate: Provision as soon as PVC created
volumeBindingMode: Immediate

# WaitForFirstConsumer: Provision when pod scheduled
volumeBindingMode: WaitForFirstConsumer
```

#### Reclaim Policies
```yaml
# Delete: Remove volume when PVC deleted
reclaimPolicy: Delete

# Retain: Keep volume for manual cleanup
reclaimPolicy: Retain
```

#### Allowed Topologies
```yaml
allowedTopologies:
  - matchLabelExpressions:
    - key: topology.kubernetes.io/zone
      values:
        - us-east-1a
        - us-east-1b
```

## Volume Expansion

### Enable Expansion
```yaml
# @author <AUTHOR_NAME>
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-system-fs-expandable
provisioner: storage-system.csi.example.com
allowVolumeExpansion: true  # Enable expansion
```

### Expand Volume
```yaml
# Edit PVC to increase size
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  resources:
    requests:
      storage: 100Gi  # Increased from 50Gi
```

### Expansion Process
```go
// @author <AUTHOR_NAME>
// Storage Interface Driver must implement ControllerExpandVolume and NodeExpandVolume

func (d *Driver) ControllerExpandVolume(ctx context.Context, req *csi.ControllerExpandVolumeRequest) (*csi.ControllerExpandVolumeResponse, error) {
    volumeID := req.GetVolumeId()
    newSize := req.GetCapacityRange().GetRequiredBytes()
    
    if err := d.expandVolume(volumeID, newSize); err != nil {
        return nil, fmt.Errorf("expand volume: %w", err)
    }
    
    return &csi.ControllerExpandVolumeResponse{
        CapacityBytes:         newSize,
        NodeExpansionRequired: true,  // If filesystem resize needed
    }, nil
}
```

## Volume Snapshots

### VolumeSnapshotClass
```yaml
# @author <AUTHOR_NAME>
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: storage-system-snapclass
driver: storage-system.csi.example.com
deletionPolicy: Delete
parameters:
  snapshotType: "shallow"
```

### Create Snapshot
```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: my-snapshot
spec:
  volumeSnapshotClassName: storage-system-snapclass
  source:
    persistentVolumeClaimName: source-pvc
```

### Restore from Snapshot
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: restored-pvc
spec:
  dataSource:
    name: my-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Gi
```

## Volume Cloning

### Clone PVC
```yaml
# @author <AUTHOR_NAME>
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: cloned-pvc
spec:
  dataSource:
    name: source-pvc
    kind: PersistentVolumeClaim
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Gi
  storageClassName: storage-system-fs
```

## Generic Ephemeral Volumes

### Inline Volume Definition
```yaml
# @author <AUTHOR_NAME>
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-ephemeral
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: scratch
      mountPath: /scratch
  volumes:
  - name: scratch
    ephemeral:
      volumeClaimTemplate:
        spec:
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 10Gi
          storageClassName: storage-system-fs
```

## Volume Topology

### Topology-Aware Provisioning
```yaml
# @author <AUTHOR_NAME>
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-system-fs-topology
provisioner: storage-system.csi.example.com
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
  - matchLabelExpressions:
    - key: topology.kubernetes.io/zone
      values:
        - zone-a
    - key: topology.storage-system.com/rack
      values:
        - rack-1
```

## Storage Capacity Tracking

### CSIStorageCapacity
```yaml
# @author <AUTHOR_NAME>
apiVersion: storage.k8s.io/v1
kind: CSIStorageCapacity
metadata:
  name: storage-system-capacity-zone-a
storageClassName: storage-system-fs
nodeTopology:
  matchLabels:
    topology.kubernetes.io/zone: zone-a
capacity: 10Ti
maximumVolumeSize: 5Ti
```

## Best Practices

### 1. Use StorageClasses
```yaml
# ✅ GOOD: Define StorageClass for dynamic provisioning
# ❌ BAD: Manually create PVs for each application
```

### 2. Set Resource Limits
```yaml
# @author <AUTHOR_NAME>
apiVersion: v1
kind: ResourceQuota
metadata:
  name: storage-quota
spec:
  hard:
    requests.storage: "1Ti"
    persistentvolumeclaims: "100"
```

### 3. Use Volume Binding Mode Appropriately
```yaml
# For distributed apps: WaitForFirstConsumer
volumeBindingMode: WaitForFirstConsumer

# For immediate availability: Immediate
volumeBindingMode: Immediate
```

## Monitoring Storage

### Check PV/PVC Status
```bash
# @author <AUTHOR_NAME>
# List PVs
kubectl get pv

# List PVCs in namespace
kubectl get pvc -n <namespace>

# Describe PVC
kubectl describe pvc <pvc-name> -n <namespace>

# Check storage usage
kubectl get pvc -A -o custom-columns=\
NAME:.metadata.name,\
NAMESPACE:.metadata.namespace,\
CAPACITY:.status.capacity.storage,\
ACCESS:.spec.accessModes
```

## Reference Files

- **Storage Interface Driver Development**: `.ai-config/skills/storage-driver-development/SKILL.md`
- **K8s Operator Patterns**: `.ai-config/skills/kubernetes-operator-patterns/SKILL.md`
- **Deploy Command**: `.ai-config/commands/deploy-driver.md`
