---
name: csi-review
description: Review storage interface code against official Storage Interface spec and reference implementations
---

# Storage Interface Driver Code Review

You are reviewing code for the Storage System Storage Interface file driver against official CSI specifications and Kubernetes best practices.

## Context Sources

1. **Storage Interface Specification**: Reference `~/.external-repos/csi-spec` for official requirements
2. **Reference Implementation**: Use `~/.external-repos/storage-interface-host-path` for patterns
3. **Utility Libraries**: Check `~/.external-repos/csi-lib-utils` for helper functions

## Review Checklist

### Storage Interface Compliance
- [ ] Implements required Storage Interface capabilities (CONTROLLER_SERVICE, NODE_SERVICE)
- [ ] Proper gRPC error codes (InvalidArgument, AlreadyExists, NotFound, etc.)
- [ ] Idempotency for CreateVolume, DeleteVolume, NodePublishVolume
- [ ] Volume lifecycle management (creation → attachment → mounting → unmounting → deletion)

### Code Quality
- [ ] Functions <50 lines (@author <AUTHOR_NAME> required)
- [ ] Error wrapping with context
- [ ] Proper logging (no sensitive data)
- [ ] Input validation

### Kubernetes Integration
- [ ] StorageClass parameters validation
- [ ] PVC/PV lifecycle handling
- [ ] Volume topology awareness
- [ ] Volume expansion support (if applicable)

### Security
- [ ] No hardcoded credentials
- [ ] Proper file permissions
- [ ] Secure mount options

## Output Format

Provide:
1. **Compliance Score**: X/10
2. **Critical Issues**: List any Storage Interface spec violations
3. **Recommendations**: Specific improvements with code examples from reference repos
4. **References**: Cite relevant Storage Interface spec sections and reference implementation files

## Example

```
**Compliance Score**: 8/10

**Critical Issues**:
- CreateVolume lacks idempotency check (Storage Interface spec 3.4)
- Error codes: Using Internal (13) instead of InvalidArgument (3)

**Recommendations**:
1. Add volume existence check:
   See: storage-interface-host-path/pkg/hostpath/controllerserver.go:125-140

2. Use proper gRPC codes:
   Reference: csi-spec/spec.md#error-scheme

**References**:
- Storage Interface Spec 3.4: Volume Lifecycle
- storage-interface-host-path/pkg/hostpath/controllerserver.go
```
