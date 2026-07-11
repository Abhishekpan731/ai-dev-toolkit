---
mode: manual
name: big-data-connector
description: big data connector development for object storage integration
tags: [spark, hadoop, java, red, infinia, s3, jni]
version: 1.0.0
---

# Spark/Hadoop Connector - RED/Infinia Integration

Expert knowledge for developing Spark connectors with Hadoop FileSystem API for object storage.

## Architecture Overview

### Component Stack
```
Spark Jobs
    ↓
Hadoop FileSystem API (org.apache.hadoop.fs.FileSystem)
    ↓
hadoop-red (RedFileSystem implementation - 48.5K LOC Java)
    ↓
red-java-sdk (S3-compatible SDK - 18.7K LOC Java)
    ↓
JNI Bridge (libred_clientJNI.so - 4.5K LOC C++)
    ↓
RED Dispatcher (native library)
    ↓
Infinia Cluster (storage backend)
```

### Key Components

**1. hadoop-red** (RedFileSystem)
- Implements `org.apache.hadoop.fs.FileSystem`
- Based on `hadoop-aws` S3A implementation pattern
- Adapter pattern: Hadoop API → RED S3-compatible storage
- Location: `spark-connector-poc/hadoop-red/hadoop-red/`

**2. red-java-sdk** 
- AWS S3 Java SDK-compatible interface
- JNI bridge to RED native library
- Location: `spark-connector-poc/red-java-sdk/red-java-sdk/`

## Development Environment

### Technology Stack
- **Hadoop**: 3.4.0
- **Java**: 21
- **Build**: Maven
- **Native**: C++17, JNI
- **Packaging**: DEB packages

### Build Commands
```bash
# hadoop-red
cd hadoop-red/hadoop-red
mvn clean package
# Output: target/hadoop-red-1.0.0.jar

# red-java-sdk (includes native compilation)
cd red-java-sdk/red-java-sdk
mvn clean package
# Output: 
#   native/target/libred_clientJNI.so
#   sdk/target/red-java-sdk-1.3.0.jar
```

## Core Patterns

### RedFileSystem Implementation

**Key Methods**:
```java
public class RedFileSystem extends FileSystem {
    // Initialize filesystem
    @Override
    public void initialize(URI uri, Configuration conf) throws IOException
    
    // Open file for reading
    @Override
    public FSDataInputStream open(Path f, int bufferSize) throws IOException
    
    // Create file for writing
    @Override
    public FSDataOutputStream create(Path f, FsPermission permission, 
        boolean overwrite, int bufferSize, short replication, 
        long blockSize, Progressable progress) throws IOException
    
    // Delete file/directory
    @Override
    public boolean delete(Path f, boolean recursive) throws IOException
    
    // Get file status
    @Override
    public FileStatus getFileStatus(Path f) throws IOException
    
    // List directory
    @Override
    public FileStatus[] listStatus(Path f) throws IOException
}
```

### Write Patterns

**Block Output Stream** (chunked writes):
```java
// RedBlockOutputStream - for large files
// Writes data in blocks, uploads to RED in chunks
public class RedBlockOutputStream extends OutputStream {
    private final BlockFactory blockFactory;
    private final WriteOperationHelper writeHelper;
    private Block activeBlock;
    
    @Override
    public void write(byte[] b, int off, int len) throws IOException {
        // Write to current block
        // Create new block when full
    }
    
    @Override
    public void close() throws IOException {
        // Upload remaining blocks
        // Commit transaction (Magic Commit Protocol)
    }
}
```

**Direct Output Stream** (direct writes):
```java
// RedDirectOutputStream - for small files
// Buffers entire file, single upload
public class RedDirectOutputStream extends OutputStream {
    private final ByteArrayOutputStream buffer;
    
    @Override
    public void close() throws IOException {
        // Upload buffered data in single operation
    }
}
```

### RED SDK Integration

**S3 Client Pattern**:
```java
// red-java-sdk follows AWS S3 SDK patterns
import com.datadirectnet.red.services.s3.S3Client;
import com.datadirectnet.red.services.s3.RedClient;
import com.datadirectnet.red.services.s3.model.*;

public class RedAdapter {
    private final S3Client redClient;
    
    public RedAdapter(Configuration conf) {
        this.redClient = RedClient.builder()
            .region(conf.get("fs.red.region"))
            .credentialsProvider(/* ... */)
            .build();
    }
    
    public void putObject(String bucket, String key, byte[] data) {
        PutObjectRequest request = PutObjectRequest.builder()
            .bucket(bucket)
            .key(key)
            .build();
        
        redClient.putObject(request, RequestBody.fromBytes(data));
    }
    
    public GetObjectResponse getObject(String bucket, String key) {
        GetObjectRequest request = GetObjectRequest.builder()
            .bucket(bucket)
            .key(key)
            .build();
        
        return redClient.getObject(request);
    }
}
```

### Transaction Commit (Magic Commit Protocol)

