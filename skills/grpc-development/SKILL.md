---
mode: auto
name: grpc-development
description: GRPC service development patterns for storage interfaces
tags: [grpc, csi, golang, rpc, protobuf]
version: 1.0.0
---

# GRPC Development for Storage Interface Drivers

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Expert knowledge for developing GRPC services for Storage Interface (Container Storage Interface) drivers in Go.

## Technology Stack

- **Language**: Go 1.21+
- **GRPC**: v1.60+
- **Protobuf**: v3
- **Storage Interface Spec**: v1.8.0
- **Transport**: Unix domain sockets

## GRPC Server Setup

### Basic Server Implementation
```go
// @author <AUTHOR_NAME>
package driver

import (
    "net"
    
    "google.golang.org/grpc"
    "github.com/container-storage-interface/spec/lib/go/csi"
)

type CSIDriver struct {
    endpoint string
    server   *grpc.Server
    
    csi.UnimplementedIdentityServer
    csi.UnimplementedControllerServer
    csi.UnimplementedNodeServer
}

func (d *CSIDriver) Run() error {
    // Remove existing socket
    if err := os.Remove(d.endpoint); err != nil && !os.IsNotExist(err) {
        return fmt.Errorf("remove socket: %w", err)
    }
    
    // Create listener
    listener, err := net.Listen("unix", d.endpoint)
    if err != nil {
        return fmt.Errorf("create listener: %w", err)
    }
    
    // Create GRPC server
    d.server = grpc.NewServer(
        grpc.UnaryInterceptor(d.logInterceptor),
    )
    
    // Register services
    csi.RegisterIdentityServer(d.server, d)
    csi.RegisterControllerServer(d.server, d)
    csi.RegisterNodeServer(d.server, d)
    
    // Serve
    return d.server.Serve(listener)
}
```

### Graceful Shutdown
```go
// @author <AUTHOR_NAME>
func (d *CSIDriver) Stop() {
    if d.server != nil {
        d.server.GracefulStop()
    }
}
```

## Storage Interface Identity Service

### GetPluginInfo
```go
// @author <AUTHOR_NAME>
func (d *CSIDriver) GetPluginInfo(ctx context.Context, req *csi.GetPluginInfoRequest) (*csi.GetPluginInfoResponse, error) {
    return &csi.GetPluginInfoResponse{
        Name:          "storage-system.csi.example.com",
        VendorVersion: d.version,
    }, nil
}
```

### GetPluginCapabilities
```go
// @author <AUTHOR_NAME>
func (d *CSIDriver) GetPluginCapabilities(ctx context.Context, req *csi.GetPluginCapabilitiesRequest) (*csi.GetPluginCapabilitiesResponse, error) {
    return &csi.GetPluginCapabilitiesResponse{
        Capabilities: []*csi.PluginCapability{
            {
                Type: &csi.PluginCapability_Service_{
                    Service: &csi.PluginCapability_Service{
                        Type: csi.PluginCapability_Service_CONTROLLER_SERVICE,
                    },
                },
            },
            {
                Type: &csi.PluginCapability_VolumeExpansion_{
                    VolumeExpansion: &csi.PluginCapability_VolumeExpansion{
                        Type: csi.PluginCapability_VolumeExpansion_ONLINE,
                    },
                },
            },
        },
    }, nil
}
```

### Probe
```go
// @author <AUTHOR_NAME>
func (d *CSIDriver) Probe(ctx context.Context, req *csi.ProbeRequest) (*csi.ProbeResponse, error) {
    // Check driver health
    if err := d.checkHealth(); err != nil {
        return nil, status.Error(codes.FailedPrecondition, err.Error())
    }
    
    return &csi.ProbeResponse{Ready: &wrapperspb.BoolValue{Value: true}}, nil
}
```

## Storage Interface Controller Service

### CreateVolume
```go
// @author <AUTHOR_NAME>
func (d *CSIDriver) CreateVolume(ctx context.Context, req *csi.CreateVolumeRequest) (*csi.CreateVolumeResponse, error) {
    // Validate request
    if err := validateCreateVolumeRequest(req); err != nil {
        return nil, status.Error(codes.InvalidArgument, err.Error())
    }
    
    // Check idempotency
    if existing, err := d.getVolume(req.GetName()); err == nil {
        return &csi.CreateVolumeResponse{Volume: existing}, nil
    }
    
    // Create volume
    volume, err := d.createVolume(ctx, req)
    if err != nil {
        return nil, status.Errorf(codes.Internal, "create volume: %v", err)
    }
    
    return &csi.CreateVolumeResponse{Volume: volume}, nil
}
```

