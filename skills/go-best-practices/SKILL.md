---
mode: auto
name: go-best-practices
description: Go programming best practices and patterns for storage interface development
tags: [go, golang, best-practices, patterns]
version: 1.0.0
---

# Go Best Practices for Storage Interface Driver

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Expert knowledge of Go best practices specifically for storage interface and Kubernetes controller development.

## Technology Stack

- **Language**: Go 1.21+
- **Key Packages**: context, errors, fmt, sync
- **Frameworks**: grpc, kubernetes client-go
- **Testing**: testing, testify, gomock

## Code Organization Patterns

### 1. Package Structure
```go
// Good package organization
pkg/
├── driver/        // storage interface implementation
│   ├── driver.go  // Main driver struct and New()
│   ├── controller.go
│   ├── node.go
│   └── identity.go
├── mount/         // Mount operations
│   ├── mount.go
│   └── filesystem.go
└── utils/         // Shared utilities
    ├── volume.go
    └── errors.go
```

### 2. Interface Definitions
```go
// @author <AUTHOR_NAME>
// Define interfaces for testability
type StorageBackend interface {
    CreateVolume(ctx context.Context, name string, size int64) (*Volume, error)
    DeleteVolume(ctx context.Context, id string) error
    GetVolume(ctx context.Context, id string) (*Volume, error)
}

// Concrete implementation
type FilesystemBackend struct {
    config *Config
}

func (l *FilesystemBackend) CreateVolume(ctx context.Context, name string, size int64) (*Volume, error) {
    // Implementation
}
```

## Error Handling Patterns

### 1. Error Wrapping
```go
// @author <AUTHOR_NAME>
// Always wrap errors with context
func (d *Driver) CreateVolume(ctx context.Context, req *Request) (*Response, error) {
    volume, err := d.backend.CreateVolume(ctx, req.Name, req.Size)
    if err != nil {
        return nil, fmt.Errorf("create volume %s: %w", req.Name, err)
    }
    
    if err := d.setQuota(volume.ID, req.Size); err != nil {
        // Cleanup on error
        _ = d.backend.DeleteVolume(ctx, volume.ID)
        return nil, fmt.Errorf("set quota for volume %s: %w", volume.ID, err)
    }
    
    return buildResponse(volume), nil
}
```

### 2. Custom Error Types
```go
// @author <AUTHOR_NAME>
// Define custom errors for specific cases
type VolumeNotFoundError struct {
    VolumeID string
}

func (e *VolumeNotFoundError) Error() string {
    return fmt.Sprintf("volume %s not found", e.VolumeID)
}

func IsVolumeNotFound(err error) bool {
    var notFoundErr *VolumeNotFoundError
    return errors.As(err, &notFoundErr)
}

// Usage
if err := deleteVolume(id); err != nil {
    if IsVolumeNotFound(err) {
        // Handle not found case
        return nil
    }
    return err
}
```

### 3. GRPC Error Codes
```go
// @author <AUTHOR_NAME>
import (
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"
)

func (d *Driver) CreateVolume(ctx context.Context, req *csi.CreateVolumeRequest) (*csi.CreateVolumeResponse, error) {
    // Input validation
    if req.GetName() == "" {
        return nil, status.Error(codes.InvalidArgument, "volume name required")
    }
    
    // Check if exists
    existing, err := d.getVolume(req.GetName())
    if err == nil {
        return &csi.CreateVolumeResponse{Volume: existing}, nil
    }
    
    // Create volume
    volume, err := d.createVolume(ctx, req)
    if err != nil {
        return nil, status.Errorf(codes.Internal, "create failed: %v", err)
    }
    
    return &csi.CreateVolumeResponse{Volume: volume}, nil
}
```

## Concurrency Patterns

### 1. Context Propagation
```go
// @author <AUTHOR_NAME>
// Always pass context through call chain
func (d *Driver) processRequest(ctx context.Context, req *Request) error {
    // Check context cancellation
    select {
    case <-ctx.Done():
        return ctx.Err()
    default:
    }
    
    // Pass context to downstream calls
    return d.backend.Execute(ctx, req)
}
```

### 2. Goroutine Management
```go
// @author <AUTHOR_NAME>
// Use errgroup for concurrent operations
import "golang.org/x/sync/errgroup"

func (d *Driver) processMultiple(ctx context.Context, volumes []string) error {
    g, ctx := errgroup.WithContext(ctx)
    
    for _, vol := range volumes {
        vol := vol // Capture loop variable
        g.Go(func() error {
            return d.processVolume(ctx, vol)
        })
    }
    
    return g.Wait()
}
```

