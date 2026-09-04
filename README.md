# ⚠️⚠️⚠️归档不推荐使用，20260904

<div align="center">
  <h1>AeroX</h1>
  高性能游戏服务器后端框架

[![Rust](https://img.shields.io/badge/rust-1.92.0+-orange.svg)](https://www.rust-lang.org)[![Crates.io](https://img.shields.io/crates/v/aerox)](https://crates.io/crates/aerox)  [![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/cherish-ltt/aerox/blob/main/LICENSE) [![doc](https://docs.rs/aerox/badge.svg)](https://docs.rs/aerox/latest/aerox/) [![Downloads](https://img.shields.io/crates/d/aerox.svg)](https://crates.io/crates/aerox)

</div>

## 简介

AeroX 是一个基于 Rust 开发的专注于游戏服务器后端和实时消息转发场景的高性能框架。它采用 Reactor 模式实现高并发连接处理，整合 Bevy ECS 架构，提供模块化、可扩展的解决方案。

### 核心特性

- ⚡ **高性能** - 基于 Tokio 异步运行时，零拷贝消息处理
- 🔌 **插件系统** - 模块化设计，功能可插拔
- 🎮 **ECS 整合** - 集成 Bevy ECS，数据驱动游戏逻辑
- 🔐 **类型安全** - Rust 类型系统保证内存安全
- 📦 **Protobuf 支持** - 高效的二进制协议
- 🛣️ **灵活路由** - Axum 风格的中间件系统
- 🌐 **真实网络** - 所有示例使用真实TCP通信，无模拟数据

## 架构

```
Application Layer     ┌─────────┐ ┌─────────┐ ┌─────────┐
                      │ PluginA │ │ PluginB │ │ PluginC │
                      └─────────┘ └─────────┘ └─────────┘

Framework Core        ┌─────────┐ ┌─────────┐ ┌─────────┐
                      │  Router │ │   ECS   │ │  Config │
                      └─────────┘ └─────────┘ └─────────┘

Network Layer         ┌─────────┐ ┌─────────┐ ┌─────────┐
                      │   TCP   │ │   KCP   │ │  QUIC   │
                      └─────────┘ └─────────┘ └─────────┘
```

## 快速开始

### 系统要求

- **Rust 版本**: 1.92.0+
- **Edition**: 2024

### 安装

将以下内容添加到 `Cargo.toml`：

```toml
[dependencies]
aerox = "0.1"
```

### 特性标志

AeroX 支持按需编译，您可以选择只启用需要的功能：

```toml
# 默认：包含服务器和客户端功能
aerox = "0.1"

# 仅服务器
aerox = { version = "0.1", default-features = false, features = ["server"] }

# 仅客户端
aerox = { version = "0.1", default-features = false, features = ["client"] }
```

### 运行示例

#### 1. ECS 游戏服务器 (端口 8080)

展示如何使用 AeroX 构建完整的 ECS 游戏服务器：

```bash
# 终端 1 - 启动服务器
cargo run --example ecs_basics -- server

# 终端 2 - 启动客户端
cargo run --example ecs_basics -- client

# 终端 3 - 启动另一个客户端（多玩家测试）
cargo run --example ecs_basics -- client
```

**功能特性**：
- ✅ 玩家登录和会话管理
- ✅ 实时位置更新
- ✅ 聊天系统
- ✅ 心跳检测
- ✅ 玩家列表查询

#### 2. 路由和中间件演示 (端口 8081)

展示自定义中间件和路由系统：

```bash
# 终端 1 - 启动服务器
cargo run --example router_middleware -- server

# 终端 2 - 启动客户端（测试认证、限流等）
cargo run --example router_middleware -- client
```

**功能特性**：
- ✅ 自定义中间件链（日志、认证、限流）
- ✅ 公开/受保护/管理员路由
- ✅ 令牌认证系统
- ✅ 限流保护（10 请求/分钟）
- ✅ 会话管理

#### 3. 完整游戏服务器 (端口 8082)

展示多玩家游戏服务器的完整实现：

```bash
# 终端 1 - 启动服务器
cargo run --example complete_game_server -- server

# 终端 2,3,4... - 启动多个客户端
cargo run --example complete_game_server -- client
```

**功能特性**：
- ✅ 完整的游戏循环
- ✅ 多玩家支持
- ✅ 实时位置同步
- ✅ 聊天广播
- ✅ 心跳监控
- ✅ ECS 实体管理

#### 4. 性能测试

运行性能基准测试：

```bash
cargo run --example benchmark
```

详见 [BENCHMARK.md](examples/BENCHMARK.md)

**重要说明**：
- ❌ `start.rs` 是实验性示例，使用了未完成的高级 API，请勿在生产环境使用
- ✅ 以上三个示例（ecs_basics, router_middleware, complete_game_server）使用真实 TCP 通信
- ✅ 所有示例都可以在多个终端中同时运行，测试多客户端场景

### 服务器示例代码

```rust
use aerox::Server;

#[tokio::main]
async fn main() -> aerox::Result<()> {
    Server::bind("127.0.0.1:8080")
        .route(1001, |ctx| async move {
            println!("收到消息: {:?}", ctx.data());
            Ok(())
        })
        .run()
        .await
}
```

### 客户端示例代码

```rust
use aerox::Client;

#[tokio::main]
async fn main() -> aerox::Result<()> {
    let mut client = Client::connect("127.0.0.1:8080").await?;

    // 注册消息处理器
    client.on_message(1001, |id, msg: MyMessage| async move {
        println!("收到: {:?}", msg);
        Ok(())
    }).await?;

    // 发送消息
    client.send(1001, &my_message).await?;

    Ok(())
}
```

## 文档

- [快速开始指南](docs/getting_started.md)
- [架构设计](docs/architecture.md)
- [配置说明](docs/configuration.md)
- [API 文档](https://docs.rs/aerox)

## Crate 结构

| Crate | 描述 |
|-------|------|
| `aerox_core` | 核心运行时和插件系统 |
| `aerox_network` | 网络层抽象和协议实现 |
| `aerox_protobuf` | Protobuf 编解码支持 |
| `aerox_ecs` | Bevy ECS 整合层 |
| `aerox_router` | 路由和中间件系统 |
| `aerox_plugins` | 内置插件 |
| `aerox_config` | 配置管理 |
| `aerox_client` | 客户端库 |

## 开发状态

**当前版本**: v0.1.1

**完成度**: 100% (核心功能全部完成)

### 已完成功能

- ✅ 项目基础设施
- ✅ 配置系统
- ✅ 错误处理
- ✅ TCP Reactor
- ✅ 连接管理
- ✅ 消息编解码
- ✅ 路由系统
- ✅ 中间件系统
- ✅ 插件系统
- ✅ Protobuf 支持
- ✅ ECS 整合
- ✅ 客户端库
- ✅ 完整示例

### 开发中

- 🔄 文档完善
- 🔄 CI/CD 配置
- 🔄 KCP 传输协议（规划）
- 🔄 QUIC 传输协议（规划）

## 性能

- **并发连接**: 支持 10,000+ 并发连接
- **消息吞吐**: 100,000+ msg/sec (单核)
- **延迟**: P99 < 1ms (本地网络)
- **内存**: 零拷贝设计，最小堆分配

## 测试

```bash
# 运行所有测试
cargo test

# 运行示例
cargo run --example ecs_basics -- server
cargo run --example ecs_basics -- client

# 性能测试
cargo run --example benchmark
```

**测试覆盖**: 129 tests，所有通过 ✅

## 贡献

欢迎贡献！

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

## 开发路线图

### v0.1.1 (当前)
- [x] 核心框架
- [x] TCP 支持
- [x] ECS 整合
- [x] 客户端库
- [x] 完整示例
- [x] Rust 2024 Edition

### v0.2.0 (计划)
- [ ] KCP 协议支持
- [ ] QUIC 协议支持
- [ ] WebSocket 支持
- [ ] 更多插件

### v0.3.0 (未来)
- [ ] 分布式支持
- [ ] 监控和追踪
- [ ] 性能优化
- [ ] 生产环境验证

## 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

## 致谢

- [Tokio](https://tokio.rs/) - 异步运行时
- [Bevy](https://bevy.org/) - ECS 框架
- [Axum](https://github.com/tokio-rs/axum) - 中间件设计灵感

## 联系方式

- **GitHub**: https://github.com/cherish-ltt/AeroX
- **Issue**: https://github.com/cherish-ltt/AeroX/issues
- **Crate.io**: https://crates.io/crates/aerox

---

<div align="center">
Made with ❤️ by AeroX Team
</div>
