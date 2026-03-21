# Codex CLI (Rust 实现)

我们提供 Codex CLI 作为独立的原生可执行文件，以确保零依赖安装。

## Fork 修改说明

本 Fork 包含以下修改，以支持额外的 API 提供商：

### Chat Completions API 支持

原始 Codex CLI 仅支持 OpenAI Responses API。本 Fork 恢复了对 **Chat Completions API** 的支持，使其能够与不支持 Responses API 的提供商（如腾讯云、使用 Chat 端点的 Azure OpenAI）兼容。

#### 恢复功能

1. **WireApi 枚举** - 在不同的 API 协议之间选择：
   - `Responses` - 默认的 OpenAI Responses API
   - `Chat` - 标准的 Chat Completions API
   - `Compact` - 压缩端点
2. **ChatClient** - 用于 Chat Completions API 流式传输的新客户端：
   ```rust
   use codex_api::{ChatClient, ChatRequestBuilder, Prompt, Provider, WireApi};

   let provider = Provider {
       wire: WireApi::Chat,
       // ... 其他配置
   };

   let client = ChatClient::new(transport, provider, auth);
   let stream = client.stream_prompt("gpt-4", &prompt).await?;
   ```
3. **Prompt 结构体** - 两种 API 的统一提示输入：
   ```rust
   pub struct Prompt {
       pub instructions: String,      // 系统指令
       pub input: Vec<ResponseItem>,  // 对话历史和消息
       pub tools: Vec<Value>,         // 工具定义
       pub parallel_tool_calls: bool, // 是否允许并行工具调用
       pub output_schema: Option<Value>, // 输出格式定义
   }
   ```
4. **SSE Chat 流处理** - 正确处理 Chat Completions SSE 事件：
   - `[DONE]` 哨兵处理
   - 跨 delta 的工具调用拼接
   - 推理内容支持

#### 配置方式

要使用 Chat API，请配置您的提供商：

```toml
[providers.custom]
name = "custom"
base_url = "https://api.example.com/v1"
wire = "chat"  # 使用 Chat Completions API 而非 Responses API
```

#### 修改/新增的文件

| 文件                                      | 变更                                      |
| --------------------------------------- | --------------------------------------- |
| `codex-api/src/provider.rs`             | 添加 `WireApi` 枚举和 `wire` 字段到 `Provider`  |
| `codex-api/src/common.rs`               | 添加 `Prompt` 结构体                         |
| `codex-api/src/sse/chat.rs`             | 新增：Chat SSE 流处理                         |
| `codex-api/src/endpoint/streaming.rs`   | 新增：`StreamingClient` 实现                 |
| `codex-api/src/endpoint/chat_legacy.rs` | 新增：`ChatClient` 和 `AggregateStreamExt`  |
| `codex-api/src/requests/chat.rs`        | 新增：`ChatRequest` 和 `ChatRequestBuilder` |
| `codex-api/src/requests/chat_legacy.rs` | 新增：遗留 Chat 请求构建器                        |
| `codex-api/src/requests/headers.rs`     | 添加 `conversation_id` 头部支持               |

### Bug 修复

1. **Azure URL 检测大小写不敏感** - 修复了 `matches_azure_responses_base_url` 函数，正确处理混合大小写的 URL。

***

## 安装 Codex

目前，安装 Codex 最简单的方式是通过 `npm`：

```shell
npm i -g @openai/codex
codex
```

