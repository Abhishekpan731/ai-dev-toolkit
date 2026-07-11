---
mode: auto
name: filesystem-performance-tuning
description: distributed filesystem performance optimization and tuning techniques
tags: [filesystem, performance, tuning, optimization, storage]
version: 1.0.0
---

# Distributed Filesystem Performance Tuning

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Expert knowledge for optimizing distributed filesystem performance for high-throughput, low-latency workloads in storage interface environments.

## Technology Stack

- **Distributed Filesystem**: 2.15+
- **Client**: Linux kernel 5.15+
- **Network**: InfiniBand, RoCE, or high-speed Ethernet
- **Storage Backend**: storage provider Storage System

## Client-Side Tuning

### 1. Mount Options
```yaml
# @author <AUTHOR_NAME>
# Optimal mount options for Storage Interface StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: storage-system-fs-performance
provisioner: storage-system.csi.example.com
parameters:
  fsType: "filesystem"
mountOptions:
  # Enable file locking
  - "flock"
  # Extended attributes
  - "user_xattr"
  # Lazy statfs for performance
  - "lazystatfs"
  # Don't update access time
  - "noatime"
  # Increase dirty cache (in MB)
  - "max_dirty_mb=1024"
  # Parallel RPCs in flight
  - "max_rpcs_in_flight=32"
  # Read-ahead (in MB)
  - "max_read_ahead_mb=256"
```

### 2. Client Parameters
```bash
# @author <AUTHOR_NAME>
tune_filesystem_client() {
    local FSNAME="filesystem01"
    
    # Increase dirty cache (default: 32MB)
    lctl set_param llite.${FSNAME}*.max_dirty_mb=1024
    
    # Increase read-ahead window (default: 40MB)
    lctl set_param llite.${FSNAME}*.max_read_ahead_mb=256
    
    # Increase RPC parallelism (default: 8)
    lctl set_param llite.${FSNAME}*.max_rpcs_in_flight=32
    
    # Enable checksums for data integrity
    lctl set_param llite.${FSNAME}*.checksum_pages=1
    
    # Optimize for small files
    lctl set_param llite.${FSNAME}*.statahead_max=64
}
```

### 3. Stripe Configuration
```bash
# @author <AUTHOR_NAME>
# Optimize striping for workload

# Large files (>1GB): Wide striping
lfs setstripe -c -1 /mnt/filesystem/large-files  # Use all OSTs

# Small files: Single stripe
lfs setstripe -c 1 /mnt/filesystem/small-files

# Medium files: 4-8 stripes
lfs setstripe -c 4 -S 1M /mnt/filesystem/medium-files

# Progressive File Layout (PFL)
lfs setstripe -E 1M -c 1 -E 16M -c 4 -E -1 -c -1 /mnt/filesystem/pfl
```

## I/O Patterns Optimization

### Sequential I/O
```bash
# @author <AUTHOR_NAME>
optimize_sequential_io() {
    # Large stripe size for sequential workloads
    lfs setstripe -c 8 -S 4M /mnt/filesystem/sequential
    
    # Increase read-ahead
    lctl set_param llite.*.max_read_ahead_mb=512
    
    # Increase max_rpcs_in_flight
    lctl set_param llite.*.max_rpcs_in_flight=64
}
```

### Random I/O
```bash
# @author <AUTHOR_NAME>
optimize_random_io() {
    # Smaller stripe size, more stripes
    lfs setstripe -c -1 -S 1M /mnt/filesystem/random
    
    # Disable read-ahead for random access
    lctl set_param llite.*.max_read_ahead_mb=0
    
    # Enable statahead for metadata
    lctl set_param llite.*.statahead_max=128
}
```

### Metadata-Intensive Workloads
```bash
# @author <AUTHOR_NAME>
optimize_metadata() {
    # Use multiple MDTs with DNE
    lfs setdirstripe -i 0 -c 4 /mnt/filesystem/metadata-intensive
    
    # Enable statahead
    lctl set_param llite.*.statahead_max=256
    
    # Disable atime updates
    mount -o remount,noatime /mnt/filesystem
}
```

## Network Tuning

### LNET Configuration
```bash
# @author <AUTHOR_NAME>
# /etc/modprobe.d/filesystem.conf

# Set peer timeout (seconds)
options lnet peer_timeout=180

# Number of peer credits
options lnet peer_credits=128

# Router buffer size
options lnet router_buffer_credits=8192
```

