# MinIO 对象存储完全指南

---

## 目录

1. [基础概念](#1-基础概念)
2. [架构与高可用](#2-架构与高可用)
3. [功能使用](#3-功能使用)
4. [Java 集成](#4-java-集成)
5. [运维与监控](#5-运维与监控)

---

## 1. 基础概念

### 1.1 对象存储 vs 文件存储 vs 块存储

三种存储类型的关键差异：

| 维度 | 对象存储 (Object Storage) | 文件存储 (File Storage) | 块存储 (Block Storage) |
|------|--------------------------|------------------------|----------------------|
| 数据组织 | 扁平结构（Bucket + Object） | 树形目录层级 | 固定大小 Block |
| 访问协议 | HTTP/HTTPS（RESTful API） | NFS / CIFS / SMB | iSCSI / FC / NVMe-oF |
| 元数据 | 丰富自定义元数据 | 有限（文件名/时间/权限） | 无 |
| 扩展性 | 无限水平扩展 | 受限于单文件系统 | 受限于 SAN 架构 |
| 修改方式 | 整体替换（不可原地修改） | 随机读写 | 随机读写 |
| 典型延迟 | 较高（毫秒级） | 中（毫秒级） | 极低（微秒级） |
| 适用场景 | 备份/归档/大数据/静态文件 | 内部文档/共享目录 | 数据库/虚拟机磁盘 |
| 成本 | 低（廉价硬件） | 中 | 高（专用硬件） |
| 一致性 | 最终一致性 / 强一致（MinIO） | 强一致性 | 强一致性 |

```mermaid
flowchart TB
    subgraph Storage["三种存储类型对比"]
        direction TB

        OS["📦 对象存储
            ─────────
            Bucket → Object
            HTTP RESTful API
            无限扩展
            适合：图片/视频/备份
            不可原地修改"] 

        FS["📁 文件存储
            ─────────
            ／dir／file
            NFS / SMB
            树形结构
            适合：共享文档
            随机读写"]

        BS["💾 块存储
            ─────────
            Block 0..N
            iSCSI / FC
            低延迟
            适合：DB / VM
            随机读写"] 
    end

    User -->|"REST API"| OS
    User -->|"POSIX API"| FS
    Server -->|"SCSI/FC"| BS
```

### 1.2 S3 兼容 API 生态

Amazon S3 已成为对象存储的事实标准 API。MinIO 完全兼容 S3 API，这意味着任何支持 S3 的工具、SDK 均可直接使用：

- **SDK 层**：AWS SDK (Java/Python/Go/Node.js/.NET/Ruby/PHP/C++)
- **工具层**：AWS CLI、`mc` (MinIO Client)、s3cmd、rclone
- **框架层**：Spark、Flink、Presto/Trino、Hive 均内置 S3 Connector
- **备份工具**：Velero (K8s 备份)、Duplicati、restic
- **存储网关**：MinIO Gateway、S3FS-FUSE、JuiceFS

```mermaid
flowchart LR
    subgraph Apps["应用生态"]
        A1["Spark / Flink"]
        A2["Trino / Hive"]
        A3["K8s / Velero"]
        A4["备份工具"]
    end

    subgraph SDK["SDK"]
        S1["AWS SDK Java"]
        S2["AWS SDK Python"]
        S3["AWS SDK Go"]
    end

    subgraph API["S3 Compatible API"]
        OSS["阿里云 OSS"]
        MIO["☁️ MinIO"]
        CEPH["Ceph RGW"]
    end

    subgraph Tool["CLI 工具"]
        T1["awscli"]
        T2["mc"]
        T3["rclone"]
    end

    Apps --> SDK --> API
    Tool --> API
```

### 1.3 MinIO vs 主流存储方案

| 特性 | MinIO | FastDFS | HDFS | Ceph (RADOS) | 阿里云 OSS |
|------|-------|---------|------|-------------|----------|
| 协议 | S3 兼容 | 自定义 HTTP | HDFS RPC | S3 / CephFS | S3 兼容 |
| 一致性 | 读写强一致 | 最终一致 | 强一致 | 强一致 | 强一致 |
| 部署复杂度 | 极简（单二进制） | 中（Tracker+Storage） | 高（NameNode 主从） | 高（MON/OSD/MGR） | 托管服务 |
| 扩展方式 | Pool 水平扩展 | 扩容 Storage | 扩容 DataNode | 扩容 OSD | 自动 |
| 容错机制 | 纠删码 (N+M) | 多副本镜像 | 3 副本 | 多副本 / EC | 多副本 |
| 最小节点 | 4（生产推荐） | 2 | 3 | 3 | 托管 |
| 性能 | 极高（Go 原生） | 中 | 高（适合大文件） | 中 | 高 |
| 定位 | 私有云对象存储 | 轻量文件系统 | 大数据分析 | 统一存储平台 | 公有云对象存储 |

### 1.4 核心概念

| 术语 | 说明 |
|------|------|
| **Bucket** | 存储桶，对象的命名空间容器，类似文件系统的"根目录"，全局唯一 |
| **Object** | 数据对象，由数据 + 元数据 + 唯一 Key 组成 |
| **AccessKey** | 访问密钥 ID，类似用户名（16-20 字符） |
| **SecretKey** | 访问密钥密码，用于 HMAC-SHA1/SHA256 签名（40 字符） |
| **Endpoint** | MinIO 服务的 API 地址（如 `http://192.168.1.100:9000`） |
| **Region** | 区域标识（MinIO 默认为 `us-east-1`） |

> **Bucket 命名规则**：3-63 字符，小写字母/数字/连字符，不能以连字符开头或结尾，不能类似 IP 地址格式。

---

## 2. 架构与高可用

### 2.1 纠删码 (Erasure Coding)

MinIO 使用 Reed-Solomon 纠删码将对象切分为 **N 个数据块（Data Blocks）** + **M 个校验块（Parity Blocks）**，分布在多个驱动器上。当数据块丢失时，可通过校验块恢复。

**8 数据块 + 4 校验块 示例（N=8, M=4）：**

- 总驱动器数：N + M = 12
- 最大容错：M = 4 个驱动器（任意组合）
- 存储利用率：N / (N+M) = 8/12 ≈ 66.7%

```mermaid
flowchart TB
    subgraph Upload["上传对象"]
        Object["📄 对象文件 mydata.bin"]
    end

    subgraph Split["M = 4 校验块  N = 8 数据块"]
        D1["D1"] --- D2["D2"] --- D3["D3"] --- D4["D4"]
        D4 --- D5["D5"] --- D6["D6"] --- D7["D7"] --- D8["D8"]
        P1["P1"] --- P2["P2"] --- P3["P3"] --- P4["P4"]
    end

    subgraph Drives["12 块硬盘分布"]
        direction LR
        HD1["HD1<br/>D1"] HD2["HD2<br/>D2"] HD3["HD3<br/>D3"] HD4["HD4<br/>D4"]
        HD5["HD5<br/>D5"] HD6["HD6<br/>D6"] HD7["HD7<br/>D7"] HD8["HD8<br/>D8"]
        HD9["HD9<br/>P1"] HD10["HD10<br/>P2"] HD11["HD11<br/>P3"] HD12["HD12<br/>P4"]
    end

    subgraph Fault["任意损坏 4 块可恢复"]
        X1["❌ HD2 损坏"] X2["❌ HD5 损坏"]
        X3["❌ HD9 损坏"] X4["❌ HD12 损坏"]
    end

    subgraph Recover["Reed-Solomon 重建"]
        R1["D1 D3 D4 D6 D7 D8<br/>P2 P3"]
        R2["→ 计算出 D2 D5 P1 P4"]
        R3["→ 完整恢复对象"]
    end

    Object --> Split --> Drives
    Drives -.-> Fault -.-> Recover
```

**纠删码优劣对比：**

| 配置 | 存储利用率 | 容错能力 | 推荐场景 |
|------|-----------|---------|---------|
| 6+2 | 75% | 2 盘 | 开发测试 |
| 8+4 | 66.7% | 4 盘 | 生产环境 |
| 10+2 | 83.3% | 2 盘 | 性能优先 |
| 12+4 | 75% | 4 盘 | 大容量生产 |

### 2.2 Bit Rot Protection（位衰减保护）

MinIO 在写入时对每个对象计算 **SHA-256 哈希** 并加密存储元数据中；读取时重新计算哈希并与元数据对比，发现数据损坏时自动通过纠删码修复。

```mermaid
flowchart LR
    Write["写入"] --> Hash["SHA-256<br/>计算哈希"] --> Store["存储数据块 + 哈希"]
    Read["读取"] --> Verify["重新计算哈希"] --> Match{"哈希匹配？"}
    Match -->|"✅ 一致"| Return["返回数据"]
    Match -->|"❌ 不一致"| Repair["从校验块恢复"]
```

### 2.3 分布式部署要求

| 要求 | 说明 |
|------|------|
| **最少节点** | 4 节点（生产环境强烈建议） |
| **每节点硬盘** | 至少 2 块（本地 JBOD，不支持 RAID） |
| **总硬盘数** | 最少 4 块，且需满足 N+M ≥ 4 且 N+M 为偶数 |
| **负载均衡** | Nginx / HAProxy / 云 LB 前置分发 |
| **网络** | 节点间 10GbE 以上推荐 |
| **时间同步** | NTP 必须配置 |

**Nginx 负载均衡示例配置：**

```nginx
upstream minio_cluster {
    least_conn;
    server 192.168.1.10:9000 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:9000 max_fails=3 fail_timeout=30s;
    server 192.168.1.12:9000 max_fails=3 fail_timeout=30s;
    server 192.168.1.13:9000 max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    server_name minio.example.com;

    # 客户端上传不限制大小
    client_max_body_size 0;

    location / {
        proxy_pass http://minio_cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    location /console {
        # Console UI
        proxy_pass http://minio_console;
    }
}
```

### 2.4 数据加密

MinIO 支持三种加密模式：

| 模式 | 密钥管理 | 说明 |
|------|---------|------|
| **SSE-C** | 客户端提供密钥 | 每次请求携带 AES-256 密钥，MinIO 不做持久化 |
| **SSE-S3** | MinIO KMS 管理 | 自动加密，透明到客户端 |
| **SSE-KMS** | 外部 KMS（KES） | 集成 AWS KMS / HashiCorp Vault / Gemalto |

```mermaid
flowchart TB
    subgraph SSE-C["SSE-C 客户端密钥"]
        C1["客户端请求：PUT /bucket/obj<br/>x-amz-server-side-encryption-customer-key: xxx"]
        C2["MinIO 使用客户端密钥加密数据"]
        C3["不持久化密钥，丢失即不可恢复"]
    end

    subgraph SSE-S3["SSE-S3 MinIO 托管"]
        S1["MinIO 自动生成密钥"]
        S2["每对象独立 DEK + KEK 加密"]
        S3["客户端无感知"]
    end

    subgraph SSE-KMS["SSE-KMS 外部 KMS"]
        K1["KES 服务 ←→ 外部 KMS"]
        K2["通过 KES 获取密钥加密"]
        K3["支持密钥轮换/审计"]
    end
```

**Java SSE-C 加密上传/下载示例：**

```java
import io.minio.*;
import io.minio.http.Method;

public class SSECExample {
    public static void main(String[] args) throws Exception {
        MinioClient client = MinioClient.builder()
                .endpoint("http://127.0.0.1:9000")
                .credentials("minioadmin", "minioadmin")
                .build();

        // 客户端自定义 AES-256 密钥（32 字节）
        SecretKey key = new SecretKeySpec(
            "0123456789abcdef0123456789abcdef".getBytes(), "AES"
        );
        ServerSideEncryptionCustomerKey ssec =
            new ServerSideEncryptionCustomerKey(key);

        // SSE-C 加密上传
        client.putObject(
            PutObjectArgs.builder()
                .bucket("my-bucket")
                .object("secret.pdf")
                .stream(new FileInputStream("secret.pdf"), -1, 10485760)
                .ssec(ssec)
                .build()
        );

        // SSE-C 解密下载
        try (InputStream is = client.getObject(
            GetObjectArgs.builder()
                .bucket("my-bucket")
                .object("secret.pdf")
                .ssec(ssec)
                .build())) {
            Files.copy(is, Paths.get("downloaded.pdf"));
        }
    }
}
```

### 2.5 版本控制 (Versioning)

版本控制保留对象的多个版本，防止误删除和覆盖：

```mermaid
flowchart TB
    subgraph NoVer["无版本控制"]
        N1["PUT myfile.txt"] --> N2["覆盖，旧版本丢失"]
        N3["DELETE myfile.txt"] --> N4["永久删除"]
    end

    subgraph WithVer["启用版本控制"]
        W1["PUT myfile.txt v1"] --> W2["PUT myfile.txt v2"]
        W2 --> W3["不同版本共存"]
        W4["DELETE myfile.txt"] --> W5["添加删除标记<br/>（Delete Marker）"]
        W6["可回滚到 v1"] --> W7["删除标记可移除"]
    end
```

**启用版本控制 (Java)：**

```java
import io.minio.*;

MinioClient client = MinioClient.builder()
    .endpoint("http://127.0.0.1:9000")
    .credentials("minioadmin", "minioadmin")
    .build();

// 开启版本控制
client.setBucketVersioning(
    SetBucketVersioningArgs.builder()
        .bucket("my-bucket")
        .config(new VersioningConfig(VersioningConfig.Status.ENABLED))
        .build()
);
```

### 2.6 对象锁定 WORM（Object Lock）

WORM（Write Once Read Many）模式确保对象在指定保留期内不可删除或覆盖。适用于合规归档（如 SEC Rule 17a-4）。

| 模式 | 说明 |
|------|------|
| **Governance Mode** | 有特殊权限可覆盖，适合内部合规 |
| **Compliance Mode** | 绝对不可删除/覆盖，直到保留期结束 |
| **Legal Hold** | 无限期锁定，直到移除标记 |

```java
import io.minio.*;

MinioClient client = MinioClient.builder()
    .endpoint("http://127.0.0.1:9000")
    .credentials("minioadmin", "minioadmin")
    .build();

// 创建启用 Lock 的 Bucket
client.makeBucket(
    MakeBucketArgs.builder()
        .bucket("locked-bucket")
        .objectLock(true)  // 必须开启
        .build()
);

// 上传并锁定对象（Compliance Mode，保留 10 天）
client.putObject(
    PutObjectArgs.builder()
        .bucket("locked-bucket")
        .object("archive.log")
        .stream(new FileInputStream("archive.log"), -1, 10485760)
        .retention(
            new Retention(
                RetentionMode.COMPLIANCE,
                RetentionDuration.parse("10d")
            )
        )
        .build()
);

// 添加 Legal Hold
client.setObjectLegalHold(
    SetObjectLegalHoldArgs.builder()
        .bucket("locked-bucket")
        .object("archive.log")
        .legalHold(true)
        .build()
);
```

---

## 3. 功能使用

### 3.1 分片上传 (Multipart Upload)

大文件（超过 64MB）应使用 Multipart Upload 以提高吞吐量和断点续传能力。MinIO 每个分片 5MiB~5GiB，最大 5GiB 对象，但推荐 16~64MiB 分片。

```mermaid
sequenceDiagram
    participant Client as 客户端
    participant MinIO as MinIO Server
    
    Client->>MinIO: 1. initiateMultipartUpload (Bucket, Object)
    MinIO-->>Client: UploadId
    
    Client->>MinIO: 2. uploadPart (UploadId, PartNumber=1)
    MinIO-->>Client: ETag-1
    
    Client->>MinIO: 3. uploadPart (UploadId, PartNumber=2)
    MinIO-->>Client: ETag-2
    
    Client->>MinIO: 4. uploadPart (UploadId, PartNumber=N)
    MinIO-->>Client: ETag-N
    
    Client->>MinIO: 5. completeMultipartUpload (UploadId, [ETag-1, ETag-2, ..., ETag-N])
    MinIO-->>Client: ✅ 对象合并完成
```

**Java 分片上传代码：**

```java
import io.minio.*;
import io.minio.messages.Part;
import java.util.*;

public class MultipartUploadExample {
    public static void main(String[] args) throws Exception {
        MinioClient client = MinioClient.builder()
                .endpoint("http://127.0.0.1:9000")
                .credentials("minioadmin", "minioadmin")
                .build();

        String bucket = "my-bucket";
        String object = "large-file.bin";
        File file = new File("large-file.bin");
        long fileSize = file.length();
        long partSize = 64 * 1024 * 1024; // 64MB 每分片

        // 1. 创建分片上传
        String uploadId = client.createMultipartUpload(
            CreateMultipartUploadArgs.builder()
                .bucket(bucket)
                .object(object)
                .build()
        );

        List<Part> parts = new ArrayList<>();
        int partNumber = 1;
        long offset = 0;

        try (RandomAccessFile raf = new RandomAccessFile(file, "r")) {
            while (offset < fileSize) {
                long size = Math.min(partSize, fileSize - offset);
                byte[] data = new byte[(int) size];
                raf.readFully(data);

                // 2. 上传分片
                UploadPartResponse response = client.uploadPart(
                    UploadPartArgs.builder()
                        .bucket(bucket)
                        .object(object)
                        .uploadId(uploadId)
                        .partNumber(partNumber)
                        .stream(new ByteArrayInputStream(data), size, -1)
                        .build()
                );

                parts.add(new Part(partNumber++, response.etag()));
                offset += size;
                System.out.printf("Part %d uploaded (%d/%d)%n",
                    partNumber - 1, offset, fileSize);
            }
        }

        // 3. 完成分片上传
        client.completeMultipartUpload(
            CompleteMultipartUploadArgs.builder()
                .bucket(bucket)
                .object(object)
                .uploadId(uploadId)
                .parts(parts)
                .build()
        );

        System.out.println("✅ Multipart upload completed");
    }
}
```

### 3.2 预签名 URL (Presigned URL)

预签名 URL 允许临时授权访问私有对象，无需暴露 AccessKey/SecretKey，适用于文件分享、上传授权等场景。

```mermaid
flowchart LR
    subgraph App["应用服务器"]
        MC["MinioClient<br/>生成 Presigned URL"]
    end

    subgraph Client["客户端（浏览器/App）"]
        UPL["上传文件"] DNL["下载文件"]
    end

    subgraph MIO["MinIO"]
        O["Object Storage"]
    end

    MC -->|"PUT Presigned URL<br/>(有效期 5 分钟)"| Client
    MC -->|"GET Presigned URL<br/>(有效期 1 小时)"| Client
    UPL --> O
    DNL --> O
```

**生成 Presigned URL (Java)：**

```java
import io.minio.*;
import io.minio.http.Method;
import java.util.concurrent.TimeUnit;

public class PresignedUrlExample {
    public static void main(String[] args) throws Exception {
        MinioClient client = MinioClient.builder()
                .endpoint("http://127.0.0.1:9000")
                .credentials("minioadmin", "minioadmin")
                .build();

        // 生成 GET 下载预签名 URL（1 小时内有效）
        String getUrl = client.getPresignedObjectUrl(
            GetPresignedObjectUrlArgs.builder()
                .method(Method.GET)
                .bucket("my-bucket")
                .object("report.pdf")
                .expiry(1, TimeUnit.HOURS)
                .build()
        );
        System.out.println("Download URL: " + getUrl);

        // 生成 PUT 上传预签名 URL（5 分钟内有效）
        String putUrl = client.getPresignedObjectUrl(
            GetPresignedObjectUrlArgs.builder()
                .method(Method.PUT)
                .bucket("my-bucket")
                .object("user-upload.jpg")
                .expiry(5, TimeUnit.MINUTES)
                .build()
        );
        System.out.println("Upload URL: " + putUrl);

        // 携带请求头条件（如 Content-Type 限制）
        Map<String, String> headers = new HashMap<>();
        headers.put("Content-Type", "image/jpeg");
        String restrictedUrl = client.getPresignedObjectUrl(
            GetPresignedObjectUrlArgs.builder()
                .method(Method.PUT)
                .bucket("my-bucket")
                .object("avatar.jpg")
                .expiry(10, TimeUnit.MINUTES)
                .extraHeaders(headers)
                .build()
        );
    }
}
```

### 3.3 事件通知 (Event Notification)

MinIO 支持将 Bucket 事件推送到多种目标系统：

| 目标 | 配置方式 | 典型场景 |
|------|---------|---------|
| Webhook | HTTP POST | 实时处理上传文件 |
| Kafka | Avro/JSON | 流式数据管道 |
| AMQP (RabbitMQ) | Exchange/Routing Key | 任务队列 |
| Redis | Pub/Sub/Keyspace | 缓存失效/实时通知 |
| PostgreSQL | SQL INSERT | 审计日志 |
| Elasticsearch | Index API | 全文检索 |

```mermaid
flowchart TB
    subgraph MinIOCluster["MinIO Cluster"]
        B1["Bucket A"] B2["Bucket B"]
        EV["Event Manager"]
    end

    subgraph Targets["事件目标"]
        WH["Webhook<br/>HTTP POST"]
        KFK["Kafka"]
        MQ["RabbitMQ / AMQP"]
        RS["Redis Pub/Sub"]
        PG["PostgreSQL"]
        ES["Elasticsearch"]
    end

    B1 -- "s3:ObjectCreated:*" --> EV
    B2 -- "s3:ObjectRemoved:*" --> EV
    EV --> WH
    EV --> KFK
    EV --> MQ
    EV --> RS
    EV --> PG
    EV --> ES
```

**配置事件通知到 Webhook (`mc admin config set`)：**

```bash
# MinIO 服务端配置 webhook
mc admin config set myminio notify_webhook:upload \
    endpoint="http://webhook.example.com/events" \
    auth_token="my-secret-token"

# 重启生效
mc admin service restart myminio

# 设置 Bucket 事件规则
mc event add myminio/my-bucket \
    arn:minio:sqs::upload:webhook \
    --event put,delete
```

### 3.4 生命周期管理 (Lifecycle Management)

自动管理对象的过期删除和存储类迁移：

```xml
<!-- 生命周期规则 XML 配置 -->
<LifecycleConfiguration>
  <Rule>
    <ID>expire-logs</ID>
    <Status>Enabled</Status>
    <Filter>
      <Prefix>logs/</Prefix>
    </Filter>
    <Expiration>
      <Days>30</Days>  <!-- 30 天后自动删除 -->
    </Expiration>
  </Rule>
  <Rule>
    <ID>transition-temp</ID>
    <Status>Enabled</Status>
    <Filter>
      <Prefix>temp/</Prefix>
    </Filter>
    <Transition>
      <Days>7</Days>
      <StorageClass>GLACIER</StorageClass>
    </Transition>
  </Rule>
</LifecycleConfiguration>
```

**Java 配置生命周期：**

```java
import io.minio.*;
import io.minio.messages.lifecycle.*;

List<LifecycleRule> rules = List.of(
    new LifecycleRule(
        LifecycleRuleStatus.ENABLED,
        null,
        new LifecycleExpiration(30, null), // 30 天过期
        new LifecycleRuleFilter(new LifecycleRulePrefix("logs/")),
        "expire-logs",
        null,
        null,
        null
    )
);

LifecycleConfiguration config = new LifecycleConfiguration(rules);
client.setBucketLifecycle(
    SetBucketLifecycleArgs.builder()
        .bucket("my-bucket")
        .config(config)
        .build()
);
```

### 3.5 桶复制 (Bucket Replication)

跨集群自动同步数据，实现异地容灾或全球加速：

```mermaid
flowchart LR
    subgraph SiteA["数据中心 A（主）"]
        BA["Bucket: primary-bucket"]
    end

    subgraph SiteB["数据中心 B（从）"]
        BB["Bucket: replica-bucket"]
    end

    subgraph SiteC["数据中心 C（从）"]
        BC["Bucket: replica-bucket"]
    end

    BA -->|"同步复制<br/>（版本需开启）"| BB
    BA -->|"同步复制"| BC
    BB -->|"复制回源（可选）"| BA
```

**配置桶复制：**

```bash
# 在源集群创建 replication 角色
mc admin policy create src minio-replication replication-policy.json

# 设置远程目标
mc replicate add myminio/primary-bucket \
    --remote "https://dest.example.com:9000/replica-bucket" \
    --access-key "ACCESSKEY" \
    --secret-key "SECRETKEY" \
    --sync
```

### 3.6 静态网站托管

MinIO 可直接作为静态网站服务器：

```java
import io.minio.*;

// 启用 Bucket 为静态网站
client.setBucketPolicy(
    SetBucketPolicyArgs.builder()
        .bucket("web-bucket")
        .config(
            """
            {
                "Version": "2012-10-17",
                "Statement": [{
                    "Effect": "Allow",
                    "Principal": {"AWS": ["*"]},
                    "Action": ["s3:GetObject"],
                    "Resource": ["arn:aws:s3:::web-bucket/*"]
                }]
            }
            """
        )
        .build()
);

// 配置索引页和错误页
BucketWebsiteConfiguration websiteConfig =
    new BucketWebsiteConfiguration("index.html", "error.html");
client.setBucketWebsite(
    SetBucketWebsiteArgs.builder()
        .bucket("web-bucket")
        .config(websiteConfig)
        .build()
);
```

通过 `http://minio-endpoint/web-bucket/` 即可访问静态网站。

### 3.7 IAM 权限控制

MinIO 支持类似 AWS IAM 的 Policy-Based 权限控制：

**Policy 示例（只读访问特定 Bucket 的特定前缀）：**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket"
      ],
      "Condition": {
        "StringLike": {
          "s3:prefix": "public/*"
        }
      }
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:GetObjectVersion"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket/public/*"
      ]
    }
  ]
}
```

**常用 Action 列表：**

| Action | 描述 |
|--------|------|
| `s3:GetObject` | 下载对象 |
| `s3:PutObject` | 上传对象 |
| `s3:DeleteObject` | 删除对象 |
| `s3:ListBucket` | 列出对象 |
| `s3:GetBucketVersioning` | 查看版本控制 |
| `s3:PutBucketPolicy` | 修改 Bucket 策略 |
| `s3:GetObjectRetention` | 查看保留设置 |
| `s3:*` | 所有操作（管理员） |

---

## 4. Java 集成

### 4.1 MinioClient 创建

**SDK Maven 依赖：**

```xml
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>8.5.12</version>
</dependency>
```

**多种客户端创建方式：**

```java
// 方式一：基础配置
MinioClient client = MinioClient.builder()
    .endpoint("http://127.0.0.1:9000")
    .credentials("minioadmin", "minioadmin")
    .build();

