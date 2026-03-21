<p align="center"><code>npm i -g @openai/codex</code><br />或 <code>brew install --cask codex</code></p>
<p align="center"><strong>Codex CLI</strong> 是 OpenAI 推出的编程代理，可在您的本地计算机上运行。
<p align="center">
  <img src="https://github.com/openai/codex/blob/main/.github/codex-cli-splash.png" alt="Codex CLI splash" width="80%" />
</p>
</br>
如果您想在代码编辑器（VS Code、Cursor、Windsurf）中使用 Codex，<a href="https://developers.openai.com/codex/ide">请在 IDE 中安装。</a>
</br>如果您想要桌面应用体验，请运行 <code>codex app</code> 或访问 <a href="https://chatgpt.com/codex?ref=cli">chatgpt.com/codex</a>。</p>

---

## Fork 修改说明

本 Fork 恢复了对 **Chat Completions API** 的支持，该功能此前从上游 Codex CLI 中被移除。这使得 Codex 能够与不支持 OpenAI Responses API 的 API 提供商（如腾讯云、使用 Chat 端点的 Azure OpenAI）兼容。

### 主要变更

| 功能 | 说明 |
|------|------|
| **WireApi 枚举** | 在 `Responses`、`Chat` 或 `Compact` API 协议之间选择 |
| **ChatClient** | 用于 Chat Completions API 流式传输的新客户端 |
| **Prompt 结构体** | Responses 和 Chat API 的统一提示输入 |
| **SSE Chat 处理** | 正确处理 Chat Completions SSE 事件 |

### 配置方式

要使用 Chat API，请在 `~/.codex/config.toml` 中配置您的提供商：

```toml
[providers.custom]
name = "custom"
base_url = "https://api.example.com/v1"
wire = "chat"  # 使用 Chat Completions API
```

详细文档请参阅 [codex-rs/README_CN.md](./codex-rs/README_CN.md)。

---

## 快速入门

### 安装和运行 Codex CLI

使用您首选的包管理器全局安装：

```shell
# npm
npm i -g @openai/codex

# yarn
yarn global add @openai/codex

# pnpm
pnpm add -g @openai/codex

# bun
bun install -g @openai/codex
```

<details>
<summary>您也可以前往 <a href="https://github.com/openai/codex/releases/latest">最新 GitHub Release</a> 下载适合您平台的二进制文件。</summary>

每个 GitHub Release 包含多个可执行文件，但实际上您可能需要以下之一：

- macOS
  - Apple Silicon/arm64: `codex-aarch64-apple-darwin.tar.gz`
  - x86_64（较旧的 Mac 硬件）: `codex-x86_64-apple-darwin.tar.gz`
- Linux
  - Most Linux distros: `codex-x86_64-unknown-linux-musl.tar.gz`
- Windows
  - x86_64: `codex-x86_64-pc-windows-msvc.zip`

每个压缩包包含一个名称中嵌入了平台信息的单个条目（例如 `codex-x86_64-unknown-linux-musl`），因此您可能需要在解压后将其重命名为 `codex`。

</details>

### 使用 ChatGPT 计划使用 Codex

运行 `codex` 并选择 **Sign in with ChatGPT**。我们建议登录您的 ChatGPT 账户，将 Codex 作为您的 Plus、Pro、Team、Edu 或 Enterprise 计划的一部分使用。[了解 ChatGPT 计划中包含的内容](https://help.openai.com/en/articles/11369540-codex-in-chatgpt)。

您也可以使用 API 密钥使用 Codex，但这需要[额外设置](https://developers.openai.com/codex/auth#sign-in-with-an-api-key)。

## 文档

- [**Codex 文档**](https://developers.openai.com/codex)
- [**贡献指南**](./docs/contributing.md)
- [**安装与构建**](./docs/install.md)
- [**开源基金**](./docs/open-source-fund.md)

本仓库采用 [Apache-2.0 许可证](LICENSE) 授权。
