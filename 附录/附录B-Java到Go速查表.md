# 附录B Java→Go速查表

> 本附录为 Java 程序员提供快速对照参考，按主题分类。

---

## B.1 语法对照

### 变量声明

| Java | Go | 说明 |
|------|-----|------|
| `int x = 10;` | `var x int = 10` 或 `x := 10` | Go 有短变量声明 `:=` |
| `String name = "Luis";` | `var name string = "Luis"` 或 `name := "Luis"` | — |
| `final int MAX = 100;` | `const MAX = 100` | Go 的常量 |
| `Object obj = null;` | `var obj interface{}` 或 `nil` | Go 无 null，用 nil |
| `int[] arr = {1,2,3};` | `arr := []int{1,2,3}` | Go 的 slice |

### 控制流

| Java | Go | 说明 |
|------|-----|------|
| `if (x > 0) { }` | `if x > 0 { }` | Go 条件不加括号 |
| `for (int i=0; i<n; i++) {}` | `for i := 0; i < n; i++ {}` | Go 只有一个 for |
| `while (x > 0) {}` | `for x > 0 {}` | Go 用 for 替代 while |
| `for (int x : list) {}` | `for _, x := range list {}` | range 遍历 |
| `switch(x) { case 1: ... break; }` | `switch x { case 1: ... }` | Go 默认不穿透，不用 break |
| `try { } catch (Exception e) { }` | `if err != nil { }` | Go 用值而非异常 |

### 类型系统

| Java | Go | 说明 |
|------|-----|------|
| `class User { }` | `type User struct { }` | Go 用 struct |
| `interface Foo { }` | `type Foo interface { }` | Go 接口隐式实现 |
| `extends Base` | 嵌入 `Base` 字段 | Go 无继承，用组合 |
| `Object` | `interface{}` / `any` | Go 1.18+ 可用 any |
| `List<String>` | `[]string` | Go 用 slice |
| `Map<K,V>` | `map[K]V` | Go 内置 map |
| `Optional<T>` | `*T` 或 `T, bool` | Go 用指针或多返回值 |
| `enum` | `type Color int` + `const` + `iota` | Go 无内置 enum |

### 并发

| Java | Go | 说明 |
|------|-----|------|
| `new Thread(() -> {}).start()` | `go func() {}()` | Goroutine |
| `ExecutorService` | Goroutine + Channel | 不需要线程池 |
| `synchronized` | `sync.Mutex` | 显式锁 |
| `ReentrantReadWriteLock` | `sync.RWMutex` | 读写锁 |
| `CountDownLatch` | `sync.WaitGroup` | 等待一组 goroutine |
| `BlockingQueue` | `chan T` | Channel |
| `Future<T>` | `<-chan T` | Channel 做异步结果 |
| `CompletableFuture` | `errgroup` + Channel | 并发错误处理 |
| `Thread.interrupt()` | `context.Cancel()` | Context 取消传播 |

---

## B.2 错误处理对照

| Java | Go | 说明 |
|------|-----|------|
| `throw new Exception()` | `return errors.New(...)` | Go 返回 error 值 |
| `try-catch-finally` | `if err != nil` + `defer` | 显式检查 + defer 清理 |
| `throws Exception` | 无 | Go 不声明异常 |
| `RuntimeException` | `panic()` | 不可恢复错误 |
| `catch(Exception)` | `recover()` | 从 panic 恢复 |
| `Exception.getCause()` | `errors.Unwrap()` | 错误链 |
| `instanceof` | `errors.As()` | 类型断言错误 |
| `@ControllerAdvice` | 无全局异常处理 | 每个 Handler 显式处理 |

---

## B.3 常用 API 对照

### 集合操作

| 操作 | Java | Go |
|------|------|-----|
| 创建列表 | `List.of(1,2,3)` | `[]int{1,2,3}` |
| 列表追加 | `list.add(x)` | `slice = append(slice, x)` |
| 列表长度 | `list.size()` | `len(slice)` |
| 创建映射 | `new HashMap<>()` | `make(map[K]V)` |
| 映射存取 | `map.put(k,v)` / `map.get(k)` | `map[k] = v` / `v, ok := map[k]` |
| 映射删除 | `map.remove(k)` | `delete(map, k)` |
| 字符串分割 | `str.split(",")` | `strings.Split(str, ",")` |
| 字符串包含 | `str.contains("x")` | `strings.Contains(str, "x")` |
| 字符串拼接 | `str1 + str2` | `str1 + str2` 或 `strings.Join` |
| 转大写 | `str.toUpperCase()` | `strings.ToUpper(str)` |

### 时间处理

| 操作 | Java | Go |
|------|------|-----|
| 当前时间 | `LocalDateTime.now()` | `time.Now()` |
| 格式化 | `DateTimeFormatter` | `time.Format("2006-01-02")` |
| 解析 | `LocalDate.parse()` | `time.Parse("2006-01-02", str)` |
| 时间差 | `Duration.between()` | `time.Since(t)` 或 `t.Sub(t2)` |
| 睡眠 | `Thread.sleep(1000)` | `time.Sleep(time.Second)` |

> 💡 提示：Go 的时间格式化用"2006-01-02 15:04:05"而非"YYYY-MM-DD"——这是 Go 的特色设计，格式串本身就是一个参考时间点（2006年1月2日 15:04:05），记忆口诀：1月2日3点4分5年6时区。

---

## B.4 HTTP 开发对照

| 操作 | Java (Spring) | Go (Gin) |
|------|--------------|----------|
| 路由注册 | `@GetMapping("/x")` | `r.GET("/x", handler)` |
| 路径参数 | `@PathVariable` | `c.Param("id")` |
| 请求体 | `@RequestBody` | `c.ShouldBindJSON(&v)` |
| 响应JSON | `@ResponseBody` / `ResponseEntity` | `c.JSON(200, obj)` |
| 中间件 | `Filter` / `Interceptor` | `r.Use(middleware)` |
| 静态文件 | `StaticResource` | `r.Static("/assets", ".")` |
| 依赖注入 | `@Autowired` | 手动构造传入 |

---

## B.5 测试对照

| 操作 | Java (JUnit) | Go (testing) |
|------|-------------|-------------|
| 测试方法 | `@Test void testX()` | `func TestX(t *testing.T)` |
| 断言相等 | `assertEquals(a, b)` | `if a != b { t.Errorf(...) }` |
| 测试前 | `@BeforeEach` | `func setup(t *testing.T) func()` |
| 参数化 | `@ParameterizedTest` | 表驱动测试 |
| 超时 | `@Timeout(1)` | `t.Parallel()` + context |
| 性能基准 | JMH | `func BenchmarkX(b *testing.B)` |
| Mock | Mockito | `mockgen` 生成 |