// 方式二：HTTPS + 自定义 Region
MinioClient client2 = MinioClient.builder()
    .endpoint("https://minio.example.com")
    .credentials("AKIAIOSFODNN7EXAMPLE", "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY")
    .region("cn-north-1")
    .build();

// 方式三：自定义 HTTP Client（如配置超时）
MinioClient client3 = MinioClient.builder()
    .endpoint("http://127.0.0.1:9000")
    .credentials("minioadmin", "minioadmin")
    .httpClient(
        OkHttpClient.Builder()
            .connectTimeout(10, TimeUnit.SECONDS)
            .readTimeout(60, TimeUnit.SECONDS)
            .writeTimeout(60, TimeUnit.SECONDS)
            .build()
    )
    .build();
```

### 4.2 基本 CRUD 操作

```java
import io.minio.*;
import io.minio.errors.*;
import io.minio.messages.*;
import java.io.*;
import java.nio.file.*;

public class MinioCRUD {
    private final MinioClient client;

    public MinioCRUD() {
        this.client = MinioClient.builder()
                .endpoint("http://127.0.0.1:9000")
                .credentials("minioadmin", "minioadmin")
                .build();
    }

    /**
     * 存储桶是否存在
     */
    public boolean bucketExists(String bucket) throws Exception {
        return client.bucketExists(
            BucketExistsArgs.builder().bucket(bucket).build()
        );
    }