您也可以通过 Homebrew 安装（`brew install --cask codex`）或直接从我们的 [GitHub Releases](https://github.com/openai/codex/releases) 下载特定平台的发布版本。

## 文档快速入门

- 首次使用 Codex？请参阅 [`docs/getting-started.md`](../docs/getting-started.md)（包含提示词、键盘快捷键和会话管理的详细说明）。
- 想要更深入的控制？请参阅 [`docs/config.md`](../docs/config.md) 和 [`docs/install.md`](../docs/install.md)。

## Rust CLI 的新功能

Rust 实现现在是维护中的 Codex CLI，并作为默认体验。它包含许多旧版 TypeScript CLI 从未支持的功能。

### 配置

Codex 支持丰富的配置选项。请注意，Rust CLI 使用 `config.toml` 而非 `config.json`。详情请参阅 [`docs/config.md`](../docs/config.md)。

### 模型上下文协议 (MCP) 支持

#### MCP 客户端

Codex CLI 作为 MCP 客户端运行，允许 Codex CLI 和 IDE 扩展在启动时连接到 MCP 服务器。详情请参阅 [`配置文档`](../docs/config.md#connecting-to-mcp-servers)。

#### MCP 服务器（实验性）

可以通过运行 `codex mcp-server` 将 Codex 启动为 MCP _服务器_。这允许 _其他_ MCP 客户端将 Codex 用作另一个代理的工具。

使用 [`@modelcontextprotocol/inspector`](https://github.com/modelcontextprotocol/inspector) 进行尝试：

```shell
npx @modelcontextprotocol/inspector codex mcp-server
```

使用 `codex mcp` 来添加/列出/获取/删除 `config.toml` 中定义的 MCP 服务器启动器，使用 `codex mcp-server` 直接运行 MCP 服务器。

### 通知

您可以通过配置一个脚本，在代理完成一轮对话时运行，来启用通知功能。[通知文档](../docs/config.md#notify) 包含一个详细示例，说明如何在 macOS 上通过 [terminal-notifier](https://github.com/julienXX/terminal-notifier) 获取桌面通知。当 Codex 检测到它在 Windows Terminal 内的 WSL 2 中运行时（设置了 `WT_SESSION`），TUI 会自动回退到原生 Windows Toast 通知，这样即使 Windows Terminal 不实现 OSC 9，审批提示和完成的对话也能显示出来。

### `codex exec` 以编程/非交互方式运行 Codex

要以非交互方式运行 Codex，请执行 `codex exec PROMPT`（您也可以通过 `stdin` 传递提示词），Codex 将处理您的任务，直到它决定完成并退出。输出直接打印到终端。您可以设置 `RUST_LOG` 环境变量来查看更多信息。
使用 `codex exec --ephemeral ...` 可以在不将会话滚动文件持久化到磁盘的情况下运行。

### 实验 Codex 沙箱

要测试在 Codex 提供的沙箱中运行命令会发生什么，我们在 Codex CLI 中提供以下子命令：

```
# macOS
codex sandbox macos [--full-auto] [--log-denials] [COMMAND]...

# Linux
codex sandbox linux [--full-auto] [COMMAND]...

# Windows
codex sandbox windows [--full-auto] [COMMAND]...

# 遗留别名
codex debug seatbelt [--full-auto] [--log-denials] [COMMAND]...
codex debug landlock [--full-auto] [COMMAND]...
```

### 通过 `--sandbox` 选择沙箱策略

Rust CLI 公开了一个专用的 `--sandbox`（`-s`）标志，让您选择沙箱策略，**无需**使用通用的 `-c/--config` 选项：

```shell
# 使用默认的只读沙箱运行 Codex
codex --sandbox read-only

# 允许代理在当前工作区内写入，同时仍然阻止网络访问
codex --sandbox workspace-write

# 危险！完全禁用沙箱（仅当您已在容器或其他隔离环境中运行时才这样做）
codex --sandbox danger-full-access
```

相同的设置可以通过顶层 `sandbox_mode = "MODE"` 键持久化在 `~/.codex/config.toml` 中，例如 `sandbox_mode = "workspace-write"`。
在 `workspace-write` 模式下，Codex 还将 `~/.codex/memories` 包含在其可写根目录中，因此内存维护不需要额外的审批。

## 代码组织

此文件夹是 Cargo 工作区的根目录。它包含相当多的实验性代码，但以下是关键 crate：

- [`core/`](./core) 包含 Codex 的业务逻辑。最终，我们希望这是一个库 crate，通常对构建其他使用 Codex 的 Rust/原生应用程序很有用。
- [`exec/`](./exec) 用于自动化的"无头" CLI。
- [`tui/`](./tui) 使用 [Ratatui](https://ratatui.rs/) 构建的全屏 TUI CLI。
- [`cli/`](./cli) 通过子命令提供上述 CLI 的 CLI 多工具。

如果您想贡献或详细检查行为，请从阅读每个 crate 下的模块级 `README.md` 文件开始，并从顶层 `codex-rs` 目录运行项目工作区，以便共享配置、功能和构建脚本保持一致。