### TCP Optimization
```bash
# /etc/sysctl.conf
net.core.rmem_max=16777216
net.core.wmem_max=16777216
net.ipv4.tcp_rmem=4096 87380 16777216
net.ipv4.tcp_wmem=4096 65536 16777216
net.core.netdev_max_backlog=30000
```

## Monitoring Performance

### Real-Time Stats
```bash
# @author <AUTHOR_NAME>
monitor_filesystem_performance() {
    # OST stats
    lctl get_param obdfilter.*.stats
    
    # Client stats
    lctl get_param llite.*.stats
    
    # RPC stats
    lctl get_param osc.*.rpc_stats
    
    # BRW stats (Bulk Read/Write)
    lctl get_param osc.*.brw_stats
}
```

### Key Metrics
```bash
# Throughput
watch -n 1 'lctl get_param obdfilter.*.stats | grep -E "read_bytes|write_bytes"'

# IOPS
watch -n 1 'lctl get_param obdfilter.*.stats | grep -E "read |write "'

# Latency
lctl get_param osc.*.rpc_stats
```

## Benchmarking

### IOR Benchmark
```bash
# @author <AUTHOR_NAME>
# Sequential write test
mpirun -np 16 ior \
  -w -r \
  -b 1m -t 1m -s 1000 \
  -o /mnt/filesystem/ior_test

# Random I/O test
mpirun -np 16 ior \
  -w -r -z \
  -b 4k -t 4k -s 10000 \
  -o /mnt/filesystem/ior_random
```

### MDTest (Metadata)
```bash
# Create many files
mdtest -n 100000 -d /mnt/filesystem/mdtest -C

# Stat files
mdtest -n 100000 -d /mnt/filesystem/mdtest -T

# Remove files
mdtest -n 100000 -d /mnt/filesystem/mdtest -r
```

### FIO Benchmark
```bash
# Sequential write
fio --name=seq-write \
  --ioengine=libaio \
  --rw=write \
  --bs=1M \
  --size=10G \
  --numjobs=8 \
  --directory=/mnt/filesystem

# Random 4K IOPS
fio --name=rand-iops \
  --ioengine=libaio \
  --rw=randrw \
  --bs=4k \
  --size=1G \
  --numjobs=16 \
  --directory=/mnt/filesystem
```

## Best Practices

### 1. Workload-Specific Tuning
```go
// @author <AUTHOR_NAME>
type WorkloadProfile struct {
    Type        string
    StripeCount int
    StripeSize  int64
    MountOpts   []string
}

var profiles = map[string]WorkloadProfile{
    "hpc": {
        Type:        "sequential",
        StripeCount: -1,  // Use all OSTs
        StripeSize:  4194304,  // 4MB
        MountOpts:   []string{"flock", "lazystatfs", "max_dirty_mb=2048"},
    },
    "database": {
        Type:        "random",
        StripeCount: 4,
        StripeSize:  1048576,  // 1MB
        MountOpts:   []string{"flock", "noatime", "max_rpcs_in_flight=64"},
    },
}
```

### 2. Avoid Anti-Patterns
```bash
# ❌ BAD: Single stripe for large file
lfs setstripe -c 1 large-file.dat

# ✅ GOOD: Wide striping for large file
lfs setstripe -c -1 -S 4M large-file.dat

# ❌ BAD: Too many stripes for small files
lfs setstripe -c 32 small-files/

# ✅ GOOD: Single stripe for small files
lfs setstripe -c 1 small-files/
```

## Troubleshooting Performance

### Identify Bottlenecks
```bash
# @author <AUTHOR_NAME>
identify_bottleneck() {
    # Check OST balance
    lfs df -h
    
    # Check slow OSTs
    lctl get_param obdfilter.*.stats | grep -A 2 "write_bytes"
    
    # Check client RPC stats
    lctl get_param osc.*.rpc_stats
    
    # Check for errors
    dmesg | grep -i filesystem
}
```

### Common Issues
```bash
# Slow metadata operations
# Solution: Enable statahead
lctl set_param llite.*.statahead_max=256

# Low throughput
# Solution: Increase stripe count
lfs setstripe -c -1 /path/to/data

# High latency
# Solution: Check network and increase RPC parallelism
lctl set_param osc.*.max_rpcs_in_flight=32
```

## Reference Files

- **Distributed Filesystem Filesystem Skill**: `.ai-config/skills/distributed-filesystem/SKILL.md`
- **Performance Optimizer Agent**: `.ai-config/agents/performance-optimizer.md`
- **Benchmark Command**: `.ai-config/commands/benchmark-performance.md`