    /**
     * 创建存储桶
     */
    public void createBucket(String bucket) throws Exception {
        if (!bucketExists(bucket)) {
            client.makeBucket(
                MakeBucketArgs.builder().bucket(bucket).build()
            );
        }
    }

    /**
     * 上传文件
     */
    public void putObject(String bucket, String object, String filePath)
            throws Exception {
        client.putObject(
            PutObjectArgs.builder()
                .bucket(bucket)
                .object(object)
                .stream(
                    new FileInputStream(filePath),
                    new File(filePath).length(),
                    -1  // 不限制分片大小
                )
                .contentType(Files.probeContentType(Paths.get(filePath)))
                .build()
        );
    }

    /**
     * 上传字节流
     */
    public void putObject(String bucket, String object,
                          byte[] data, String contentType) throws Exception {
        client.putObject(
            PutObjectArgs.builder()
                .bucket(bucket)
                .object(object)
                .stream(new ByteArrayInputStream(data), data.length, -1)
                .contentType(contentType)
                .build()
        );
    }

    /**
     * 下载到本地文件
     */
    public void getObject(String bucket, String object,
                          String downloadPath) throws Exception {
        try (InputStream stream = client.getObject(
            GetObjectArgs.builder()
                .bucket(bucket)
                .object(object)
                .build())) {
            Files.copy(stream, Paths.get(downloadPath),
                StandardCopyOption.REPLACE_EXISTING);
        }
    }

