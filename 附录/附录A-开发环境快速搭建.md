# 附录A 开发环境快速搭建

> 本附录帮助你从零搭建 Go 开发环境，预计 15 分钟完成。

---

## A.1 Go 安装

### macOS

```bash
# 方式一：Homebrew（推荐）
brew install go

# 方式二：官方安装包
# 从 https://go.dev/dl/ 下载 .pkg 文件，双击安装

# 验证
go version
# go version go1.22.x darwin/amd64
```

### Linux

```bash
# 下载并解压
wget https://go.dev/dl/go1.22.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.0.linux-amd64.tar.gz

# 配置环境变量（~/.bashrc 或 ~/.zshrc）
export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

source ~/.bashrc
go version
```

### Windows

从 https://go.dev/dl/ 下载 `.msi` 安装包，双击安装即可，安装程序会自动配置环境变量。

---

## A.2 环境变量说明

| 变量 | 说明 | 推荐值 |
|------|------|--------|
| `GOROOT` | Go 安装目录 | 安装程序自动设置，不用手动配 |
| `GOPATH` | 工作空间（旧模块模式用） | `$HOME/go` |
| `GOPROXY` | 模块代理（国内必配） | `https://goproxy.cn,direct` |
| `GOBIN` | go install 安装目录 | `$GOPATH/bin` |

```bash
# 国内开发者必配：加速模块下载
go env -w GOPROXY=https://goproxy.cn,direct
go env -w GOSUMDB=sum.golang.google.cn
```

---

## A.3 IDE 选择

### VS Code（推荐，轻量免费）

```bash
# 安装 Go 扩展
# 打开 VS Code → 扩展商店 → 搜索 "Go" → 安装官方扩展

# 安装 Go 工具链（VS Code 会自动提示）
# 手动安装：
go install golang.org/x/tools/gopls@latest
go install github.com/go-delve/delve/cmd/dlv@latest
go install honnef.co/go/tools/cmd/staticcheck@latest
```

### GoLand（JetBrains，付费，功能最全）

从 https://www.jetbrains.com/go/ 下载安装。如果你有 IntelliJ IDEA Ultimate 许可证，可直接安装 Go 插件。

### IDE 对比

| 维度 | VS Code | GoLand |
|------|---------|--------|
| 价格 | 免费 | 付费 |
| 启动速度 | 快 | 较慢 |
| 补全/重构 | 优秀 | 极佳 |
| 调试器 | delve 集成 | 自带图形化调试 |
| 适合人群 | 大多数开发者 | 重度 IDE 用户 |

---

## A.4 常用工具安装

```bash
# 热重载工具（开发时自动重启）
go install github.com/cosmtrek/air@latest

# 代码检查工具
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

# Mock 生成工具
go install go.uber.org/mock/mockgen@latest

# SQL 代码生成器
go install github.com/kyleconroy/sqlc/cmd/sqlc@latest

# API 文档生成
go install github.com/swaggo/swag/cmd/swag@latest
```

---

## A.5 第一个 Go 项目

```bash
# 创建项目
mkdir myapp && cd myapp
go mod init github.com/yourname/myapp

# 创建 main.go
cat > main.go << 'EOF'
package main

import "fmt"

func main() {
    fmt.Println("Hello, Go!")
}
EOF

# 运行
go run main.go

# 编译
go build -o myapp .
./myapp
```

---

## A.6 Java→Go 工具对照表

| 功能 | Java 工具 | Go 工具 |
|------|----------|---------|
| 项目管理 | Maven / Gradle | `go mod` |
| 编译 | `javac` / Maven | `go build` |
| 运行 | `java -jar` | `go run` / 直接执行二进制 |
| 测试 | JUnit / TestNG | `go test`（内置） |
| 依赖管理 | pom.xml / build.gradle | `go.mod` / `go.sum` |
| 代码格式化 | 无标准 | `gofmt` / `go fmt`（内置） |
| 静态检查 | SpotBugs / SonarQube | `go vet` / `golangci-lint` |
| 性能分析 | JProfiler / async-profiler | `pprof`（内置） |
| 文档生成 | Javadoc | `go doc` / pkg.go.dev |
| 热重载 | Spring DevTools | `air` |
