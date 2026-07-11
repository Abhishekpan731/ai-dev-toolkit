---
mode: manual
name: distributed-filesystem
description: distributed filesystem operations, troubleshooting, and Storage System integration
tags: [filesystem, storage-system, storage, filesystem, ddn]
version: 1.0.0
---

# Distributed Filesystem Filesystem - Storage System Integration

Expert knowledge for working with distributed filesystems in Storage System/storage provider environments.

## Overview

**Distributed Filesystem** is a high-performance distributed parallel filesystem used in HPC and large-scale storage systems.
**Storage System** is storage provider's commercial Distributed Filesystem distribution with enterprise features.

### Components
- **MGS** (Management Server): Stores configuration
- **MDS** (Metadata Server): Stores file metadata
- **MDT** (Metadata Target): Storage for metadata
- **OSS** (Object Storage Server): Stores file data
- **OST** (Object Storage Target): Storage for data
- **Client**: Mounts the filesystem

### Distributed Filesystem Path Format
```
<MGS_NID>@<NET>:/<FILESYSTEM_NAME>

Example: 10.3.3.200@tcp:/csi-fs
```

## Common Operations

### Mount Distributed Filesystem Filesystem
```bash
# Manual mount
mount -t filesystem 10.3.3.200@tcp:/csi-fs /mnt/filesystem

# Mount with options
mount -t filesystem -o user_xattr,flock 10.3.3.200@tcp:/csi-fs /mnt/filesystem

# Check mount
mount | grep filesystem
df -h | grep filesystem
```

### Unmount
```bash
# Unmount
umount /mnt/filesystem

# Force unmount (if stuck)
umount -f /mnt/filesystem
umount -l /mnt/filesystem  # Lazy unmount
```

### Distributed Filesystem Parameters (lctl)
```bash
# Get parameter
lctl get_param osc.*.checksums

# Set parameter
lctl set_param osc.*.checksums=0

# Set for specific filesystem (Storage Interface use case)
lctl set_param osc.csi-fs-*.checksums=0

# List all parameters
lctl list_param -R osc.*
```

### Quotas
```bash
# Check quota for user
lfs quota -u username /mnt/filesystem

# Check quota for project
lfs quota -p projectid /mnt/filesystem

# Set user quota (requires admin)
lfs setquota -u username -b 0 -B 100G -i 0 -I 1000000 /mnt/filesystem

# Set project quota
lfs setquota -p projectid -b 0 -B 500G /mnt/filesystem
```

### File Striping
```bash
# Get file striping info
lfs getstripe /mnt/filesystem/file

# Set stripe count (number of OSTs)
lfs setstripe -c 4 /mnt/filesystem/file

# Set stripe size
lfs setstripe -S 1M /mnt/filesystem/file

# Set on directory (default for new files)
lfs setstripe -c 2 -S 1M /mnt/filesystem/dir
```

## Storage System Storage Interface Integration

### Storage Class Parameters
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-system-sc
provisioner: exa.csi.ddn.com
allowVolumeExpansion: true
parameters:
  # Required: distributed filesystem path
  exaFS: "10.3.3.200@tcp:/csi-fs"
  
  # Required: Local mount point on Kubernetes node
  mountPoint: "/exa-csi-mnt"
  
  # Optional: Default mount options
  mountOptions: "user_xattr,flock"
  
  # Optional: Distributed Filesystem parameters (applied via lctl set_param)
  exaParams: "osc.*.checksums=0 ldlm.namespaces.*.lru_max_age=5000000"
  
  # Optional: Project ID for quota
  projectId: "1000"
```

### exaParams Format
Parameters in `exaParams` use wildcard `*` which is replaced by actual PVC mountpoint:

```yaml
# Before substitution (StorageClass)
exaParams: "osc.*.checksums=0"