    /**
     * 读取对象为字符串
     */
    public String getObjectAsString(String bucket, String object)
            throws Exception {
        try (InputStream stream = client.getObject(
            GetObjectArgs.builder()
                .bucket(bucket)
                .object(object)
                .build())) {
            return new String(stream.readAllBytes(), StandardCharsets.UTF_8);
        }
    }

    /**
     * 获取对象元数据
     */
    public StatObjectResponse statObject(String bucket, String object)
            throws Exception {
        return client.statObject(
            StatObjectArgs.builder()
                .bucket(bucket)
                .object(object)
                .build()
        );
    }

    /**
     * 删除对象
     */
    public void removeObject(String bucket, String object) throws Exception {
        client.removeObject(
            RemoveObjectArgs.builder()
                .bucket(bucket)
                .object(object)
                .build()
        );
    }

    /**
     * 批量删除对象
     */
    public void removeObjects(String bucket, List<String> objects)
            throws Exception {
        List<DeleteObject> deleteObjects = objects.stream()
            .map(DeleteObject::new)
            .collect(Collectors.toList());

        Iterable<Result<DeleteError>> results =
            client.removeObjects(
                RemoveObjectsArgs.builder()
                    .bucket(bucket)
                    .objects(deleteObjects)
                    .build()
            );

        for (Result<DeleteError> result : results) {
            DeleteError error = result.get();
            System.err.println("Delete failed: " + error.objectName()
                + " - " + error.message());
        }
    }

