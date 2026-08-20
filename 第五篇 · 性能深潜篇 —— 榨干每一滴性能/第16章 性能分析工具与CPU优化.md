### 第16章 性能分析工具与CPU优化：pprof与缓存友好编程

> 📖 本章你将学会：
> - 用pprof的CPU/内存/Goroutine/阻塞分析定位性能瓶颈
> - 读懂火焰图，从发现慢到定位到一行代码
> - 理解CPU缓存层级与缓存行，写出缓存友好代码
> - 预防false sharing，用数据布局优化提升性能

---

#### 16.1 pprof：Go的性能X光机

##### 16.1.1 开篇：JProfiler vs pprof——收费仪器 vs 内建X光

Java性能分析需要JProfiler（收费）或async-profiler（第三方）。
Go的pprof**内建于标准库**——免费、零依赖、一个命令启动。

```go
// HTTP服务中集成pprof
import _ "net/http/pprof"

func main() {
    // 只需import，pprof自动注册到http.DefaultServeMux
    http.ListenAndServe("localhost:6060", nil)
}
```

```bash
# 实时分析运行中的服务
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
go tool pprof http://localhost:6060/debug/pprof/heap
go tool pprof http://localhost:6060/debug/pprof/goroutine

# 火焰图（浏览器打开）
go tool pprof -http=:8080 http://localhost:6060/debug/pprof/profile?seconds=30
```

> 💡 比喻：Java性能分析像去医院做核磁共振——预约、排队、收费、
> 出报告。Go pprof像随身带的X光机——随时随地拍一张，立刻看结果。

##### 16.1.2 pprof四种核心分析

| 分析类型 | 抓取什么 | Java对应 | 典型用途 |
|------|------|------|------|
| CPU Profile | 函数调用采样 | JProfiler CPU | 找CPU热点 |
| Heap Profile | 内存分配采样 | JProfiler Memory | 找分配热点 |
| Goroutine | 当前所有Goroutine | Thread Dump | 检测泄漏 |
| Block/Mutex | 阻塞/锁竞争 | 无直接对应 | 找并发瓶颈 |

##### 16.1.3 火焰图解读

火焰图是性能分析的"杀手锏"——横轴是CPU时间占比，纵轴是调用栈深度。

```
火焰图示意:
│ ┌───┐ │ ┌──────────┐
│ │ A │ │ │    B     │ ← 顶层：实际执行的函数
│ ├───┘ │ ├──────────┤
│ │     │ │          │
│ ├───┐ │ ├────┐ ┌──┤
│ │ C │ │ │ D  │ │E │ ← 中层：调用者
│ ├───┘ │ ├────┘ ├──┤
│ │     │ │      │  │
│ main()│ │      │  │ ← 底层：入口
└───────┴─┴──────┴──┘
        横轴: CPU时间占比
```

**读图规则**：
1. 越宽的块=占用CPU越多=优化目标
2. 从上往下看=从叶子到根=函数调用链
3. 找最宽的叶子函数=CPU热点

---

#### 16.2 CPU缓存友好编程

##### 16.2.1 CPU缓存层级

```
速度阶梯:
L1 Cache   ~0.5ns   32-64KB   (桌面=伸手就到)
L2 Cache   ~3ns     256KB-1MB  (抽屉=转身就到)
L3 Cache   ~12ns    4-32MB    (走廊柜子=走几步)
主存(RAM)  ~100ns   ∞         (地下车库=下楼取)
```

> 💡 比喻：CPU缓存层级像办公场景——L1是桌面（伸手就到，最快），
> L2是抽屉（转身就到），L3是走廊柜子（走几步），主存是地下车库
>（下楼取，最慢）。数据在L1比在主存快200倍。

##### 16.2.2 缓存行（Cache Line）

CPU不以字节为单位读内存，而是以**缓存行**为单位——通常64字节。
这意味着如果你读了`[0]`，`[1]`到`[7]`（假设8字节int）也被加载到缓存。

```go
// ✅ 缓存友好: 连续内存遍历
nums := make([]int, 1000)
sum := 0
for i := 0; i < len(nums); i++ {
    sum += nums[i] // 顺序访问，缓存命中率高
}

// ❌ 缓存不友好: 随机访问
indices := shuffleIndices(1000)
for _, idx := range indices {
    sum += nums[idx] // 随机访问，缓存失效频繁
}
```

##### 16.2.3 Struct布局优化

```go
// ❌ 差: 字段排列导致padding浪费
type Bad struct {
    a bool   // 1B + 7B padding
    b int64  // 8B
    c bool   // 1B + 7B padding
} // 总计 24B，50%是padding

// ✅ 好: 大字段在前，小字段在后
type Good struct {
    b int64  // 8B
    a bool   // 1B
    c bool   // 1B + 6B padding
} // 总计 16B，节省33%
```

用`fieldalignment`工具检查：

```bash
go install golang.org/x/tools/go/analysis/passes/fieldalignment/cmd/fieldalignment@latest
fieldalignment ./...
```

##### 16.2.4 False Sharing

当两个Goroutine频繁写相邻的变量（同一个缓存行），会导致缓存行
反复失效——这叫**false sharing**。

```go
// ❌ false sharing: 两个计数器在同一缓存行
type Counters struct {
    a int64 // 8B
    b int64 // 8B — 和a在同一缓存行(64B)!
}

// ✅ 填充缓存行隔离
type Counters struct {
    a int64
    _ [56]byte // 填充到64B边界
    b int64
}
```

---

#### ⚡ 16.3 性能贴士：从profile到优化的完整流程

**优化流程**：

```
1. 发现慢 → pprof CPU profile (30秒采样)
2. 看火焰图 → 找最宽的函数
3. 分析原因 → 逃逸? 分配? 缓存失效? 锁竞争?
4. 针对优化 → 预分配/sync.Pool/缓存友好/false sharing
5. Benchmark验证 → 确认提升
```

| 优化技术 | 场景 | 典型提升 |
|------|------|----------|
| 预分配slice | 已知大小的循环 | 2-5x |
| sync.Pool | 临时对象复用 | 减少50%+ GC |
| 字段对齐 | 高频struct | 10-30% |
| false sharing填充 | 多核计数器 | 2-10x |
| SoA布局 | 高频遍历特定字段 | 2-5x |

> 💡 性能口诀：**pprof内建免费查，火焰图找宽块挖；缓存行64字节对齐，
> false sharing填充解。**

---

#### 本章小结

1. **pprof内建于标准库** —— CPU/Heap/Goroutine/Block四种分析，
   `go tool pprof -http`一键出火焰图。零依赖零收费。

2. **火焰图找最宽的块** —— 横轴CPU时间，纵轴调用栈。最宽的叶子
   函数就是优化目标。

3. **CPU缓存层级决定性能** —— L1到主存差200倍。连续访问、预取、
   缓存行对齐是缓存友好的核心。

4. **struct字段排列影响内存** —— 大字段在前减少padding，
   `fieldalignment`工具自动检测。

5. **false sharing是多核杀手** —— 相邻变量在同一缓存行导致反复失效，
   用`[56]byte`填充隔离。

> 🤔 思考题：Go的slice是连续内存，天然缓存友好。但Go的map不是
> 连续的（哈希桶）。频繁遍历map会比遍历slice慢多少？为什么？
