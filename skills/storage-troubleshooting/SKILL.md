---
mode: auto
name: storage-troubleshooting
description: Advanced storage interface debugging and troubleshooting techniques
tags: [csi, debugging, troubleshooting, kubernetes, storage]
version: 1.0.0
---

# Storage Interface Driver Troubleshooting

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Expert knowledge for troubleshooting Storage Interface (Container Storage Interface) driver issues in Kubernetes and OpenShift environments.

## Technology Stack

- **Storage Interface Spec**: v1.8.0
- **Kubernetes**: 1.24+
- **OpenShift**: 4.12+
- **Storage**: storage provider Storage System (Distributed Filesystem)
- **Debugging Tools**: kubectl, grpcurl, csi-sanity

## Common Storage Interface Issues

### 1. Volume Provisioning Failures

#### Symptoms
```
Events:
  Warning  ProvisioningFailed  External provisioner failed to provision volume
```

#### Diagnosis
```bash
# @author <AUTHOR_NAME>
# Check controller logs
kubectl logs -n storage-system-storage-interface \
  -l app=storage-system-controller \
  -c csi-provisioner

# Check driver logs
kubectl logs -n storage-system-storage-interface \
  -l app=storage-system-controller \
  -c storage-interface | grep CreateVolume
```

#### Common Causes

**A. Invalid Parameters**
```yaml
# Check StorageClass parameters
kubectl describe storageclass storage-system-fs

# Validate parameters
parameters:
  fsType: "filesystem"  # Must be valid
  filesystem: "filesystem01"  # Must exist
```

**B. Insufficient Permissions**
```bash
# Check RBAC
kubectl auth can-i create persistentvolumes \
  --as=system:serviceaccount:storage-system-storage-interface:storage-interface-controller
```

**C. Storage Backend Unavailable**
```bash
# Test Distributed Filesystem connectivity from controller pod
kubectl exec -it -n storage-system-storage-interface \
  <controller-pod> -c storage-interface -- \
  lfs df
```

### 2. Volume Mount Failures

#### Symptoms
```
Events:
  Warning  FailedMount  MountVolume.MountDevice failed for volume
```

#### Diagnosis Steps
```bash
# @author <AUTHOR_NAME>
diagnose_mount_failure() {
    local POD_NAME=$1
    local NAMESPACE=$2
    
    # Get PVC name
    PVC=$(kubectl get pod $POD_NAME -n $NAMESPACE \
      -o jsonpath='{.spec.volumes[*].persistentVolumeClaim.claimName}')
    
    # Get PV
    PV=$(kubectl get pvc $PVC -n $NAMESPACE \
      -o jsonpath='{.spec.volumeName}')
    
    # Check VolumeAttachment
    kubectl get volumeattachments | grep $PV
    
    # Check node pod logs
    NODE=$(kubectl get pod $POD_NAME -n $NAMESPACE \
      -o jsonpath='{.spec.nodeName}')
    kubectl logs -n storage-system-storage-interface \
      -l app=storage-system-node \
      --field-selector spec.nodeName=$NODE
}
```

#### Common Causes

**A. Distributed Filesystem Client Not Installed**
```bash
# Check on node
kubectl exec -it -n storage-system-storage-interface \
  <node-pod> -c storage-interface -- \
  which mount.filesystem

# If missing, install filesystem client on nodes
```

**B. Network Connectivity**
```bash
# Test MGS/MDS/OSS reachability
kubectl exec -it -n storage-system-storage-interface \
  <node-pod> -c storage-interface -- \
  ping <filesystem-mds-ip>
```

**C. Mount Options Invalid**
```bash
# Check mount options in StorageClass
kubectl get storageclass storage-system-fs -o yaml | grep mountOptions

# Valid options: flock, user_xattr, lazystatfs, noatime
```

### 3. Volume Deletion Issues

#### Symptoms
```
PersistentVolume stuck in "Terminating" state
VolumeAttachment not deleted
```