    /**
     * 列出存储桶
     */
    public List<Bucket> listBuckets() throws Exception {
        return client.listBuckets();
    }

    /**
     * 列出 Bucket 中的对象
     */
    public void listObjects(String bucket, String prefix)
            throws Exception {
        Iterable<Result<Item>> results = client.listObjects(
            ListObjectsArgs.builder()
                .bucket(bucket)
                .prefix(prefix)
                .recursive(true)
                .build()
        );

        for (Result<Item> result : results) {
            Item item = result.get();
            System.out.printf("%s  %s  %d bytes%n",
                item.lastModified(), item.objectName(), item.size());
        }
    }
}
```

### 4.3 Spring Boot 整合 MinIO

**依赖 (`pom.xml`)：**

```xml
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>8.5.12</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

**配置类 (`application.yml`)：**

```yaml
minio:
  endpoint: http://127.0.0.1:9000
  access-key: minioadmin
  secret-key: minioadmin
  bucket: default-bucket
  # 可选配置
  region: us-east-1
  connect-timeout: 10s
  read-timeout: 60s
  write-timeout: 60s
```

**属性绑定类：**

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "minio")
public class MinioProperties {
    private String endpoint;
    private String accessKey;
    private String secretKey;
    private String bucket;
    private String region = "us-east-1";
    private Duration connectTimeout = Duration.ofSeconds(10);
    private Duration readTimeout = Duration.ofSeconds(60);
    private Duration writeTimeout = Duration.ofSeconds(60);

    // getters / setters
    public String getEndpoint() { return endpoint; }
    public void setEndpoint(String endpoint) { this.endpoint = endpoint; }
    public String getAccessKey() { return accessKey; }
    public void setAccessKey(String accessKey) { this.accessKey = accessKey; }
    public String getSecretKey() { return secretKey; }
    public void setSecretKey(String secretKey) { this.secretKey = secretKey; }
    public String getBucket() { return bucket; }
    public void setBucket(String bucket) { this.bucket = bucket; }
    public String getRegion() { return region; }
    public void setRegion(String region) { this.region = region; }
    public Duration getConnectTimeout() { return connectTimeout; }
    public void setConnectTimeout(Duration connectTimeout) {
        this.connectTimeout = connectTimeout;
    }
    public Duration getReadTimeout() { return readTimeout; }
    public void setReadTimeout(Duration readTimeout) {
        this.readTimeout = readTimeout;
    }
    public Duration getWriteTimeout() { return writeTimeout; }
    public void setWriteTimeout(Duration writeTimeout) {
        this.writeTimeout = writeTimeout;
    }
}
```

**配置类：**

```java
import io.minio.MinioClient;
import okhttp3.OkHttpClient;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import java.util.concurrent.TimeUnit;

@Configuration
@EnableConfigurationProperties(MinioProperties.class)
public class MinioConfig {

    @Bean
    public MinioClient minioClient(MinioProperties properties) {
        OkHttpClient httpClient = new OkHttpClient.Builder()
            .connectTimeout(
                properties.getConnectTimeout().toMillis(), TimeUnit.MILLISECONDS)
            .readTimeout(
                properties.getReadTimeout().toMillis(), TimeUnit.MILLISECONDS)
            .writeTimeout(
                properties.getWriteTimeout().toMillis(), TimeUnit.MILLISECONDS)
            .build();

        return MinioClient.builder()
            .endpoint(properties.getEndpoint())
            .credentials(properties.getAccessKey(), properties.getSecretKey())
            .region(properties.getRegion())
            .httpClient(httpClient)
            .build();
    }
}
```

**工具类：**

```java
import io.minio.*;
import io.minio.http.Method;
import io.minio.messages.Bucket;
import io.minio.messages.Item;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;
import java.io.InputStream;
import java.util.*;
import java.util.concurrent.TimeUnit;

@Component
public class MinioUtil {

    private final MinioClient client;
    private final MinioProperties properties;

    public MinioUtil(MinioClient client, MinioProperties properties) {
        this.client = client;
        this.properties = properties;
    }

    /**
     * 是否存在桶
     */
    public boolean bucketExists(String bucket) {
        try {
            return client.bucketExists(
                BucketExistsArgs.builder().bucket(bucket).build());
        } catch (Exception e) {
            throw new RuntimeException("MinIO bucket check failed", e);
        }
    }

    /**
     * 创建桶
     */
    public void createBucket(String bucket) {
        try {
            if (!bucketExists(bucket)) {
                client.makeBucket(
                    MakeBucketArgs.builder().bucket(bucket).build());
            }
        } catch (Exception e) {
            throw new RuntimeException("MinIO create bucket failed", e);
        }
    }

    /**
     * 上传 Multipart 文件
     */
    public String uploadFile(String bucket, String object,
                             MultipartFile file) {
        try {
            client.putObject(
                PutObjectArgs.builder()
                    .bucket(bucket)
                    .object(object)
                    .stream(file.getInputStream(), file.getSize(), -1)
                    .contentType(file.getContentType())
                    .build()
            );
            return object;
        } catch (Exception e) {
            throw new RuntimeException("MinIO upload failed", e);
        }
    }

