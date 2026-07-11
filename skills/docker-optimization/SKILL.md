---
mode: auto
name: docker-optimization
description: Docker image optimization techniques for minimal size and maximum security
tags: [docker, optimization, security, containers]
version: 1.0.0
---

# Docker Image Optimization

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Expert knowledge for optimizing Docker images for the Storage System storage interface: reducing size, improving security, and accelerating builds.

## Technology Stack

- **Container Runtime**: Docker, containerd, CRI-O
- **Base Images**: Alpine, Distroless, Scratch
- **Build Tools**: Docker Buildx, BuildKit
- **Security**: Trivy, Docker Scout

## Multi-Stage Build Pattern

### Production-Ready Multi-Stage

```dockerfile
# Build stage
# @author <AUTHOR_NAME>
FROM golang:1.21-alpine AS builder

WORKDIR /build

# Dependency caching layer
COPY go.mod go.sum ./
RUN go mod download

# Build binary
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build \
    -ldflags="-w -s" \
    -o /storage-interface-driver \
    ./cmd/storage-interface-driver

# Final minimal image
FROM alpine:3.18

RUN apk add --no-cache ca-certificates filesystem-client && \
    rm -rf /var/cache/apk/*

COPY --from=builder /storage-interface-driver /usr/local/bin/

ENTRYPOINT ["/usr/local/bin/storage-interface-driver"]
```

**Size Reduction**: ~900MB → ~45MB (95% reduction)

## Layer Optimization Techniques

### 1. Combine RUN Commands
```dockerfile
# ❌ BAD: Multiple layers (3 layers)
RUN apk add --no-cache ca-certificates
RUN apk add --no-cache filesystem-client
RUN rm -rf /var/cache/apk/*

# ✅ GOOD: Single layer (1 layer)
RUN apk add --no-cache \
    ca-certificates \
    filesystem-client && \
    rm -rf /var/cache/apk/*
```

### 2. Order Layers by Change Frequency
```dockerfile
# ✅ GOOD: Least frequently changed first
FROM golang:1.21-alpine

# Rarely changes - cache this layer
RUN apk add --no-cache git make

# Changes occasionally - separate layer
COPY go.mod go.sum ./
RUN go mod download

# Changes frequently - last
COPY . .
RUN go build -o app ./cmd/app
```

### 3. Use .dockerignore
```
# .dockerignore
.git
.gitignore
*.md
!README.md
docs/
tests/
*.test
coverage.out
bin/
.vscode/
.idea/
```

## Base Image Selection

### Comparison

| Base Image | Size | Security | Use Case |
|------------|------|----------|----------|
| `scratch` | 0 MB | Highest | Static binaries only |
| `distroless/static` | 2 MB | High | Go apps (no shell) |
| `alpine:3.18` | 7 MB | Medium | Needs package manager |
| `ubuntu:22.04` | 77 MB | Low | Full OS features |

### Scratch Base (Minimal)
```dockerfile
FROM scratch
COPY --from=builder /storage-interface-driver /storage-interface
ENTRYPOINT ["/storage-interface"]
```

**Pros**: Smallest size, highest security  
**Cons**: No shell, no debugging tools

### Distroless Base
```dockerfile
FROM gcr.io/distroless/static:nonroot
COPY --from=builder /storage-interface-driver /usr/local/bin/
USER nonroot:nonroot
ENTRYPOINT ["/usr/local/bin/storage-interface-driver"]
```

**Pros**: Small, secure, no shell  
**Cons**: Harder to debug

## Build Optimization

### 1. BuildKit Cache Mounts
```dockerfile
# Enable BuildKit
# syntax=docker/dockerfile:1

FROM golang:1.21-alpine AS builder

WORKDIR /build

# Use cache mount for go modules
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=bind,source=go.sum,target=go.sum \
    --mount=type=bind,source=go.mod,target=go.mod \
    go mod download

# Use cache mount for build cache
RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    --mount=type=bind,target=. \
    go build -o /storage-interface ./cmd/storage-interface-driver
```