**Atomic Directory Operations**:
```java
// Ensures atomic commits for multi-file operations
public class MagicCommitProtocol {
    // Write to temporary location
    Path tempPath = new Path("s3://bucket/__magic/job-id/task-id/file");
    
    // Commit by renaming
    Path finalPath = new Path("s3://bucket/output/file");
    fs.rename(tempPath, finalPath);  // Atomic operation
}
```

## Configuration

### Hadoop Configuration
```xml
<!-- core-site.xml -->
<configuration>
    <property>
        <name>fs.red.impl</name>
        <value>org.apache.hadoop.fs.red.RedFileSystem</value>
    </property>
    
    <property>
        <name>fs.red.endpoint</name>
        <value>http://infinia-cluster:9000</value>
    </property>
    
    <property>
        <name>fs.red.access.key</name>
        <value>ACCESS_KEY</value>
    </property>
    
    <property>
        <name>fs.red.secret.key</name>
        <value>SECRET_KEY</value>
    </property>
    
    <!-- Buffer size for reads/writes -->
    <property>
        <name>fs.red.buffer.dir</name>
        <value>/tmp/hadoop-red-buffer</value>
    </property>
</configuration>
```

### Spark Configuration
```scala
val spark = SparkSession.builder()
    .appName("RedConnectorTest")
    .config("spark.hadoop.fs.red.impl", "org.apache.hadoop.fs.red.RedFileSystem")
    .config("spark.hadoop.fs.red.endpoint", "http://infinia-cluster:9000")
    .config("spark.hadoop.fs.red.access.key", "ACCESS_KEY")
    .getOrCreate()

// Read from RED
val df = spark.read.parquet("red://bucket/path/to/data")

// Write to RED
df.write.parquet("red://bucket/output/")
```

## Testing

### Unit Tests
```java
@Test
public void testRedFileSystemRead() throws IOException {
    Configuration conf = new Configuration();
    conf.set("fs.red.impl", "org.apache.hadoop.fs.red.RedFileSystem");
    
    RedFileSystem fs = new RedFileSystem();
    fs.initialize(URI.create("red://test-bucket"), conf);
    
    Path testFile = new Path("red://test-bucket/test.txt");
    try (FSDataInputStream in = fs.open(testFile)) {
        byte[] buffer = new byte[1024];
        int bytesRead = in.read(buffer);
        assertThat(bytesRead).isGreaterThan(0);
    }
}
```

### Integration Tests
```bash
# Run Spark benchmark with RED connector
cd hadoop-red/hadoop-red/benchmark
./run_sparkbench.sh

# Categories:
# - Micro: Sort, WordCount, TeraSort
# - ML: K-means, Bayes, LDA, PCA
# - SQL: TPC-DS queries
```

## Common Issues & Solutions

### ClassNotFoundException for RedFileSystem
```bash
# Ensure JAR in Spark classpath
export SPARK_CLASSPATH=/path/to/hadoop-red-1.0.0.jar:$SPARK_CLASSPATH

# Or in spark-defaults.conf
spark.driver.extraClassPath=/path/to/hadoop-red-1.0.0.jar
spark.executor.extraClassPath=/path/to/hadoop-red-1.0.0.jar
```

### JNI Library Not Found
```bash
# Set LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/path/to/red-java-sdk/native/target:$LD_LIBRARY_PATH

# Or Java library path
-Djava.library.path=/path/to/red-java-sdk/native/target
```

### Connection Refused to RED Dispatcher
```bash
# Check dispatcher service
systemctl status red-dispatcher

# Verify endpoint
curl http://infinia-cluster:9000/health
```

## Performance Tuning

### Buffer Sizes
```xml
<!-- Increase for better throughput -->
<property>
    <name>fs.red.io.buffer.size</name>
    <value>8388608</value>  <!-- 8 MB -->
</property>
```

### Block Size
```xml
<!-- Match RED cluster block size -->
<property>
    <name>fs.red.block.size</name>
    <value>134217728</value>  <!-- 128 MB -->
</property>
```

### Parallelism
```scala
// Increase Spark parallelism for RED I/O
spark.conf.set("spark.sql.shuffle.partitions", "200")
spark.conf.set("spark.default.parallelism", "200")
```

## Code Quality Standards

### Function Length
**Mandatory**: <50 lines per function (per .ai-config/rules/golang-standards.md)

```java
// ✅ GOOD: Extract helper methods
public FSDataOutputStream create(Path f, ...) throws IOException {
    validatePath(f);
    WriteOperationHelper helper = buildWriteHelper(f);
    return createOutputStream(f, helper);
}

// ❌ BAD: 100-line create() method
```

### Documentation
```java
/**
 * Opens file for reading from RED storage.
 * 
 * @author <AUTHOR_NAME>
 * @param f Path to file
 * @param bufferSize Buffer size for read operations
 * @return FSDataInputStream for reading file
 * @throws IOException if file not found or read fails
 */
@Override
public FSDataInputStream open(Path f, int bufferSize) throws IOException {
```

## References
- Hadoop FileSystem: https://hadoop.apache.org/docs/r3.4.0/api/org/apache/hadoop/fs/FileSystem.html
- AWS S3 SDK: https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/
- Project Docs: spark-connector-poc/SPARK_CONNECTOR_POC_REVIEW_TRACKER.md