    /**
     * 上传文件（默认桶）
     */
    public String uploadFile(String object, MultipartFile file) {
        return uploadFile(properties.getBucket(), object, file);
    }

    /**
     * 下载文件流
     */
    public InputStream downloadFile(String bucket, String object) {
        try {
            return client.getObject(
                GetObjectArgs.builder()
                    .bucket(bucket)
                    .object(object)
                    .build()
            );
        } catch (Exception e) {
            throw new RuntimeException("MinIO download failed", e);
        }
    }

    /**
     * 删除文件
     */
    public void deleteFile(String bucket, String object) {
        try {
            client.removeObject(
                RemoveObjectArgs.builder()
                    .bucket(bucket)
                    .object(object)
                    .build()
            );
        } catch (Exception e) {
            throw new RuntimeException("MinIO delete failed", e);
        }
    }

    /**
     * 生成文件访问 URL（GET，默认 7 天有效）
     */
    public String getPreviewUrl(String bucket, String object,
                                int expiryDays) {
        try {
            return client.getPresignedObjectUrl(
                GetPresignedObjectUrlArgs.builder()
                    .method(Method.GET)
                    .bucket(bucket)
                    .object(object)
                    .expiry(expiryDays, TimeUnit.DAYS)
                    .build()
            );
        } catch (Exception e) {
            throw new RuntimeException("MinIO presigned URL failed", e);
        }
    }

    /**
     * 生成上传 URL（PUT）
     */
    public String getUploadUrl(String bucket, String object,
                               int expiryMinutes) {
        try {
            return client.getPresignedObjectUrl(
                GetPresignedObjectUrlArgs.builder()
                    .method(Method.PUT)
                    .bucket(bucket)
                    .object(object)
                    .expiry(expiryMinutes, TimeUnit.MINUTES)
                    .build()
            );
        } catch (Exception e) {
            throw new RuntimeException("MinIO presigned upload URL failed", e);
        }
    }

    /**
     * 列出桶内所有文件
     */
    public List<String> listFiles(String bucket, String prefix) {
        List<String> files = new ArrayList<>();
        try {
            Iterable<Result<Item>> results = client.listObjects(
                ListObjectsArgs.builder()
                    .bucket(bucket)
                    .prefix(prefix)
                    .recursive(true)
                    .build()
            );
            for (Result<Item> result : results) {
                files.add(result.get().objectName());
            }
        } catch (Exception e) {
            throw new RuntimeException("MinIO list files failed", e);
        }
        return files;
    }

    /**
     * 获取文件元数据
     */
    public Map<String, String> getFileMetadata(String bucket, String object) {
        try {
            StatObjectResponse stat = client.statObject(
                StatObjectArgs.builder()
                    .bucket(bucket)
                    .object(object)
                    .build()
            );
            Map<String, String> meta = new HashMap<>();
            meta.put("size", String.valueOf(stat.size()));
            meta.put("etag", stat.etag());
            meta.put("contentType", stat.contentType());
            meta.put("lastModified", stat.lastModified().toString());
            return meta;
        } catch (Exception e) {
            throw new RuntimeException("MinIO stat failed", e);
        }
    }
}
```

**REST Controller 示例：**

```java
import org.springframework.core.io.InputStreamResource;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

@RestController
@RequestMapping("/files")
public class FileController {

    private final MinioUtil minioUtil;

    public FileController(MinioUtil minioUtil) {
        this.minioUtil = minioUtil;
    }

    @PostMapping("/upload")
    public String upload(@RequestParam("file") MultipartFile file) {
        String object = UUID.randomUUID() + "_" + file.getOriginalFilename();
        return minioUtil.uploadFile(object, file);
    }

    @GetMapping("/download")
    public ResponseEntity<InputStreamResource> download(
            @RequestParam String object) {
        InputStreamResource resource =
            new InputStreamResource(minioUtil.downloadFile("default", object));
        return ResponseEntity.ok()
            .contentType(MediaType.APPLICATION_OCTET_STREAM)
            .body(resource);
    }

    @DeleteMapping
    public void delete(@RequestParam String object) {
        minioUtil.deleteFile("default", object);
    }

    @GetMapping("/preview-url")
    public String previewUrl(@RequestParam String object) {
        return minioUtil.getPreviewUrl("default", object, 7);
    }
}
```

### 4.4 大文件分片上传（前后端分离）

```mermaid
sequenceDiagram
    participant FE as 前端浏览器
    participant BE as 后端 Java
    participant M as MinIO

    FE->>BE: 1. POST /init-upload (filename)
    BE->>M: initiateMultipartUpload
    M-->>BE: UploadId
    BE-->>FE: { uploadId, chunkSize }

    loop 对于每个分片
        FE->>BE: 2. POST /upload-part (uploadId, partNumber, chunk)
        BE->>M: uploadPart
        M-->>BE: ETag
        BE-->>FE: { etag, partNumber }
    end

    FE->>BE: 3. POST /complete-upload (uploadId, parts[])
    BE->>M: completeMultipartUpload
    M-->>BE: ✅ 合并完成
    BE-->>FE: { finalObjectUrl }
```

**后端实现：**

```java
import io.minio.*;
import io.minio.messages.Part;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

@RestController
@RequestMapping("/minio/multipart")
public class MultipartUploadController {

    private final MinioClient client;
    private final Map<String, List<Part>> uploadCache = new ConcurrentHashMap<>();

    public MultipartUploadController(MinioClient client) {
        this.client = client;
    }

    /**
     * 1. 初始化分片上传
     */
    @PostMapping("/init")
    public Map<String, Object> initUpload(
            @RequestParam String bucket,
            @RequestParam String object) throws Exception {
        String uploadId = client.createMultipartUpload(
            CreateMultipartUploadArgs.builder()
                .bucket(bucket)
                .object(object)
                .build()
        );
        uploadCache.put(uploadId, new ArrayList<>());

        Map<String, Object> result = new HashMap<>();
        result.put("uploadId", uploadId);
        result.put("chunkSize", 64 * 1024 * 1024); // 64MB
        return result;
    }

    /**
     * 2. 上传分片
     */
    @PostMapping("/part")
    public Map<String, Object> uploadPart(
            @RequestParam String bucket,
            @RequestParam String object,
            @RequestParam String uploadId,
            @RequestParam int partNumber,
            @RequestParam MultipartFile chunk) throws Exception {

        UploadPartResponse response = client.uploadPart(
            UploadPartArgs.builder()
                .bucket(bucket)
                .object(object)
                .uploadId(uploadId)
                .partNumber(partNumber)
                .stream(chunk.getInputStream(), chunk.getSize(), -1)
                .build()
        );

        Part part = new Part(partNumber, response.etag());
        uploadCache.get(uploadId).add(part);

        Map<String, Object> result = new HashMap<>();
        result.put("etag", response.etag());
        result.put("partNumber", partNumber);
        return result;
    }

    /**
     * 3. 完成合并
     */
    @PostMapping("/complete")
    public Map<String, Object> completeUpload(
            @RequestParam String bucket,
            @RequestParam String object,
            @RequestParam String uploadId) throws Exception {

        List<Part> parts = uploadCache.remove(uploadId);
        // 按分片号排序
        parts.sort(Comparator.comparingInt(Part::partNumber));

        client.completeMultipartUpload(
            CompleteMultipartUploadArgs.builder()
                .bucket(bucket)
                .object(object)
                .uploadId(uploadId)
                .parts(parts)
                .build()
        );

        Map<String, Object> result = new HashMap<>();
        result.put("url", String.format("/files/download?bucket=%s&object=%s",
            bucket, object));
        return result;
    }

