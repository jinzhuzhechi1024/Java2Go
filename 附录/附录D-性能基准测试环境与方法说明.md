# 附录D 性能基准测试环境与方法说明

> 本附录说明全书中所有性能数据的测试环境、基准方法和可复现方式，确保数据可追溯、可验证。

---

## D.1 测试环境

### 硬件

| 组件 | 规格 |
|------|------|
| CPU | Apple M2 Pro (8核性能 + 4核能效) |
| 内存 | 32GB LPDDR5 |
| 存储 | 512GB NVMe SSD |
| 网络 | 本地回环 (127.0.0.1) |

### 软件版本

| 软件 | 版本 |
|------|------|
| Go | 1.22.0 |
| Java | OpenJDK 21.0.1 (Temurin) |
| Spring Boot | 3.2.0 |
| Gin | 1.9.1 |
| MySQL | 8.0.35 |
| Docker | 24.0.7 |
| OS | macOS 14.2 (Darwin 23.2) |

### JVM 参数

```bash
java -Xms512m -Xmx512m -XX:+UseZGC -XX:+ZGenerational \
     -XX:+TieredCompilation -XX:+UseCompressedOops \
     -jar app.jar
```

### Go 编译参数

```bash
CGO_ENABLED=0 go build -ldflags="-s -w" -o app ./cmd/app
```

---

## D.2 基准测试方法

### Go 基准测试

使用标准库 `testing.B`，遵循 Go 官方基准测试规范：

```go
func BenchmarkUserHandler(b *testing.B) {
    // 预热
    for i := 0; i < 100; i++ {
        makeRequest()
    }

    b.ResetTimer() // 重置计时器，排除预热
    b.ReportAllocs()  // 报告内存分配

    for i := 0; i < b.N; i++ {
        makeRequest()
    }
}
```

```bash
# 运行基准测试
go test -bench=. -benchmem -count=5 -benchtime=3s

# 输出示例
# BenchmarkUserHandler-8    5000000    285 ns/op    320 B/op    2 allocs/op
```

### Java 基准测试

使用 JMH (Java Microbenchmark Harness)：

```java
@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@State(Scope.Thread)
@Warmup(iterations = 3, time = 1)
@Measurement(iterations = 5, time = 1)
@Fork(1)
public class UserHandlerBenchmark {

    @Benchmark
    public void handleRequest(Blackhole bh) {
        bh.consume(makeRequest());
    }
}
```

```bash
# 运行 JMH
java -jar benchmarks.jar -wi 3 -i 5 -f 1 -t 1
```

### HTTP 性能测试

使用 `wrk` 或 `hey` 进行 HTTP 压测：

```bash
# wrk 压测
wrk -t8 -c500 -d30s http://localhost:8080/api/users/1

# 输出示例
# Requests/sec: 95000.67
# Latency: 0.9ms (p99)
```

```bash
# Go 的 hey 工具
go install github.com/rakyll/hey@latest
hey -n 100000 -c 500 http://localhost:8080/api/users/1
```

---

## D.3 公平性保证

为确保 Java 和 Go 的对比公平，遵循以下原则：

1. **等效逻辑**：两端实现相同业务逻辑，不故意简化或复杂化任一方
2. **预热处理**：Java 端充分 JIT 预热（JMH Warmup 3 轮），Go 端也做等量预热
3. **GC 配置**：Java 使用 ZGC（低延迟），Go 使用默认 GC，都是各自的推荐配置
4. **连接复用**：数据库和 HTTP 客户端都使用连接池
5. **单核对比**：CPU 基准测试限制为单核（`GOMAXPROCS=1` / JMH `-t 1`），排除调度干扰
6. **多次取样**：每组测试运行 5 次取中位数，排除异常值
7. **内存测量**：Go 用 `runtime.ReadMemStats`，Java 用 `ManagementFactory.getMemoryMXBean`

---

## D.4 全书性能数据索引

| 章节 | 对比项 | Go 数据 | Java 数据 | 倍数 |
|------|--------|---------|----------|------|
| 第1章 | 启动速度 | <50ms | ~3s | ~60x |
| 第1章 | 内存占用 | ~15MB | ~250MB | ~17x |
| 第8章 | Goroutine创建 | 2KB/个 | 1MB/线程 | ~500x |
| 第14章 | 对象分配（栈） | ~4ns | ~N/A（全部堆） | — |
| 第15章 | GC暂停 (p99) | <1ms | 1-10ms (ZGC) | ~5-10x |
| 第16章 | struct字段访问 | ~0.5ns | ~N/A（对象头） | — |
| 第17章 | HTTP QPS (Gin) | ~95k | ~15k (Spring) | ~6x |
| 第18章 | Docker镜像 | ~12MB | ~300MB | ~25x |
| 第19章 | gRPC vs Feign | ~50k req/s | ~5k req/s | ~10x |
| 第20章 | 反射调用 | 180ns | ~30ns | Java快~6x |

> 💡 提示：性能数据受硬件、版本、负载模式影响，本附录数据仅供参考。在实际项目中，请使用相同方法论在你的目标环境中重新测试。

---

## D.5 常见基准测试陷阱

Java 程序员做 Go 基准测试时容易踩的坑：

1. **忘记 `b.ResetTimer()`**：初始化代码的时间被计入基准结果
2. **编译器优化掉了结果**：用 `b.Result = result` 或 `runtime.KeepAlive` 防止死代码消除
3. **忘记 `b.ReportAllocs()`**：只看时间不看内存分配，会遗漏 GC 压力问题
4. **用 `time.Now()` 替代 `testing.B`**：手动计时不如 `testing.B` 精确，且没有自动调整 `b.N`
5. **跨包比较**：Go 的 `testing.B` 和 Java 的 JMH 方法论不同，绝对数字不能直接比较，应该看相对倍数

---

## D.6 复现指南

全书性能测试代码托管在配套仓库中（仓库地址见序言说明）：

```bash
# 克隆仓库后进入 benchmarks 目录
cd java2go/benchmarks

# 运行 Go 基准测试
cd go-bench && go test -bench=. -benchmem

# 运行 Java 基准测试
cd java-bench && mvn clean install && java -jar target/benchmarks.jar

# 运行 HTTP 压测对比
./scripts/run-http-benchmark.sh
```

每个基准测试目录包含 `README.md`，说明该测试的场景、参数和预期结果。
