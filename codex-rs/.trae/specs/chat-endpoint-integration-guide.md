# Chat端口修复功能集成到主分支指导文档

## 1. 概述

本文档描述如何将 Chat 端口修复功能集成到主分支代码中。本次修复从 tag80 版本恢复了 Chat Completions API 的支持，采用将改动集中到新文件的策略，主分支只需添加引用即可。

## 2. 修复背景

codex 的 wire_api 移除了对于 chat 端口的支持，需要从 tag80 版本恢复相关代码以重新支持 Chat Completions API。这主要是为了兼容腾讯云等不支持 Responses API 的服务商。

## 3. 文件结构对比

### 3.1 新增文件（修复分支独有）

| 文件路径 | 用途 | 来源 |
|---------|------|------|
| `codex-api/src/endpoint/streaming.rs` | StreamingClient 实现 | tag80 独有 |
| `codex-api/src/endpoint/chat_legacy.rs` | ChatClient 实现（使用 StreamingClient） | tag80 的 chat.rs 改名 |
| `codex-api/src/requests/chat_legacy.rs` | ChatRequest 和 ChatRequestBuilder | tag80 的 requests/chat.rs 改名 |

### 3.2 修改文件

| 文件路径 | 修改内容 |
|---------|---------|
| `codex-api/src/common.rs` | 添加 Prompt 结构体 |
| `codex-api/src/endpoint/mod.rs` | 添加 chat_legacy 模块声明 |
| `codex-api/src/requests/mod.rs` | 添加 chat_legacy 模块声明和导出 |
| `codex-api/src/lib.rs` | 导出 ChatClient, ChatRequest, ChatRequestBuilder, Prompt |
| `codex-api/src/sse/chat.rs` | Chat SSE 处理（已存在，需确认兼容性） |

## 4. API 差异说明

### 4.1 ChatRequestBuilder::build 方法

| 版本 | 签名 |
|------|------|
| tag80 (chat_legacy) | `build(self, provider: &Provider) -> Result<ChatRequest, ApiError>` |
| 当前版本 (chat) | `build(self) -> Result<ChatRequest, ApiError>` |

### 4.2 ChatClient 底层实现

| 版本 | 底层实现 |
|------|---------|
| tag80 (chat_legacy) | 使用 `StreamingClient` |
| 当前版本 (chat) | 使用 `EndpointSession` |

### 4.3 关键兼容性修改

在 `requests/chat_legacy.rs` 中：
- 将 `developer` 角色转换为 `system` 角色（兼容腾讯云等 API）
- 移除了 reasoning 字段（标准 Chat Completions API 不支持）

## 5. 集成步骤

### 步骤 1：复制新文件到主分支

将以下文件从修复分支复制到主分支：

```
codex-api/src/endpoint/streaming.rs
codex-api/src/endpoint/chat_legacy.rs
codex-api/src/requests/chat_legacy.rs
```

### 步骤 2：修改 common.rs

在 `codex-api/src/common.rs` 中添加 `Prompt` 结构体：

```rust
/// Canonical prompt input for Chat and Responses endpoints.
#[derive(Debug, Clone)]
pub struct Prompt {
    /// Fully-resolved system instructions for this turn.
    pub instructions: String,
    /// Conversation history and user/tool messages.
    pub input: Vec<ResponseItem>,
    /// JSON-encoded tool definitions compatible with the target API.
    pub tools: Vec<Value>,
    /// Whether parallel tool calls are permitted.
    pub parallel_tool_calls: bool,
    /// Optional output schema used to build the `text.format` controls.
    pub output_schema: Option<Value>,
}
```

### 步骤 3：修改 endpoint/mod.rs

在 `codex-api/src/endpoint/mod.rs` 中添加模块声明：

```rust
mod streaming;
pub mod chat_legacy;
// ... 其他模块
```

### 步骤 4：修改 requests/mod.rs

在 `codex-api/src/requests/mod.rs` 中添加模块声明和导出：

```rust
pub mod chat;
pub mod chat_legacy;
// ... 其他模块

pub use chat::ChatRequest;
pub use chat::ChatRequestBuilder;
pub use chat_legacy::ChatRequest as LegacyChatRequest;
pub use chat_legacy::ChatRequestBuilder as LegacyChatRequestBuilder;
```

### 步骤 5：修改 lib.rs

在 `codex-api/src/lib.rs` 中添加导出：

```rust
// 在文件末尾添加
pub use crate::endpoint::chat_legacy::AggregateStreamExt;
pub use crate::endpoint::chat_legacy::ChatClient;
pub use crate::requests::chat_legacy::ChatRequest;
pub use crate::requests::chat_legacy::ChatRequestBuilder;
pub use crate::common::Prompt;
```