    /**
     * 4. 取消上传（清理未完成分片）
     */
    @PostMapping("/abort")
    public void abortUpload(
            @RequestParam String bucket,
            @RequestParam String object,
            @RequestParam String uploadId) throws Exception {
        uploadCache.remove(uploadId);
        client.abortMultipartUpload(
            AbortMultipartUploadArgs.builder()
                .bucket(bucket)
                .object(object)
                .uploadId(uploadId)
                .build()
        );
    }
}
```

### 4.5 图片处理与 MinIO 结合

MinIO 本身不内置图片处理，通常结合 **Thumbnailator**、**Imgscalr** 或 **ImageMagick** 实现：

```java
import net.coobird.thumbnailator.Thumbnails;
import java.io.ByteArrayOutputStream;

/**
 * 上传原图 + 自动生成缩略图
 */
public void uploadWithThumbnail(String bucket, String object,
                                 MultipartFile file) throws Exception {
    // 1. 上传原图
    client.putObject(
        PutObjectArgs.builder()
            .bucket(bucket)
            .object(object)
            .stream(file.getInputStream(), file.getSize(), -1)
            .contentType(file.getContentType())
            .build()
    );

    // 2. 生成缩略图（200x200）
    ByteArrayOutputStream thumbOut = new ByteArrayOutputStream();
    Thumbnails.of(file.getInputStream())
        .size(200, 200)
        .outputFormat("jpg")
        .toOutputStream(thumbOut);

    byte[] thumbBytes = thumbOut.toByteArray();

    // 3. 上传缩略图到 thumb/ 目录
    String thumbObject = "thumb/" + object;
    client.putObject(
        PutObjectArgs.builder()
            .bucket(bucket)
            .object(thumbObject)
            .stream(new ByteArrayInputStream(thumbBytes), thumbBytes.length, -1)
            .contentType("image/jpeg")
            .build()
    );
}
```

```mermaid
flowchart TB
    Upload["📤 用户上传图片"] --> Origin["原图: uploads/photo.jpg"]
    Upload --> Process["thumbnailator / imgscalr"]
    Process --> Thumb["缩略图: thumbs/photo.jpg"]
    Process --> Watermark["加水印: watermarked/photo.jpg"]
    Origin --> CDN["CDN / 直接访问"]
    Thumb --> CDN
    Watermark --> CDN
```

---

## 5. 运维与监控

### 5.1 MinIO Console 管理界面

MinIO Console 是官方 Web 管理界面，默认端口 `9001`：

| 功能 | 说明 |
|------|------|
| Bucket 管理 | 创建/删除/浏览/策略配置 |
| 对象管理 | 上传/下载/删除/分享/元数据编辑 |
| IAM 管理 | 创建用户/组/策略/AccessKey |
| 存储监控 | 容量/磁盘/错误率实时面板 |
| 配置管理 | 运行时动态修改配置 |
| 审计日志 | 操作日志查询 |
| 桶复制 | 图形化配置跨集群复制 |

**启动 Console：**

```bash
# 单机模式（端口 9000 API + 9001 Console）
minio server /data --console-address ":9001"

# 分布式模式
minio server \
  http://node{1..4}:9000/data{1..4} \
  --console-address ":9001"
```

### 5.2 mc 命令行工具常用命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `mc alias set` | 添加 MinIO 集群别名 | `mc alias set myminio http://localhost:9000 minioadmin minioadmin` |
| `mc ls` | 列出存储桶或对象 | `mc ls myminio/my-bucket` |
| `mc cp` | 复制（上传/下载/跨集群） | `mc cp file.txt myminio/my-bucket/` |
| `mc mb` | 创建存储桶 | `mc mb myminio/my-bucket` |
| `mc rb` | 删除存储桶 | `mc rb myminio/my-bucket --force` |
| `mc rm` | 删除对象 | `mc rm myminio/bucket/obj.txt` |
| `mc cat` | 输出对象内容 | `mc cat myminio/bucket/log.txt` |
| `mc find` | 查找对象 | `mc find myminio --name "*.log" --size "+10M"` |
| `mc diff` | 对比两个桶差异 | `mc diff myminio/bucket1 myminio/bucket2` |
| `mc mirror` | 桶间同步 | `mc mirror myminio/bucket1 myminio/bucket2` |
| `mc share` | 生成分享链接 | `mc share download myminio/bucket/obj.txt` |
| `mc admin info` | 集群信息概览 | `mc admin info myminio` |
| `mc admin status` | 各节点状态 | `mc admin status myminio` |
| `mc admin trace` | 实时请求跟踪 | `mc admin trace myminio -v` |
| `mc admin prometheus` | 生成 Prometheus 配置 | `mc admin prometheus generate myminio` |
| `mc encrypt` | 配置 SSE 密钥 | `mc encrypt set myminio` |

**常用操作示例：**

```bash
# 添加别名
mc alias set prod https://minio.example.com:9000 ACCESSKEY SECRETKEY

# 创建桶并设置公开读策略
mc mb prod/public-assets
mc anonymous set download prod/public-assets

# 上传目录
mc cp --recursive ./images/ prod/public-assets/images/

# 查看集群健康
mc admin health prod

# 实时跟踪 API 调用
mc admin trace prod --callers --errors
```

### 5.3 Prometheus + Grafana 监控

MinIO 内置 Prometheus 指标端点：

```mermaid
flowchart LR
    M1["MinIO Node 1<br/>:9000/minio/v2/metrics/cluster"]
    M2["MinIO Node 2<br/>:9000/minio/v2/metrics/cluster"]
    M3["MinIO Node 3<br/>:9000/minio/v2/metrics/cluster"]
    M4["MinIO Node 4<br/>:9000/minio/v2/metrics/cluster"]

    P["Prometheus Server"]

    G["Grafana Dashboard<br/>（ID: 13502 / 15951）"]

    M1 --> P
    M2 --> P
    M3 --> P
    M4 --> P
    P --> G
```

**Prometheus scrape 配置：**

```yaml
scrape_configs:
  - job_name: 'minio-cluster'
    metrics_path: /minio/v2/metrics/cluster
    static_configs:
      - targets:
        - '192.168.1.10:9000'
        - '192.168.1.11:9000'
        - '192.168.1.12:9000'
        - '192.168.1.13:9000'
    bearer_token: 'your-prometheus-bearer-token'

  - job_name: 'minio-node'
    metrics_path: /minio/v2/metrics/node
    static_configs:
      - targets:
        - '192.168.1.10:9000'
        - '192.168.1.11:9000'
        - '192.168.1.12:9000'
        - '192.168.1.13:9000'
    bearer_token: 'your-prometheus-bearer-token'
```

**关键监控指标：**

| 指标 | 含义 | 告警建议 |
|------|------|---------|
| `minio_disk_storage_used_bytes` | 已用存储 | > 85% warning, > 95% critical |
| `minio_disk_free_inodes` | 剩余 inode | < 1M 时告警 |
| `minio_disk_offline` | 离线磁盘数 | > 0 即告警 |
| `minio_heal_objects_error_count` | 修复失败数 | > 0 即告警 |
| `minio_s3_requests_errors_total` | 请求错误数 | 速率升高时告警 |
| `minio_node_uptime` | 节点运行时间 | 重启/异常停止时告警 |