### 3. Mutex for Shared State
```go
// @author <AUTHOR_NAME>
type VolumeCache struct {
    mu      sync.RWMutex
    volumes map[string]*Volume
}

func (c *VolumeCache) Get(id string) (*Volume, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    vol, ok := c.volumes[id]
    return vol, ok
}

func (c *VolumeCache) Set(id string, vol *Volume) {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    c.volumes[id] = vol
}
```

## Resource Management

### 1. Defer for Cleanup
```go
// @author <AUTHOR_NAME>
func (d *Driver) mountVolume(volumeID string) error {
    // Acquire lock
    d.mu.Lock()
    defer d.mu.Unlock()
    
    // Open file
    f, err := os.Open("/path/to/volume")
    if err != nil {
        return err
    }
    defer f.Close()
    
    // Process file
    return process(f)
}
```

### 2. Context with Timeout
```go
// @author <AUTHOR_NAME>
func (d *Driver) createWithTimeout(parentCtx context.Context, req *Request) error {
    ctx, cancel := context.WithTimeout(parentCtx, 30*time.Second)
    defer cancel()
    
    done := make(chan error, 1)
    
    go func() {
        done <- d.create(ctx, req)
    }()
    
    select {
    case err := <-done:
        return err
    case <-ctx.Done():
        return fmt.Errorf("create timeout: %w", ctx.Err())
    }
}
```

## Testing Patterns

### 1. Table-Driven Tests
```go
// @author <AUTHOR_NAME>
func TestValidateRequest(t *testing.T) {
    tests := []struct {
        name    string
        req     *Request
        wantErr bool
        errCode codes.Code
    }{
        {
            name:    "valid request",
            req:     &Request{Name: "vol1", Size: 1024},
            wantErr: false,
        },
        {
            name:    "missing name",
            req:     &Request{Size: 1024},
            wantErr: true,
            errCode: codes.InvalidArgument,
        },
        {
            name:    "zero size",
            req:     &Request{Name: "vol1", Size: 0},
            wantErr: true,
            errCode: codes.InvalidArgument,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := validateRequest(tt.req)
            
            if (err != nil) != tt.wantErr {
                t.Errorf("validateRequest() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            
            if err != nil && status.Code(err) != tt.errCode {
                t.Errorf("error code = %v, want %v", status.Code(err), tt.errCode)
            }
        })
    }
}
```

### 2. Mock Interfaces
```go
// @author <AUTHOR_NAME>
//go:generate mockgen -source=backend.go -destination=mock_backend.go -package=driver

// Test with mocks
func TestCreateVolume(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()
    
    mockBackend := NewMockStorageBackend(ctrl)
    mockBackend.EXPECT().
        CreateVolume(gomock.Any(), "vol1", int64(1024)).
        Return(&Volume{ID: "vol-123"}, nil)
    
    driver := &Driver{backend: mockBackend}
    
    resp, err := driver.CreateVolume(context.Background(), &Request{
        Name: "vol1",
        Size: 1024,
    })
    
    assert.NoError(t, err)
    assert.Equal(t, "vol-123", resp.Volume.ID)
}
```

## Performance Optimization

### 1. Avoid Allocations
```go
// @author <AUTHOR_NAME>
// ❌ BAD: Creates new slice on each call
func processItems(items []string) {
    result := []string{}
    for _, item := range items {
        result = append(result, process(item))
    }
}

// ✅ GOOD: Pre-allocate with capacity
func processItems(items []string) {
    result := make([]string, 0, len(items))
    for _, item := range items {
        result = append(result, process(item))
    }
}
```

### 2. String Building
```go
// @author <AUTHOR_NAME>
// ❌ BAD: String concatenation
func buildPath(parts []string) string {
    path := ""
    for _, p := range parts {
        path += "/" + p
    }
    return path
}

// ✅ GOOD: Use strings.Builder
func buildPath(parts []string) string {
    var b strings.Builder
    for _, p := range parts {
        b.WriteString("/")
        b.WriteString(p)
    }
    return b.String()
}
```

## Reference Materials

- **Go Spec**: https://go.dev/ref/spec
- **Effective Go**: https://go.dev/doc/effective_go
- **Go Proverbs**: https://go-proverbs.github.io/
- **Uber Go Style**: https://github.com/uber-go/guide/blob/master/style.md
