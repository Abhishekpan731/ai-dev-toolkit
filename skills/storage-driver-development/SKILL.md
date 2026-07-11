---
mode: auto
name: storage-driver-development
description: storage interface development best practices for Storage System/storage system
tags: [kubernetes, csi, storage, filesystem, storage-system]
version: 1.0.0
---

# Storage Interface Driver Development - Storage System/storage provider

Expert knowledge for developing and debugging Kubernetes storage interfaces for Storage System/Distributed Filesystem storage systems.

## Technology Stack

- **Language**: Go 1.20+
- **Framework**: Kubernetes Storage Interface Spec 1.5+
- **Storage**: Storage System (Distributed Filesystem-based), storage provider Storage
- **Deployment**: Kubernetes DaemonSet (node), Deployment (controller)
- **Build**: Makefile, Docker multi-stage builds

## Architecture Overview

### Components
1. **Controller Service** (Deployment)
   - CreateVolume, DeleteVolume
   - ControllerPublishVolume, ControllerUnpublishVolume
   - ValidateVolumeCapabilities

2. **Node Service** (DaemonSet)
   - NodeStageVolume, NodeUnstageVolume
   - NodePublishVolume, NodeUnpublishVolume
   - NodeGetCapabilities

3. **Identity Service** (Both)
   - GetPluginInfo, GetPluginCapabilities, Probe

### Volume Lifecycle
```
CreateVolume (controller) 
  → ControllerPublishVolume (controller)
  → NodeStageVolume (node)
  → NodePublishVolume (node)
  → [Pod uses volume]
  → NodeUnpublishVolume (node)
  → NodeUnstageVolume (node)
  → ControllerUnpublishVolume (controller)
  → DeleteVolume (controller)
```

## Code Quality Standards

### Function Length
**Mandatory**: All functions <50 lines (per .ai-config/rules/golang-standards.md)

```go
// ✅ GOOD: Extract helper functions
func (d *Driver) CreateVolume(ctx context.Context, req *csi.CreateVolumeRequest) (*csi.CreateVolumeResponse, error) {
    if err := d.validateCreateVolumeRequest(req); err != nil {
        return nil, err
    }
    
    volPath := d.buildVolumePath(req)
    if err := d.provisionVolume(volPath, req.CapacityRange); err != nil {
        return nil, fmt.Errorf("provision failed: %w", err)
    }
    
    return d.buildCreateVolumeResponse(volPath, req), nil
}

// ❌ BAD: Single 100-line function
func (d *Driver) CreateVolume(...) {
    // 100+ lines of validation, provisioning, response building
}
```

### Error Handling
```go
// ✅ GOOD: Wrap errors with context
if err := os.MkdirAll(targetPath, 0750); err != nil {
    return nil, status.Errorf(
        codes.Internal,
        "failed to create target path %s: %v", 
        targetPath, 
        err,
    )
}

// ❌ BAD: Silent failure or generic error
if err := os.MkdirAll(targetPath, 0750); err != nil {
    return nil, status.Error(codes.Internal, "mkdir failed")
}
```

### Documentation
```go
// NodePublishVolume mounts the volume to the target path on the node.
// It performs bind mount from staging path to target path.
//
// @author <AUTHOR_NAME>
// Parameters:
//   - ctx: Context for cancellation and timeout
//   - req: NodePublishVolumeRequest with volume ID and target path
//
// Returns:
//   - NodePublishVolumeResponse: Empty response on success
//   - error: GRPC status error with appropriate code
func (d *Driver) NodePublishVolume(ctx context.Context, req *csi.NodePublishVolumeRequest) (*csi.NodePublishVolumeResponse, error) {
```

## Common Patterns

### Idempotency
Storage Interface operations MUST be idempotent:

```go
func (d *Driver) NodeStageVolume(ctx context.Context, req *csi.NodeStageVolumeRequest) (*csi.NodeStageVolumeResponse, error) {
    // Check if already mounted
    mounted, err := d.isMounted(stagingPath)
    if err != nil {
        return nil, status.Errorf(codes.Internal, "mount check failed: %v", err)
    }
    
    if mounted {
        // Already staged - idempotent success
        return &csi.NodeStageVolumeResponse{}, nil
    }
    
    // Proceed with staging
    if err := d.mount(source, stagingPath, options); err != nil {
        return nil, status.Errorf(codes.Internal, "mount failed: %v", err)
    }
    
    return &csi.NodeStageVolumeResponse{}, nil
}
```

### GRPC Error Codes
Use appropriate status codes:

| Scenario | Code |
|----------|------|
| Invalid argument | `codes.InvalidArgument` |
| Volume not found | `codes.NotFound` |
| Already exists | `codes.AlreadyExists` |
| Out of capacity | `codes.ResourceExhausted` |
| Internal error | `codes.Internal` |
| Not implemented | `codes.Unimplemented` |

### Resource Cleanup
Always cleanup on errors:

```go
func (d *Driver) provisionVolume(path string) (err error) {
    created := false
    
    defer func() {
        if err != nil && created {
            // Cleanup on error
            _ = os.RemoveAll(path)
        }
    }()
    
    if err = os.MkdirAll(path, 0750); err != nil {
        return fmt.Errorf("mkdir failed: %w", err)
    }
    created = true
    
    if err = d.setQuota(path); err != nil {
        return fmt.Errorf("quota failed: %w", err)
    }
    
    return nil
}
```

## Testing

### Unit Tests
```go
func TestCreateVolume_EdgeCases(t *testing.T) {
    tests := []struct {
        name    string
        req     *csi.CreateVolumeRequest
        wantErr bool
        errCode codes.Code
    }{
        {
            name: "valid request",
            req: &csi.CreateVolumeRequest{
                Name: "test-vol",
                CapacityRange: &csi.CapacityRange{RequiredBytes: 1073741824},
            },
            wantErr: false,
        },
        {
            name: "missing name",
            req: &csi.CreateVolumeRequest{},
            wantErr: true,
            errCode: codes.InvalidArgument,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            d := NewFakeDriver()
            resp, err := d.CreateVolume(context.Background(), tt.req)
            
            if tt.wantErr {
                require.Error(t, err)
                st, ok := status.FromError(err)
                require.True(t, ok)
                assert.Equal(t, tt.errCode, st.Code())
            } else {
                require.NoError(t, err)
                assert.NotNil(t, resp)
            }
        })
    }
}
```

## Debugging

### Common Issues

1. **Mount failures**: Check node logs, verify filesystem connectivity
2. **Permission errors**: Review pod security context, fsGroup
3. **Quota issues**: Verify Storage System quota settings
4. **Stale mounts**: Cleanup orphaned mounts on node

### Debug Commands
```bash
# Check mounts on node
kubectl debug node/node-name -it --image=busybox -- mount | grep exa

# Inspect PVC/PV
kubectl get pvc,pv -o wide

# Driver logs
kubectl logs -n kube-system daemonset/exa-csi-node -c storage-interface-driver

# Events
kubectl get events --sort-by='.lastTimestamp' | grep -i csi
```

## References
- Storage Interface Spec: https://github.com/container-storage-interface/spec
- Storage System Storage Interface: storage-interface-driver/README.md
- Kubernetes Storage Interface: https://kubernetes-csi.github.io/docs/