**生成 Prometheus Token：**

```bash
# 创建 Prometheus 专用的访问密钥
mc admin user add myminio prometheus-user strong-password
mc admin policy set myminio prometheus user=prometheus-user

# 获取 token
mc admin prometheus generate myminio
```

### 5.4 扩容方式

MinIO 采用 **Pool（存储池）** 架构进行水平扩展：

```mermaid
flowchart TB
    subgraph Pool1["Pool 1（初始 4 节点 × 4 盘 = 16 盘）"]
        P1N1["Node1<br/>d1 d2 d3 d4"]
        P1N2["Node2<br/>d1 d2 d3 d4"]
        P1N3["Node3<br/>d1 d2 d3 d4"]
        P1N4["Node4<br/>d1 d2 d3 d4"]
    end

    subgraph Pool2["Pool 2（扩容 4 节点 × 4 盘 = 16 盘）"]
        P2N1["Node5<br/>d1 d2 d3 d4"]
        P2N2["Node6<br/>d1 d2 d3 d4"]
        P2N3["Node7<br/>d1 d2 d3 d4"]
        P2N4["Node8<br/>d1 d2 d3 d4"]
    end

    LB["负载均衡器"] --> Pool1
    LB --> Pool2
```

**扩容注意事项：**

| 要点 | 说明 |
|------|------|
| **在线扩容** | 集群无需停机，新 Pool 直接加入 |
| **数据不重新分布** | Pool 1 旧数据不会自动迁移到 Pool 2 |
| **不缩减** | MinIO 不支持删除 Pool，扩容前规划充足 |
| **均衡写入** | 新对象按 Pool 剩余空间比例写入 |
| **纠删码独立** | 每个 Pool 独立纠删码配置，扩容时可更改 N:M |

**启动时指定多 Pool：**

```bash
# 两个 Pool 共 8 节点
minio server \
  http://node{1..4}:9000/data{1..4} \    # Pool 1
  http://node{5..8}:9000/data{1..4}      # Pool 2
```

### 5.5 常见问题排查

| 现象 | 可能原因 | 解决方案 |
|------|---------|---------|
| `Access Denied` | AccessKey/SecretKey 错误或 Policy 不足 | 检查凭证；检查 Bucket Policy；验证用户权限 |
| `SignatureDoesNotMatch` | 客户端时间偏差 > 15 分钟 | 同步 NTP 时间；`w32tm /resync` |
| `Connection Refused` | MinIO 服务未启动或端口被防火墙阻拦 | `mc admin status` 检查；`netstat -ano \| findstr :9000` |
| `Bucket Not Found` | Bucket 不存在 | `mc ls` 确认桶名，注意大小写敏感 |
| `Quota Exceeded` / `Insufficient Storage` | 磁盘空间不足 | `mc admin info` 查看使用率；扩容 Pool 或清理旧数据 |
| `Request Timeout` | 网络延迟大或客户端超时太短 | 增大 HTTP 超时配置；检查网络质量 |
| `The difference between the request time and the current time is too large` | 时钟不同步 | 所有节点配置 NTP 服务 |
| `Malformed XML` | 生命周期/策略 XML 格式错误 | 用 JSON 格式替代；XML 验证 |
| 上传大文件失败 | 未使用 Multipart Upload | 超过 5GiB 必须分片；检查 Nginx `client_max_body_size` |
| `InvalidPart` / `EntityTooSmall` | 最后分片以外的分片 < 5MiB | 确保每个分片 ≥ 5MiB（最后分片不受限） |
| 纠删码修复慢 | 磁盘 I/O 瓶颈 | 检查磁盘健康；减少并发修复任务 |
| Console 无法访问 | `--console-address` 未配置 | 启动时加 `--console-address ":9001"` |
| `502 Bad Gateway` | Nginx 与 MinIO 连接断开 | 检查 Nginx upstream 配置；确保 `proxy_http_version 1.1` |

**常用诊断命令：**

```bash
# 1. 检查集群健康
mc admin info myminio

# 2. 检查各节点磁盘
mc admin disk myminio

# 3. 查看最近错误日志
mc admin console myminio

# 4. 实时 API 跟踪（找出慢请求）
mc admin trace myminio --callers --duration ">1s"

# 5. 检查纠删码修复状态
mc admin heal myminio

# 6. 检查桶权限
mc anonymous set myminio/my-bucket

# 7. 测试 MinIO 连通性
curl -I http://localhost:9000/minio/health/live

# 8. 查看 Prometheus 指标
curl http://localhost:9000/minio/v2/metrics/cluster
```

---

## 附录：MinIO 体系架构总览

```mermaid
flowchart TB
    subgraph ClientLayer["客户端层"]
        C1["AWS SDK (Java/Python/Go/JS)"]
        C2["mc CLI"]
        C3["S3 Browser / Cyberduck"]
        C4["Web 浏览器 (Console UI)"]
    end

    subgraph ProxyLayer["代理/负载均衡层"]
        LB["Nginx / HAProxy / Cloud LB"]
    end

    subgraph MinIOLayer["MinIO Server 层"]
        direction TB
        subgraph APILayer["S3 Compatible REST API"]
            AUTH["认证（AWS Signature V4）"]
            IAM["IAM Policy 引擎"]
            RATE["限流 & 限速"]
        end

        subgraph CoreLayer["核心引擎"]
            EC["纠删码 Reed-Solomon<br/>N 数据 + M 校验"]
            BITROT["Bit Rot 保护<br/>SHA-256 校验"]
            VER["版本控制<br/>Versioning"]
            LOCK["对象锁定<br/>WORM / Legal Hold"]
            ENC["加密<br/>SSE-C/S3/KMS"]
            LIFE["生命周期<br/>过期 / 迁移"]
            REPL["桶复制<br/>Replication"]
        end

        subgraph Notification["事件通知层"]
            WEBHOOK["Webhook"]
            KAFKA["Kafka"]
            AMQP["RabbitMQ"]
            REDIS["Redis"]
            PG["PostgreSQL"]
            ES["Elasticsearch"]
        end

        APILayer --> CoreLayer
        CoreLayer --> Notification
    end

    subgraph StorageLayer["存储层"]
        D1["NVMe / SSD / HDD (JBOD, 无RAID)"]
        D2["本地文件系统 (XFS / ext4)"]
    end

    subgraph Monitor["监控运维层"]
        P8["Prometheus"]
        G8["Grafana"]
        LOG["审计日志"]
        S3CON["Console (端口 9001)"]
    end

    C1 --> LB
    C2 --> LB
    C3 --> LB
    C4 --> S3CON
    LB --> MinIOLayer
    MinIOLayer --> StorageLayer
    MinIOLayer --> Monitor
```

---

> **参考资料：**
> - [MinIO 官方文档](https://min.io/docs/minio/linux/index.html)
> - [MinIO Java SDK](https://min.io/docs/minio/linux/developers/java/minio-java.html)
> - [MinIO GitHub](https://github.com/minio/minio)
> - [MinIO Erasure Coding](https://min.io/docs/minio/linux/operations/concepts/erasure-coding.html)
