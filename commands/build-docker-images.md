---
name: build-docker-images
description: Build Docker images for storage interface with multi-arch support
shortcut: /docker-build
---

# Build Docker Images Command

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Build Docker images for the Storage System storage interface with support for multiple architectures.

## Usage

```bash
# Build for current platform
/docker-build

# Build multi-arch (AMD64 + ARM64)
/docker-build --multi-arch

# Build and push to registry
/docker-build --push

# Build with specific version tag
/docker-build --tag=v1.2.3
```

## Command Execution

### Standard Build Script
```bash
#!/bin/bash
# @author <AUTHOR_NAME>

set -e

VERSION=${1:-latest}
REGISTRY=${2:-quay.io/storage-system}
IMAGE_NAME="storage-interface"
PUSH=${3:-false}

echo "=== Building Docker Images ==="
echo "Version: $VERSION"
echo "Registry: $REGISTRY"
echo ""

# 1. Build application binary
echo "Step 1: Building Go binary..."
cd storage-interface-driver
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags="-w -s -X main.version=$VERSION" \
    -o bin/storage-interface-driver-amd64 \
    ./cmd/storage-interface-driver

echo "✅ Binary built successfully"

# 2. Build Docker image
echo ""
echo "Step 2: Building Docker image..."
docker build \
    --build-arg VERSION=$VERSION \
    -t ${REGISTRY}/${IMAGE_NAME}:${VERSION} \
    -t ${REGISTRY}/${IMAGE_NAME}:latest \
    -f Dockerfile .

echo "✅ Image built: ${REGISTRY}/${IMAGE_NAME}:${VERSION}"

# 3. Tag image
echo ""
echo "Step 3: Tagging image..."
docker tag ${REGISTRY}/${IMAGE_NAME}:${VERSION} ${REGISTRY}/${IMAGE_NAME}:latest

# 4. Push to registry (if requested)
if [ "$PUSH" = "true" ]; then
    echo ""
    echo "Step 4: Pushing to registry..."
    docker push ${REGISTRY}/${IMAGE_NAME}:${VERSION}
    docker push ${REGISTRY}/${IMAGE_NAME}:latest
    echo "✅ Images pushed to registry"
fi

# 5. Display summary
echo ""
echo "=== Build Summary ==="
echo "Image: ${REGISTRY}/${IMAGE_NAME}:${VERSION}"
docker images ${REGISTRY}/${IMAGE_NAME}:${VERSION}
```

### Multi-Arch Build
```bash
#!/bin/bash
# @author <AUTHOR_NAME>

VERSION=${1:-latest}
REGISTRY="quay.io/storage-system"
IMAGE_NAME="storage-interface"

echo "=== Multi-Arch Docker Build ==="

# Create buildx builder
docker buildx create --name multiarch --use || docker buildx use multiarch

# Build for multiple platforms
docker buildx build \
    --platform linux/amd64,linux/arm64 \
    --build-arg VERSION=$VERSION \
    -t ${REGISTRY}/${IMAGE_NAME}:${VERSION} \
    -t ${REGISTRY}/${IMAGE_NAME}:latest \
    --push \
    -f Dockerfile \
    .

echo "✅ Multi-arch image built and pushed"
```

## Build Configurations

### Production Build
```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /build

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build \
    -ldflags="-w -s" \
    -o storage-system-storage-interface \
    ./cmd/storage-interface-driver

FROM alpine:3.18
RUN apk add --no-cache ca-certificates filesystem-client
COPY --from=builder /build/storage-interface-driver /usr/local/bin/
ENTRYPOINT ["/usr/local/bin/storage-interface-driver"]
```

### Development Build
```dockerfile
FROM golang:1.21
WORKDIR /app

COPY . .
RUN go build -o storage-system-storage-interface ./cmd/storage-interface-driver

ENTRYPOINT ["/app/storage-interface-driver"]
```

## Image Optimization

### Size Reduction Techniques
```dockerfile
# Use multi-stage builds
FROM golang:1.21-alpine AS builder
# ... build stage

FROM scratch
# Minimal final image

# Combine RUN commands
RUN apk add --no-cache \
    ca-certificates \
    && rm -rf /var/cache/apk/*

# Use .dockerignore
# .git
# *.md
# tests/
```

### Security Hardening
```dockerfile
# Run as non-root user
RUN addgroup -g 1000 csi && \
    adduser -u 1000 -G csi -s /bin/sh -D csi

USER csi

# Read-only root filesystem
COPY --from=builder --chown=csi:csi /build/binary /usr/local/bin/
```

## Testing Built Images

### Functional Test
```bash
# Test image runs
docker run --rm ${REGISTRY}/${IMAGE_NAME}:${VERSION} --version

# Test entrypoint
docker run --rm ${REGISTRY}/${IMAGE_NAME}:${VERSION} --help
```

### Security Scan
```bash
# Scan with trivy
trivy image ${REGISTRY}/${IMAGE_NAME}:${VERSION}

# Scan for critical vulnerabilities only
trivy image --severity CRITICAL ${REGISTRY}/${IMAGE_NAME}:${VERSION}
```

### Size Check
```bash
# Check image size
docker images ${REGISTRY}/${IMAGE_NAME}:${VERSION}

# Analyze layers
docker history ${REGISTRY}/${IMAGE_NAME}:${VERSION}
```

## Expected Output

```
=== Building Docker Images ===
Version: v1.2.3
Registry: quay.io/storage-system

Step 1: Building Go binary...
✅ Binary built successfully

Step 2: Building Docker image...
[+] Building 45.2s (12/12) FINISHED
✅ Image built: registry.example.com/storage-interface:v1.2.3

Step 3: Tagging image...

=== Build Summary ===
Image: registry.example.com/storage-interface:v1.2.3
REPOSITORY                        TAG       IMAGE ID       SIZE
registry.example.com/storage-interface     v1.2.3    abc123def456   45MB
```

## Makefile Integration

### Build Targets
```makefile
.PHONY: docker-build docker-push

VERSION ?= latest
REGISTRY ?= quay.io/storage-system

docker-build:
	@echo "Building Docker image..."
	docker build -t $(REGISTRY)/storage-interface:$(VERSION) .

docker-push: docker-build
	@echo "Pushing to registry..."
	docker push $(REGISTRY)/storage-interface:$(VERSION)

docker-multiarch:
	docker buildx build --platform linux/amd64,linux/arm64 \
		-t $(REGISTRY)/storage-interface:$(VERSION) --push .
```

## Reference Files

- **Docker Builder Agent**: `.ai-config/agents/docker-builder.md`
- **Project Dockerfile**: `storage-interface-driver/Dockerfile`
- **Makefile**: `storage-interface-driver/Makefile`
