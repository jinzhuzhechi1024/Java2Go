# 附录C Go标准库速览

> Go 标准库是语言的第二张名片。Java 有 JDK + Apache Commons + Guava，Go 用一个标准库覆盖了大部分日常需求。本附录按主题列出高频使用的标准库包。

---

## C.1 标准库全景图

Go 标准库包含约 140 个包，覆盖了字符串、集合、IO、网络、加密、测试等领域。与 Java 不同，Go 标准库的定位是"生产可用"而非"教学示例"——`net/http` 直接可以写生产服务，`encoding/json` 直接可以做 API 序列化。

---

## C.2 核心包速查

### 基础类型与操作

| 包 | 对标 Java | 常用功能 |
|------|----------|----------|
| `fmt` | `System.out.printf` | 格式化输入输出 |
| `strings` | `String` / `StringUtils` | 字符串操作（Split/Join/Contains/Trim） |
| `strconv` | `Integer.parseInt` | 字符串与基本类型转换 |
| `unicode` | `Character` | Unicode 判断 |
| `bytes` | `ByteArrayOutputStream` | 字节切片操作 |
| `errors` | 无（异常体系） | 错误创建与处理 |
| `log` | `java.util.logging` | 基础日志 |
| `slog` | SLF4J | 结构化日志（Go 1.21+） |

### 集合与数据结构

| 包 | 对标 Java | 常用功能 |
|------|----------|----------|
| `sort` | `Collections.sort` | 排序与搜索 |
| `container/heap` | `PriorityQueue` | 堆操作 |
| `container/list` | `LinkedList` | 双向链表 |
| `container/ring` | 无 | 环形缓冲 |
| `slices` | `List` 工具方法 | slice 通用操作（Go 1.21+，泛型） |
| `maps` | `Map` 工具方法 | map 通用操作（Go 1.21+，泛型） |

### 时间与数学

| 包 | 对标 Java | 常用功能 |
|------|----------|----------|
| `time` | `java.time` | 时间、定时器、睡眠 |
| `math` | `java.lang.Math` | 数学函数 |
| `math/rand` | `java.util.Random` | 伪随机数 |
| `math/big` | `BigInteger` | 大数运算 |

### IO 与文件

| 包 | 对标 Java | 常用功能 |
|------|----------|----------|
| `io` | `InputStream`/`OutputStream` | IO 接口定义 |
| `os` | `java.lang.System` / `java.io.File` | 操作系统交互、文件 |
| `bufio` | `BufferedReader` | 带缓冲的 IO |
| `filepath` | `java.nio.file.Path` | 跨平台路径操作 |
| `io/fs` | `java.nio.file` | 文件系统抽象接口 |
| `embed` | 无 | 嵌入静态文件到二进制 |

### 编码与序列化

| 包 | 对标 Java | 常用功能 |
|------|----------|----------|
| `encoding/json` | Jackson / Gson | JSON 编解码 |
| `encoding/xml` | JAXB | XML 编解码 |
| `encoding/base64` | `Base64` | Base64 编解码 |
| `encoding/csv` | 无（第三方） | CSV 读写 |
| `encoding/binary` | `ByteBuffer` | 二进制数据读写 |

### 网络

| 包 | 对标 Java | 常用功能 |
|------|----------|----------|
| `net/http` | `java.net.http` / Spring | HTTP 客户端与服务端 |
| `net/url` | `java.net.URL` | URL 解析 |
| `net/rpc` | RMI | RPC（已不推荐，用 gRPC） |
| `net` | `java.net.Socket` | 底层网络（TCP/UDP） |
| `html/template` | Thymeleaf | HTML 模板渲染 |
| `text/template` | 无 | 通用文本模板 |

### 并发

| 包 | 对标 Java | 常用功能 |
|------|----------|----------|
| `sync` | `java.util.concurrent` | Mutex/RWMutex/WaitGroup/Once |
| `sync/atomic` | `AtomicInteger` | 原子操作 |
| `context` | 无 | 取消传播与超时 |
| `time` | `ScheduledExecutorService` | Timer/Ticker |

### 加密

| 包 | 对标 Java | 常用功能 |
|------|----------|----------|
| `crypto/md5` | `MessageDigest` | MD5 哈希 |
| `crypto/sha256` | `MessageDigest` | SHA256 哈希 |
| `crypto/aes` | `javax.crypto` | AES 加解密 |
| `crypto/tls` | `javax.net.ssl` | TLS 配置 |

### 测试与调试

| 包 | 对标 Java | 常用功能 |
|------|----------|----------|
| `testing` | JUnit | 测试框架（内置） |
| `testing/fuzzing` | JUnit Quickcheck | 模糊测试 |
| `runtime/pprof` | JProfiler | 性能分析 |
| `runtime/trace` | async-profiler | 执行追踪 |
| `expvar` | Micrometer | 运行时变量暴露 |

### 反射与底层

| 包 | 对标 Java | 常用功能 |
|------|----------|----------|
| `reflect` | `java.lang.reflect` | 运行时反射 |
| `unsafe` | `sun.misc.Unsafe` | 绕过类型安全（慎用） |
| `runtime` | JVM 运行时 | 运行时控制（GC/GOMAXPROCS） |

---

## C.3 Java 程序员最需要熟悉的 10 个包

如果你时间有限，优先掌握以下 10 个包，覆盖 90% 的日常开发：

1. **`fmt`** — 格式化输出，比 Java 的 printf 更强大
2. **`strings`** — 字符串操作，替代 StringUtils
3. **`errors`** — 错误处理，替代 try-catch
4. **`encoding/json`** — JSON 序列化，替代 Jackson
5. **`net/http`** — HTTP 服务，替代 Spring Boot
6. **`sync`** — 并发控制，替代 java.util.concurrent
7. **`context`** — 取消传播，Java 中没有等价物
8. **`testing`** — 测试框架，替代 JUnit
9. **`time`** — 时间处理，替代 java.time
10. **`log/slog`** — 结构化日志，替代 SLF4J

---

## C.4 标准库 vs 第三方库决策指南

| 需求 | 标准库方案 | 第三方方案 | 建议 |
|------|----------|----------|------|
| HTTP 服务 | `net/http` | Gin / Echo | 简单服务用标准库，复杂路由用 Gin |
| JSON | `encoding/json` | jsoniter / son | 标准库够用，极致性能用 jsoniter |
| 配置 | `os.Getenv` + `flag` | viper / envconfig | 简单配置用标准库，复杂层级用 viper |
| 日志 | `log/slog` | zap / zerolog | slog 够用，极致性能用 zap |
| 数据库 | `database/sql` | sqlc / ent | 标准库打底，类型安全用 sqlc |
| 测试 Mock | 手写接口 mock | mockgen | 简单接口手写，复杂依赖用 mockgen |

> 💡 Go 标准库的设计哲学：**能用标准库解决就不引第三方**。这和 Java 生态"Spring 管一切"截然不同。好处是依赖少、编译快、安全可控；代价是某些高级功能需要自己组合。