#### Resolution
```bash
# @author <AUTHOR_NAME>
force_volume_cleanup() {
    local PV_NAME=$1
    
    # Delete VolumeAttachment
    VA=$(kubectl get volumeattachments -o json | \
      jq -r --arg pv "$PV_NAME" \
      '.items[] | select(.spec.source.persistentVolumeName==$pv) | .metadata.name')
    
    if [ -n "$VA" ]; then
        kubectl delete volumeattachment $VA --force --grace-period=0
    fi
    
    # Remove finalizers
    kubectl patch pv $PV_NAME -p '{"metadata":{"finalizers":null}}' --type=merge
    
    # Delete PV
    kubectl delete pv $PV_NAME --force --grace-period=0
}
```

## Storage Interface Sanity Testing

### Running Sanity Tests
```bash
# Install csi-sanity
go install github.com/kubernetes-csi/csi-test/cmd/csi-sanity@latest

# Run tests
csi-sanity \
  --csi.endpoint=/csi/csi.sock \
  --csi.stagingdir=/tmp/staging \
  --csi.mountdir=/tmp/mount
```

### Interpreting Results
```
# Expected output
Using ginkgo.DefaultReporter
PASS
```

## GRPC Debugging

### Inspect Storage Interface Calls
```bash
# @author <AUTHOR_NAME>
# Install grpcurl
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest

# List services
grpcurl -plaintext \
  -unix /csi/csi.sock \
  list

# Get plugin info
grpcurl -plaintext \
  -unix /csi/csi.sock \
  csi.v1.Identity/GetPluginInfo

# Get capabilities
grpcurl -plaintext \
  -unix /csi/csi.sock \
  csi.v1.Controller/ControllerGetCapabilities
```

### Test Volume Creation
```bash
# Create volume via GRPC
grpcurl -plaintext \
  -unix /csi/csi.sock \
  -d '{
    "name": "test-vol",
    "capacity_range": {"required_bytes": 1073741824},
    "volume_capabilities": [{"mount": {}, "access_mode": {"mode": 5}}]
  }' \
  csi.v1.Controller/CreateVolume
```

## Log Analysis Patterns

### Error Patterns
```go
// @author <AUTHOR_NAME>
// Common error patterns to search for
var errorPatterns = []string{
    "rpc error",
    "permission denied",
    "not found",
    "already exists",
    "invalid argument",
    "timeout",
    "deadline exceeded",
}

func analyzeCSILogs(logFile string) {
    for _, pattern := range errorPatterns {
        // grep pattern in logs
        cmd := fmt.Sprintf("grep -i '%s' %s", pattern, logFile)
        // execute and analyze
    }
}
```

### Critical Log Lines
```bash
# Search for critical errors
kubectl logs -n storage-system-storage-interface \
  -l app=storage-system-controller \
  | grep -E "error|failed|panic|fatal"

# Search for specific volume
kubectl logs -n storage-system-storage-interface \
  -l app=storage-system-controller \
  | grep "vol-123"
```

## Performance Issues

### Slow Volume Operations
```bash
# Enable profiling in driver
# Add flag: --enable-pprof=true

# Collect CPU profile
curl http://localhost:6060/debug/pprof/profile?seconds=30 > cpu.prof

# Analyze with pprof
go tool pprof cpu.prof
```

### High Memory Usage
```bash
# Check memory profile
curl http://localhost:6060/debug/pprof/heap > heap.prof
go tool pprof heap.prof
```

## Troubleshooting Checklist

### Pre-Deployment
- [ ] storage interface image available
- [ ] filesystem client installed on all nodes
- [ ] Network connectivity to Distributed Filesystem MGS/MDS/OSS
- [ ] RBAC permissions configured
- [ ] StorageClass created

### Runtime Debugging
- [ ] Controller pods running
- [ ] Node pods running on all nodes
- [ ] storage interface registered (kubectl get csidrivers)
- [ ] Storage Interface nodes registered (kubectl get csinodes)
- [ ] No errors in controller logs
- [ ] No errors in node logs

### Volume Issues
- [ ] PVC in Bound state
- [ ] VolumeAttachment created
- [ ] Volume mounted on correct node
- [ ] Mount options correct
- [ ] distributed filesystem accessible

## Reference Files

- **Storage Interface Debugger Agent**: `.ai-config/agents/storage-debugger.md`
- **K8s Troubleshooter**: `.ai-config/agents/k8s-troubleshooter.md`
- **Debug Command**: `.ai-config/commands/debug-csi-issue.md`
- **Troubleshooting Guide**: `.ai-config/guides/TROUBLESHOOTING_GUIDE.md`