### 2. Build Arguments for Optimization
```dockerfile
ARG GO_VERSION=1.21
ARG ALPINE_VERSION=3.18

FROM golang:${GO_VERSION}-alpine AS builder

# Conditional compilation
ARG ENABLE_RACE_DETECTOR=false
RUN if [ "$ENABLE_RACE_DETECTOR" = "true" ]; then \
        go build -race -o /app ./cmd/app; \
    else \
        go build -ldflags="-w -s" -o /app ./cmd/app; \
    fi
```

## Security Hardening

### 1. Run as Non-Root
```dockerfile
FROM alpine:3.18

# Create user
RUN addgroup -g 1000 csi && \
    adduser -u 1000 -G csi -s /bin/sh -D csi

# Copy and set ownership
COPY --from=builder --chown=csi:csi /app /usr/local/bin/app

# Switch to non-root
USER csi

ENTRYPOINT ["/usr/local/bin/app"]
```

### 2. Read-Only Root Filesystem
```dockerfile
FROM alpine:3.18

# Create writable directory
RUN mkdir -p /tmp && chmod 1777 /tmp

# Run with read-only root
ENTRYPOINT ["/usr/local/bin/app"]
```

**Usage**:
```bash
docker run --read-only storage-system-storage-interface:latest
```

### 3. Minimal Capabilities
```dockerfile
FROM alpine:3.18

# Drop all capabilities
# Add back only what's needed at runtime
USER 1000:1000
```

## Binary Optimization

### 1. Linker Flags
```bash
# Remove debug info and symbol table
go build -ldflags="-w -s" -o app

# -w: Disable DWARF debug info
# -s: Disable symbol table
```

**Size Reduction**: ~30-40% smaller binary

### 2. Disable CGO (When Possible)
```bash
CGO_ENABLED=0 go build -o app
```

**Benefits**:
- Static binary (no libc dependency)
- Can use scratch base
- Smaller size
- Better portability

### 3. UPX Compression (Optional)
```dockerfile
FROM alpine:3.18 AS compress
RUN apk add --no-cache upx
COPY --from=builder /app /app
RUN upx --best --lzma /app

FROM scratch
COPY --from=compress /app /app
```

**Warning**: May impact startup time

## Multi-Architecture Builds

### Buildx for Multi-Arch
```bash
# Create builder
docker buildx create --name multiarch --use

# Build for multiple platforms
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t registry.example.com/storage-interface:latest \
  --push \
  .
```

### Platform-Specific Optimizations
```dockerfile
ARG TARGETPLATFORM
ARG BUILDPLATFORM

FROM --platform=$BUILDPLATFORM golang:1.21-alpine AS builder

# Extract platform info
ARG TARGETOS
ARG TARGETARCH

RUN CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} \
    go build -o /app
```

## Measurement and Validation

### Check Image Size
```bash
# List images with size
docker images storage-system-storage-interface

# Check layer sizes
docker history storage-system-storage-interface:latest

# Detailed analysis with dive
dive storage-system-storage-interface:latest
```

### Security Scanning
```bash
# Scan with trivy
trivy image storage-system-storage-interface:latest

# Fail on HIGH/CRITICAL
trivy image --severity HIGH,CRITICAL --exit-code 1 storage-system-storage-interface:latest
```

## Best Practices Checklist

- ✅ Multi-stage build for size reduction
- ✅ Specific base image version (not :latest)
- ✅ Combined RUN commands to reduce layers
- ✅ .dockerignore to exclude files
- ✅ No secrets in image layers
- ✅ Run as non-root user
- ✅ Minimal dependencies
- ✅ Security scanning in CI/CD
- ✅ Layer caching optimization
- ✅ Health check configured

## Reference Resources

- **Docker Best Practices**: https://docs.docker.com/develop/dev-best-practices/
- **Distroless Images**: https://github.com/GoogleContainerTools/distroless
- **Dive Tool**: https://github.com/wagoodman/dive
