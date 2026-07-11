---
name: benchmark-performance
description: Run performance benchmarks for storage interface
shortcut: /benchmark
---

# Benchmark Performance Command

**Author**: <AUTHOR_NAME>  
**Date**: 2026-04-06

Execute comprehensive performance benchmarks for the Storage System storage interface.

## Usage

```bash
# Run all benchmarks
/benchmark

# Run specific benchmark
/benchmark --test=CreateVolume

# Run with profiling
/benchmark --profile

# Save results to file
/benchmark --output=benchmark_results.txt
```

## Command Execution

### Complete Benchmark Script
```bash
#!/bin/bash
# @author <AUTHOR_NAME>

set -e

OUTPUT=${1:-}

echo "=== Storage Interface Driver Performance Benchmarks ==="
echo "Date: $(date)"
echo ""

# 1. Go Benchmarks
echo "Step 1: Running Go benchmarks..."
go test -bench=. -benchmem ./pkg/driver > go_bench.txt
cat go_bench.txt

# Extract key metrics
CREATE_LATENCY=$(grep "BenchmarkCreateVolume" go_bench.txt | awk '{print $3}')
PUBLISH_LATENCY=$(grep "BenchmarkNodePublishVolume" go_bench.txt | awk '{print $3}')

echo ""
echo "CreateVolume latency: $CREATE_LATENCY"
echo "NodePublishVolume latency: $PUBLISH_LATENCY"

# 2. Volume Operation Benchmarks
echo ""
echo "Step 2: Volume operation benchmarks..."
./scripts/benchmark-volume-ops.sh

# 3. I/O Performance Tests
echo ""
echo "Step 3: I/O performance tests..."
./scripts/benchmark-io.sh

# 4. Stress Test
echo ""
echo "Step 4: Stress test (100 concurrent operations)..."
./scripts/benchmark-stress.sh

# 5. Generate Report
echo ""
echo "Step 5: Generating performance report..."
./scripts/generate-perf-report.sh

echo ""
echo "=== Benchmarks Complete ==="
echo "Results saved to: benchmark_results/"
```

## Benchmark Categories

### 1. Unit Benchmarks
```go
// @author <AUTHOR_NAME>
func BenchmarkCreateVolume(b *testing.B) {
    driver := setupBenchDriver()
    req := &csi.CreateVolumeRequest{
        Name: "bench-vol",
        CapacityRange: &csi.CapacityRange{
            RequiredBytes: 1073741824,
        },
    }
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        _, err := driver.CreateVolume(context.Background(), req)
        if err != nil {
            b.Fatal(err)
        }
    }
}
```

### 2. I/O Benchmarks
```bash
# Sequential write
fio --name=seq-write \
    --ioengine=libaio \
    --rw=write \
    --bs=1M \
    --size=10G \
    --numjobs=1 \
    --runtime=60 \
    --directory=/mnt/filesystem

# Sequential read
fio --name=seq-read \
    --ioengine=libaio \
    --rw=read \
    --bs=1M \
    --size=10G \
    --numjobs=1 \
    --runtime=60 \
    --directory=/mnt/filesystem

# Random 4K IOPS
fio --name=random-iops \
    --ioengine=libaio \
    --rw=randrw \
    --bs=4k \
    --size=1G \
    --numjobs=4 \
    --runtime=60 \
    --directory=/mnt/filesystem
```

### 3. Volume Operation Benchmarks
```bash
#!/bin/bash
# benchmark-volume-ops.sh
# @author <AUTHOR_NAME>

echo "Benchmarking volume operations..."

# Create 100 volumes
START=$(date +%s)
for i in {1..100}; do
    kubectl create -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: bench-pvc-$i
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: storage-system-fs
EOF
done

# Wait for all to be bound
kubectl wait --for=condition=Bound pvc --all --timeout=300s

END=$(date +%s)
DURATION=$((END - START))

echo "Created 100 volumes in ${DURATION}s"
echo "Average: $((DURATION / 100))s per volume"

# Cleanup
kubectl delete pvc --all
```

## Performance Targets

| Metric | Target | Acceptable |
|--------|--------|------------|
| CreateVolume latency | <500ms | <1s |
| DeleteVolume latency | <300ms | <800ms |
| PublishVolume latency | <2s | <5s |
| Sequential Write | >1 GB/s | >500 MB/s |
| Sequential Read | >1 GB/s | >500 MB/s |
| Random 4K IOPS | >100k | >50k |
| CPU usage (avg) | <30% | <50% |
| Memory usage | <200MB | <512MB |

## Expected Output

```
=== Storage Interface Driver Performance Benchmarks ===
Date: 2026-04-06 10:00:00

Step 1: Running Go benchmarks...
BenchmarkCreateVolume-8            2000    523147 ns/op    1024 B/op    10 allocs/op
BenchmarkDeleteVolume-8            3000    412895 ns/op     512 B/op     8 allocs/op
BenchmarkNodePublishVolume-8       1000   1894523 ns/op    2048 B/op    15 allocs/op

CreateVolume latency: 523147 ns/op
NodePublishVolume latency: 1894523 ns/op

Step 2: Volume operation benchmarks...
Created 100 volumes in 45s
Average: 0.45s per volume
✅ Within target (<1s)

Step 3: I/O performance tests...
Sequential Write: 1.2 GB/s ✅
Sequential Read: 1.5 GB/s ✅
Random 4K Write IOPS: 120k ✅
Random 4K Read IOPS: 150k ✅

Step 4: Stress test (100 concurrent operations)...
All 100 operations completed successfully
Average latency: 580ms
Max latency: 1.2s
✅ PASS

Step 5: Generating performance report...
Report saved to: benchmark_results/report_2026-04-06.md

=== Benchmarks Complete ===
Results saved to: benchmark_results/
```

## Profiling

### CPU Profile
```bash
# Run with CPU profiling
go test -cpuprofile=cpu.prof -bench=BenchmarkCreateVolume ./pkg/driver

# Analyze profile
go tool pprof -http=:8080 cpu.prof

# Generate flame graph
go tool pprof -svg cpu.prof > cpu_flame.svg
```

### Memory Profile
```bash
# Run with memory profiling
go test -memprofile=mem.prof -bench=. ./pkg/driver

# Analyze memory usage
go tool pprof -alloc_space mem.prof
go tool pprof -inuse_space mem.prof
```

## CI/CD Integration

### GitHub Actions
```yaml
- name: Performance Benchmarks
  run: |
    make benchmark
    
    # Compare with baseline
    go test -bench=. -benchmem ./... > new_bench.txt
    benchstat baseline_bench.txt new_bench.txt
```

## Reference Files

- **Performance Optimizer**: `.ai-config/agents/performance-optimizer.md`
- **Go Best Practices**: `.ai-config/skills/go-best-practices/SKILL.md`