### 步骤 6：确认 sse/chat.rs 兼容性

确认 `codex-api/src/sse/chat.rs` 文件存在且包含 `spawn_chat_stream` 函数。该文件应该已经存在于主分支，但需要确认其签名与 `chat_legacy.rs` 中的调用一致：

```rust
pub fn spawn_chat_stream(
    stream_response: StreamResponse,
    idle_timeout: Duration,
    telemetry: Option<Arc<dyn SseTelemetry>>,
    _turn_state: Option<Arc<OnceLock<String>>>,
) -> ResponseStream
```

注意：`chat_legacy.rs` 中调用时传入 4 个参数，而主分支的 `chat.rs` 只传入 3 个参数。需要确保 `sse/chat.rs` 的签名兼容。

## 6. 使用示例

### 6.1 使用 ChatClient

```rust
use codex_api::{ChatClient, ChatRequestBuilder, Prompt, Provider, WireApi};

// 创建 Provider（使用 Chat 端口）
let provider = Provider {
    wire: WireApi::Chat,
    // ... 其他配置
};

// 创建 ChatClient
let client = ChatClient::new(transport, provider, auth);

// 使用 Prompt 构建请求
let prompt = Prompt {
    instructions: "You are a helpful assistant.".to_string(),
    input: vec![ResponseItem::Message {
        role: "user".to_string(),
        content: vec![ContentItem::InputText {
            text: "Hello!".to_string(),
        }],
        // ...
    }],
    tools: vec![],
    parallel_tool_calls: true,
    output_schema: None,
};

// 流式请求
let stream = client.stream_prompt(
    "gpt-4",
    &prompt.instructions,
    &prompt.input,
    &prompt.tools,
    None,
    None,
).await?;
```

### 6.2 使用 ChatRequestBuilder

```rust
use codex_api::{ChatRequestBuilder, Provider};

let request = ChatRequestBuilder::new("gpt-4", "instructions", &input, &tools)
    .conversation_id(Some("conv-1".to_string()))
    .session_source(Some(session_source))
    .build(&provider)?;
```

## 7. 测试验证

### 7.1 编译验证

```bash
cd codex-rs
cargo build -p codex-api
```

### 7.2 单元测试

```bash
cargo test -p codex-api
```

### 7.3 集成测试

确保以下测试场景通过：
1. ChatClient 可以正常创建
2. ChatRequestBuilder 可以正确构建请求
3. Prompt 结构体可以正确使用
4. 流式响应可以正常处理

## 8. 注意事项

### 8.1 模块命名

- 使用 `chat_legacy` 命名是为了与现有的 `chat.rs` 区分
- `chat_legacy` 版本需要 `Provider` 参数来构建请求
- 现有的 `chat.rs` 版本不需要 `Provider` 参数

### 8.2 兼容性处理

在 `requests/chat_legacy.rs` 中：
- `developer` 角色自动转换为 `system` 角色
- 移除了 reasoning 字段以兼容标准 Chat Completions API

### 8.3 导出别名

在 `requests/mod.rs` 中使用别名导出：
```rust
pub use chat_legacy::ChatRequest as LegacyChatRequest;
pub use chat_legacy::ChatRequestBuilder as LegacyChatRequestBuilder;
```

这样用户可以选择使用 `ChatRequest`（来自 chat.rs）或 `LegacyChatRequest`（来自 chat_legacy.rs）。

## 9. 回滚方案

如果集成出现问题，可以按以下步骤回滚：

1. 删除新增的文件：
   - `codex-api/src/endpoint/streaming.rs`
   - `codex-api/src/endpoint/chat_legacy.rs`
   - `codex-api/src/requests/chat_legacy.rs`

2. 恢复修改的文件到原始状态：
   - `codex-api/src/common.rs`
   - `codex-api/src/endpoint/mod.rs`
   - `codex-api/src/requests/mod.rs`
   - `codex-api/src/lib.rs`

## 10. 文件清单

### 需要复制的文件

```
codex-api/src/endpoint/streaming.rs     (88 行)
codex-api/src/endpoint/chat_legacy.rs   (285 行)
codex-api/src/requests/chat_legacy.rs   (504 行)
```

### 需要修改的文件

```
codex-api/src/common.rs      (添加 Prompt 结构体，约 15 行)
codex-api/src/endpoint/mod.rs (添加 2 行)
codex-api/src/requests/mod.rs (添加 4 行)
codex-api/src/lib.rs          (添加 5 行)
```

## 11. 总结

本次集成采用最小侵入性策略：
- 新增 3 个独立文件，包含所有修复逻辑
- 主分支只需添加约 26 行引用代码
- 保持与现有代码的兼容性
- 支持腾讯云等非标准 API 服务商
