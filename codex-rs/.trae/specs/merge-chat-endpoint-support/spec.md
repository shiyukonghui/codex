# 合并Chat端口支持功能 Spec

## Why
最新版本的codex-rs移除了对Chat Completions API（wire_api::Chat）的支持，需要从"兼容chat版"分支恢复相关代码以重新支持Chat Completions API，使系统能够与腾讯云等使用Chat Completions API的服务提供商兼容。

## What Changes
- 添加 `sse/chat.rs` - Chat SSE流处理模块
- 添加 `endpoint/streaming.rs` - StreamingClient实现
- 添加 `endpoint/chat_legacy.rs` - ChatClient实现
- 添加 `requests/chat_legacy.rs` - ChatRequest和ChatRequestBuilder
- 添加 `requests/chat.rs` - Chat请求模块
- 修改 `sse/mod.rs` - 导出chat模块
- 修改 `endpoint/mod.rs` - 添加chat_legacy和streaming模块
- 修改 `requests/mod.rs` - 添加chat和chat_legacy模块
- 修改 `common.rs` - 添加Prompt结构体
- 修改 `lib.rs` - 导出ChatClient, ChatRequest, ChatRequestBuilder, Prompt等
- 修改 `provider.rs` - 添加WireApi枚举和wire字段
- **BREAKING** Provider结构体新增wire字段，需要更新所有Provider创建代码

## Impact
- Affected specs: codex-api模块
- Affected code:
  - `codex-api/src/sse/chat.rs` (新增)
  - `codex-api/src/sse/mod.rs` (修改)
  - `codex-api/src/endpoint/streaming.rs` (新增)
  - `codex-api/src/endpoint/chat_legacy.rs` (新增)
  - `codex-api/src/endpoint/mod.rs` (修改)
  - `codex-api/src/requests/chat.rs` (新增)
  - `codex-api/src/requests/chat_legacy.rs` (新增)
  - `codex-api/src/requests/mod.rs` (修改)
  - `codex-api/src/common.rs` (修改)
  - `codex-api/src/lib.rs` (修改)
  - `codex-api/src/provider.rs` (修改)

## ADDED Requirements

### Requirement: Chat端口支持恢复
系统应恢复对Chat Completions API的支持，通过从"兼容chat版"分支复制相关代码实现。

#### Scenario: ChatClient可用
- **WHEN** 用户配置使用WireApi::Chat
- **THEN** 系统能够使用ChatClient进行API调用

#### Scenario: ChatRequest构建正确
- **WHEN** 用户构建ChatRequest
- **THEN** 请求体和头部正确构建

#### Scenario: Prompt结构体可用
- **WHEN** 用户创建Prompt
- **THEN** 能够正确设置instructions, input, tools, parallel_tool_calls, output_schema

### Requirement: SSE流处理
系统应正确处理Chat Completions API的SSE流响应。

#### Scenario: 处理[DONE]哨兵
- **WHEN** SSE流收到`[DONE]`或`DONE`哨兵
- **THEN** 正确完成流处理并发送Completed事件

#### Scenario: 处理工具调用
- **WHEN** SSE流包含tool_calls
- **THEN** 正确解析并构建FunctionCall项

#### Scenario: 处理reasoning内容
- **WHEN** SSE流包含reasoning字段
- **THEN** 正确解析并构建Reasoning项

### Requirement: Provider WireApi支持
系统应支持配置Provider使用不同的API协议。

#### Scenario: WireApi枚举
- **WHEN** 创建Provider
- **THEN** 可以指定wire字段为Responses、Chat或Compact

## MODIFIED Requirements

### Requirement: Provider结构体
Provider结构体需要添加wire字段以支持不同的API协议。

**修改前**:
```rust
pub struct Provider {
    pub name: String,
    pub base_url: String,
    pub query_params: Option<HashMap<String, String>>,
    pub headers: HeaderMap,
    pub retry: RetryConfig,
    pub stream_idle_timeout: Duration,
}
```

**修改后**:
```rust
pub struct Provider {
    pub name: String,
    pub base_url: String,
    pub query_params: Option<HashMap<String, String>>,
    pub wire: WireApi,
    pub headers: HeaderMap,
    pub retry: RetryConfig,
    pub stream_idle_timeout: Duration,
}
```

## REMOVED Requirements
无

## 关键文件对比

### "兼容chat版"分支独有文件
1. `codex-api/src/sse/chat.rs` - Chat SSE流处理
2. `codex-api/src/endpoint/streaming.rs` - StreamingClient实现
3. `codex-api/src/endpoint/chat_legacy.rs` - ChatClient实现
4. `codex-api/src/requests/chat.rs` - Chat请求模块
5. `codex-api/src/requests/chat_legacy.rs` - Chat请求构建器
6. `codex-api/src/common.rs`中的Prompt结构体

### 需要修改的现有文件
1. `codex-api/src/sse/mod.rs` - 添加chat模块导出
2. `codex-api/src/endpoint/mod.rs` - 添加chat_legacy和streaming模块
3. `codex-api/src/requests/mod.rs` - 添加chat和chat_legacy模块
4. `codex-api/src/lib.rs` - 导出ChatClient等
5. `codex-api/src/provider.rs` - 添加WireApi枚举和wire字段

## 实现策略
1. 复制"兼容chat版"的chat.rs到sse目录
2. 复制"兼容chat版"的streaming.rs到endpoint目录
3. 复制"兼容chat版"的chat_legacy.rs到endpoint目录
4. 复制"兼容chat版"的chat.rs和chat_legacy.rs到requests目录
5. 添加Prompt结构体到common.rs
6. 更新各模块的mod.rs文件
7. 更新lib.rs导出
8. 更新provider.rs添加WireApi枚举
9. 修复任何编译错误
10. 运行测试验证