### DeleteVolume
```go
// @author <AUTHOR_NAME>
func (d *CSIDriver) DeleteVolume(ctx context.Context, req *csi.DeleteVolumeRequest) (*csi.DeleteVolumeResponse, error) {
    volumeID := req.GetVolumeId()
    if volumeID == "" {
        return nil, status.Error(codes.InvalidArgument, "volume ID required")
    }
    
    // Idempotent: succeed if already deleted
    if err := d.deleteVolume(ctx, volumeID); err != nil {
        if isNotFoundError(err) {
            return &csi.DeleteVolumeResponse{}, nil
        }
        return nil, status.Errorf(codes.Internal, "delete volume: %v", err)
    }
    
    return &csi.DeleteVolumeResponse{}, nil
}
```

## Error Handling

### GRPC Status Codes
```go
// @author <AUTHOR_NAME>
import (
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
)

// Map errors to GRPC codes
func toGRPCError(err error) error {
    switch {
    case errors.Is(err, ErrNotFound):
        return status.Error(codes.NotFound, err.Error())
    case errors.Is(err, ErrAlreadyExists):
        return status.Error(codes.AlreadyExists, err.Error())
    case errors.Is(err, ErrInvalidArgument):
        return status.Error(codes.InvalidArgument, err.Error())
    default:
        return status.Error(codes.Internal, err.Error())
    }
}
```

### Error with Details
```go
// @author <AUTHOR_NAME>
func returnErrorWithDetails(err error) error {
    st := status.New(codes.Internal, err.Error())
    
    // Add error details
    details := &errdetails.ErrorInfo{
        Reason: "VOLUME_CREATION_FAILED",
        Domain: "storage-system.csi.example.com",
    }
    
    st, _ = st.WithDetails(details)
    return st.Err()
}
```

## Interceptors

### Logging Interceptor
```go
// @author <AUTHOR_NAME>
func (d *CSIDriver) logInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {
    start := time.Now()
    
    log.Infof("GRPC call: %s", info.FullMethod)
    log.Debugf("Request: %+v", req)
    
    resp, err := handler(ctx, req)
    
    duration := time.Since(start)
    if err != nil {
        log.Errorf("GRPC error: %s, duration: %v, error: %v", info.FullMethod, duration, err)
    } else {
        log.Infof("GRPC success: %s, duration: %v", info.FullMethod, duration)
    }
    
    return resp, err
}
```

### Recovery Interceptor
```go
// @author <AUTHOR_NAME>
func recoveryInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {
    defer func() {
        if r := recover(); r != nil {
            log.Errorf("Panic in %s: %v\n%s", info.FullMethod, r, debug.Stack())
        }
    }()
    
    return handler(ctx, req)
}
```

## Testing GRPC Services

### Unit Test with Mock Client
```go
// @author <AUTHOR_NAME>
func TestCreateVolume(t *testing.T) {
    driver := &CSIDriver{}
    
    req := &csi.CreateVolumeRequest{
        Name: "test-vol",
        CapacityRange: &csi.CapacityRange{
            RequiredBytes: 1073741824,
        },
    }
    
    resp, err := driver.CreateVolume(context.Background(), req)
    assert.NoError(t, err)
    assert.NotNil(t, resp.Volume)
    assert.Equal(t, "test-vol", resp.Volume.VolumeId)
}
```

### Integration Test with Real Server
```go
// @author <AUTHOR_NAME>
func TestCreateVolumeIntegration(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping integration test")
    }
    
    // Start server
    endpoint := "/tmp/csi-test.sock"
    driver := NewTestDriver(endpoint)
    go driver.Run()
    defer driver.Stop()
    
    // Create client
    conn, err := grpc.Dial(
        "unix://"+endpoint,
        grpc.WithTransportCredentials(insecure.NewCredentials()),
    )
    require.NoError(t, err)
    defer conn.Close()
    
    client := csi.NewControllerClient(conn)
    
    // Test CreateVolume
    resp, err := client.CreateVolume(context.Background(), &csi.CreateVolumeRequest{
        Name: "integration-test-vol",
    })
    
    require.NoError(t, err)
    assert.NotNil(t, resp.Volume)
}
```

## Performance Optimization

### Connection Pooling
```go
// @author <AUTHOR_NAME>
// Client-side connection pooling
func newGRPCClient(endpoint string) (csi.ControllerClient, error) {
    conn, err := grpc.Dial(
        endpoint,
        grpc.WithTransportCredentials(insecure.NewCredentials()),
        grpc.WithDefaultCallOptions(
            grpc.MaxCallRecvMsgSize(16*1024*1024),
            grpc.MaxCallSendMsgSize(16*1024*1024),
        ),
    )
    if err != nil {
        return nil, err
    }
    
    return csi.NewControllerClient(conn), nil
}
```

## Reference Files

- **Storage Interface Driver Development**: `.ai-config/skills/storage-driver-development/SKILL.md`
- **Go Best Practices**: `.ai-config/skills/go-best-practices/SKILL.md`
- **Storage Interface Troubleshooting**: `.ai-config/skills/storage-troubleshooting/SKILL.md`