# After substitution (applied by driver)
lctl set_param osc.csi-fs-pvc-abc123-*.checksums=0
```

Multiple parameters separated by space:
```yaml
exaParams: "osc.*.checksums=0 ldlm.namespaces.*.lru_max_age=5000000"
```

## Troubleshooting

### Mount Failures

**Check connectivity to MGS/MDS**:
```bash
# Ping MGS IP
ping 10.3.3.200

# Check Distributed Filesystem network
lctl list_nids
lctl ping 10.3.3.200@tcp
```

**Check kernel modules**:
```bash
# List loaded Distributed Filesystem modules
lsmod | grep filesystem

# Load modules (if missing)
modprobe filesystem
modprobe llite
```

**Check filesystem client version**:
```bash
# Client version
filesystem_rmmod -V

# Compatibility with server
# Client must be same or newer minor version
# Example: Client 2.14.0 can connect to Server 2.14.0 or 2.13.x
```

### Permission Denied

**Check mount options**:
```bash
# Add user_xattr if SELinux enabled
mount -t filesystem -o user_xattr 10.3.3.200@tcp:/csi-fs /mnt/filesystem
```

**Check filesystem permissions**:
```bash
ls -ld /mnt/filesystem
# Should match pod securityContext.fsGroup
```

### Quota Exceeded

**Check current usage**:
```bash
lfs quota -p <projectid> /mnt/filesystem

# Output shows:
# - block quota (storage)
# - inode quota (file count)
# - grace period if soft limit exceeded
```

**Increase quota** (admin required):
```bash
lfs setquota -p <projectid> -B <new_limit> /mnt/filesystem
```

### Stale File Handle

Occurs when file was deleted but process has it open.

```bash
# Find processes with open files
lsof | grep -i stale

# Kill processes
kill -9 <PID>

# Remount filesystem
umount -l /mnt/filesystem
mount -t filesystem 10.3.3.200@tcp:/csi-fs /mnt/filesystem
```

### Performance Issues

**Check OST status**:
```bash
lfs df -h
# Shows capacity and status of all OSTs
# Look for "inactive" OSTs
```

**Check client stats**:
```bash
lctl get_param llite.*.stats
lctl get_param osc.*.stats

# Reset stats
lctl set_param llite.*.stats=0
```

**Stripe optimization**:
```bash
# Large files: increase stripe count
lfs setstripe -c 8 /mnt/filesystem/largefile

# Small files: use single stripe
lfs setstripe -c 1 /mnt/filesystem/smallfile
```

## Storage System-Specific Features

### Compatibility Matrix
From storage-interface-driver/README.md:

| Storage Interface Driver | Storage System Client | Storage System Server |
|------------|------------------|------------------|
| >=2.7.0    | >=2.14.0-ddn235  | >=6.3.5          |
| 2.3.0-2.6.0| >=2.14.0-ddn182  | 6.3.2-6.3.4      |

### Client Installation
```bash
# Install Storage System client (Kubernetes node)
# Provided by exa-client-6.3.5.tar.gz or DEB package

dpkg -i storage-system-client_6.3.5_amd64.deb
```

### Metrics Collection
storage interface exports Prometheus metrics:
- `exa_csi_pvc_capacity_bytes`: Total PVC capacity
- `exa_csi_pvc_used_bytes`: Used bytes
- `exa_csi_pvc_available_bytes`: Available bytes
- `exa_csi_pvc_pod_count`: Number of pods using PVC

Access metrics:
```bash
curl http://localhost:32666/metrics | grep exa_csi
```

## Best Practices

1. **Always use project quotas** for PVCs (not user quotas)
2. **Match client/server versions** (same minor version)
3. **Monitor OST balance** - avoid hotspots
4. **Use appropriate striping** based on workload
5. **Enable checksums in production** (`osc.*.checksums=1`)
6. **Regular cleanup** of orphaned mounts

## References
- Distributed Filesystem Manual: https://doc.filesystem.org/
- Storage System Docs: storage provider customer portal
- Storage Interface Driver: storage-interface-driver/README.md
