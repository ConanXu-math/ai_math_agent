# OpenCode 入门

本文档帮助你在本项目中快速理解 OpenCode 是什么、怎么运行、以及和平台的关系。

---

## 一、OpenCode 是什么？

**OpenCode** 是一个**开源的 AI 编程 Agent**，能力和 Claude Code / Cursor 类似：在终端或桌面里，通过自然语言让 AI 读代码、改代码、跑命令、查文档。

和 Claude Code 的主要区别：

- **100% 开源**，不绑定某一家模型商
- **多模型支持**：可用 Claude、OpenAI、Google、DeepSeek、本地模型等（通过 [OpenCode Zen](https://opencode.ai/zen) 或自建）
- **内置 LSP**：代码补全、跳转、诊断
- **Client/Server 架构**：核心是一个 HTTP 服务，TUI/桌面/Web 都是“客户端”，方便**用程序驱动**（我们的多智能体平台就是通过 API 调它）

官网与社区：<https://opencode.ai>，文档：<https://opencode.ai/docs>。

---

## 二、核心架构：Client + Server

```
┌─────────────────┐     HTTP / OpenAPI      ┌─────────────────────────┐
│  TUI / 桌面 /   │ ◄─────────────────────► │  OpenCode Server        │
│  Web / 你的脚本 │                         │  (opencode serve)       │
└─────────────────┘                         │  - Project / Session    │
                                            │  - Message / File       │
                                            │  - LSP / MCP            │
                                            └─────────────────────────┘
```

- 平时你运行 `opencode` 会同时起 **TUI + 本机 Server**。
- 也可以**只起 Server**：`opencode serve`，默认端口 **4096**，然后用 HTTP 调它（或用官方 SDK）。
- 我们的平台设计里，**OpenCodeAdapter** 就是通过这个 HTTP API 去“驱动” OpenCode 完成“草案代码生成”等步骤。

---

## 三、核心概念（和 API 对应）

| 概念 | 含义 | API 大致用法 |
|------|------|----------------|
| **Project** | 一个“工作区/项目”，可多工作树 | `GET /project`、`POST /project/init` |
| **Session** | 一次对话/任务会话，属于某个 Project，有目录 | `POST /project/:id/session`（body 里 `directory`）、`GET/POST .../session/:id/...` |
| **Message** | 会话里的一条条消息（用户 + AI） | `POST .../session/:id/message` 发消息并拿回复，`GET .../message` 拉历史 |
| **File** | 会话内对工作区文件的读/写/状态 | `GET .../session/:id/find/file`、`.../file`、`.../file/status` |

所以：**创建一个 Session（绑定目录）→ 发 Message（例如“请根据问题写一段 Python”）→ 从回复或 File 里拿到生成的代码**，这就是我们“草案阶段”要用的流程。

更细的接口见仓库里的 `opencode/specs/project.md`，以及运行时的 OpenAPI 文档：`http://localhost:4096/doc`（需先 `opencode serve`）。

---

## 四、怎么安装和运行？

### 1. 安装（任选其一）

```bash
# 一键安装
curl -fsSL https://opencode.ai/install | bash

# 或 npm/bun
npm i -g opencode-ai@latest

# 或 macOS Homebrew（推荐，易更新）
brew install anomalyco/tap/opencode
```

安装后终端执行 `opencode --version` 检查是否成功。若提示命令找不到，可新开一个终端或确认安装目录在 `PATH` 中。

### 2. 如何运行 OpenCode（两种方式）

#### 方式 A：交互式使用（TUI + 自带 Server）

在终端执行：

```bash
opencode
```

会同时启动：
- **TUI**：在终端里和 AI 对话、改代码、跑命令
- **本机 Server**：默认端口 4096，供 TUI 连接

适合日常在项目目录下边聊边写代码。用 `Tab` 可切换 **build** / **plan** Agent。

#### 方式 B：只起 Server（给脚本或平台用）

在终端执行：

```bash
opencode serve
```

默认监听 `127.0.0.1:4096`。可选参数：

```bash
opencode serve --port 4096 --hostname 127.0.0.1
# 允许浏览器跨域时加 --cors
opencode serve --cors http://localhost:5173
```

验证是否在跑：
- 浏览器打开：`http://localhost:4096/doc`（OpenAPI 文档）
- 或：`curl http://localhost:4096/global/health`

适合做「平台通过 HTTP 调 OpenCode」时的本地服务。

### 3. 认证（可选）

若需给 Server 加密码（防止本机被他人访问）：

```bash
OPENCODE_SERVER_PASSWORD=你的密码 opencode serve
```

用户名默认 `opencode`，可用 `OPENCODE_SERVER_USERNAME` 覆盖。

---

### 4. 在本仓库的 opencode 目录下从源码运行（开发模式）

不需要全局安装，可以在项目里的 **`opencode`** 目录用 Bun 直接跑当前源码。

**前置**：已安装 [Bun](https://bun.sh)（1.3+）。若刚装完 Bun，当前终端可能还找不到 `bun`，先执行 `source ~/.zshrc` 或新开一个终端，再执行下面的命令。

**步骤**：

```bash
# 1. 进入本地 opencode 目录（ai_math_agent 仓库下的 opencode）
cd /Users/conanxu/Desktop/ai_math_agent/opencode

# 2. 安装依赖
bun install

# 3. 运行（三选一）
bun dev                    # 启动 TUI（默认工作目录为 packages/opencode）
bun dev serve              # 只启动无头 Server（默认端口 4096）
bun dev serve --port 8080  # 指定端口
```

**等价关系**（开发时用 `bun dev`，安装后用 `opencode`）：

| 开发（在 opencode 目录下） | 安装后（全局）     |
|---------------------------|--------------------|
| `bun dev`                 | `opencode`         |
| `bun dev serve`           | `opencode serve`   |
| `bun dev web`             | `opencode web`     |
| `bun dev <目录>`          | `opencode <目录>` |

**在指定目录跑 TUI**（例如在平台根目录或当前项目）：

```bash
cd /Users/conanxu/Desktop/ai_math_agent/opencode
bun dev /Users/conanxu/Desktop/ai_math_agent   # 在 ai_math_agent 根目录启动 TUI
bun dev .                                      # 在 opencode 根目录启动 TUI
```

**只起 Server 时**，和安装后一样：浏览器打开 `http://localhost:4096/doc` 可看 API 文档。

---

## 五、API / 模型配置（必配）

OpenCode 需要连接一个 LLM 服务（Claude、OpenAI、DeepSeek 等）才能对话和改代码，**第一次用前要配好**。

### 方式一：环境变量 + 全局配置（推荐）

1. **设置对应提供商的 API Key（环境变量）**  
   按你要用的厂商选一个设置，例如：
   - **Anthropic（Claude）**：`export ANTHROPIC_API_KEY="sk-..."`
   - **OpenAI**：`export OPENAI_API_KEY="sk-..."`
   - **DeepSeek**：可用 OpenAI 兼容接口，例如 `export OPENAI_API_KEY="sk-..."` 并设置 `baseURL`（见下）

2. **建全局配置文件**（指定默认模型）  
   创建并编辑 `~/.config/opencode/opencode.json`：

   ```json
   {
     "$schema": "https://opencode.ai/config.json",
     "model": "anthropic/claude-sonnet-4-5"
   }
   ```

   若用 **DeepSeek**（OpenAI 兼容），可写成：

   ```json
   {
     "$schema": "https://opencode.ai/config.json",
     "model": "openai/gpt-4o",
     "provider": {
       "openai": {
         "options": {
           "apiKey": "{env:OPENAI_API_KEY}",
           "baseURL": "https://api.deepseek.com"
         }
       }
     }
   }
   ```

   把 `OPENAI_API_KEY` 换成你的 DeepSeek API Key，模型名按 DeepSeek 文档填（如 `deepseek-chat` 等）。

3. **用环境变量存 Key（更安全）**  
   在配置里用 `{env:变量名}` 引用，例如：

   ```json
   {
     "$schema": "https://opencode.ai/config.json",
     "model": "anthropic/claude-sonnet-4-5",
     "provider": {
       "anthropic": {
         "options": {
           "apiKey": "{env:ANTHROPIC_API_KEY}"
         }
       }
     }
   }
   ```
   然后在 shell 里 `export ANTHROPIC_API_KEY="sk-..."`，再运行 `bun dev`。

### 方式二：在 TUI 里用 /connect

启动 `bun dev` 后，在 TUI 里输入 **`/connect`**（或菜单里找「连接 / Connect」），按提示选提供商并填 API Key，会写入 OpenCode 自己的存储，下次不用再配。

### 小结

- **必做**：至少有一种 API Key（环境变量或 `/connect`），并让 OpenCode 能用到（默认模型在配置里或 TUI 里选）。
- **配置优先级**：项目下的 `opencode.json` > 全局 `~/.config/opencode/opencode.json`。
- 更多选项（超时、多模型等）见官方文档：<https://opencode.ai/docs/config>、<https://opencode.ai/docs/models>。

### 为什么我没配也能用？

常见情况有几种：

1. **OpenCode 自带免费模型（最常见）**  
   OpenCode 内置了免费模型（例如 **big-pickle**），TUI 右侧 “Getting started” 里写的是：“OpenCode includes free models so you can start immediately.” 所以不配置任何 API Key 也能直接对话、改代码。当前若显示 **Build · big-pickle**，用的就是这套免费模型。**big-pickle 不是本地模型**：它属于 OpenCode Zen，请求会发到 OpenCode 的云端；只是免费用、无需自己配 Key。需要 Claude、GPT、Gemini 或本地模型时，再通过 `/connect` 连接 75+ 提供商即可。

2. **环境变量里已经有 Key**  
   OpenCode 会自动读常见环境变量（如 `ANTHROPIC_API_KEY`、`OPENAI_API_KEY`）。如果你本机之前为 Cursor、Claude 或其他工具配过，同一终端里跑 `bun dev` 就会直接用这些 Key，无需再建 `opencode.json`。

3. **之前用过 `/connect`**  
   凭证会写在 `~/.local/share/opencode/auth.json`，下次启动会自动用，看起来就像「没配置」。

4. **默认选了已有能力的提供商**  
   没在 config 里写 `model` 时，OpenCode 会从「自带免费模型」、「最近用过的模型」或「当前已连接提供商里的第一个模型」里选一个；只要有其一可用，就会直接能用。

所以「没单独配 OpenCode」可能是：**在用自带免费模型（如 big-pickle）**，或环境里已有 Key，或之前连过一次并保存了。想确认当前用的是谁，看 TUI 底部/状态栏的模型名（如 “Build · big-pickle” 或 “OpenCode Zen”）即可。

---

## 六、内置 Agent 类型

OpenCode 自带两类可切换的 Agent（TUI 里用 Tab 切换）：

- **build**：默认，可改文件、跑命令，做完整开发任务。
- **plan**：只读分析，不改文件、执行命令会先问你是否允许，适合看代码、做方案。

平台里用 API 发消息时，可以指定用哪个 agent（若 API 支持）。另外还有 **general** 子 agent，用于复杂搜索和多步任务。

---

## 七、本仓库里的 OpenCode 相关路径

| 路径 | 说明 |
|------|------|
| `opencode/` | OpenCode 主仓库（含 Server、TUI、各客户端） |
| `opencode/specs/project.md` | Project/Session/Message 等接口的简要约定 |
| `opencode/packages/opencode/` | 核心运行时（含 Server 实现） |
| `opencode/packages/sdk/` | 官方 SDK（如 JS），用于程序化调用 Server |
| `opencode/packages/web/src/content/docs/zh-cn/server.mdx` | 服务器与 API 的中文说明 |
| `opencode/openevolve/` | 已放入的 OpenEvolve 进化框架（见 02、03 文档） |

---

## 八、和本平台的关系（简要）

在我们的「基于 OpenCode 的数学研究多智能体平台」里：

- **OpenCode** = 负责“**草案阶段**”：根据问题描述，在指定目录下开 Session、发 Message，让 AI 生成或修改初始代码。
- **OpenEvolve**（已位于 `opencode/openevolve/`）= 负责“**进化阶段**”：对这份初始代码做 MAP-Elites 等进化，得到 best 程序。
- **Numina**（在 `numina-lean-agent`）= 负责“**形式化 + Lean**”：best 代码 → 数学形式 → R1–R7 校验 → 翻成 Lean 并修复。

平台通过 **OpenCodeAdapter** 只和 OpenCode 的 HTTP API 打交道（Project/Session/Message/File），不关心 OpenCode 内部是用什么模型、什么 UI。

---

## 九、建议的下一步

1. **本地跑通一次**：`opencode` 或 `opencode serve`，确认能对话或能访问 `/doc`。
2. **看 API 文档**：浏览器打开 `http://localhost:4096/doc`，扫一眼 Session、Message、File 相关接口。
3. **做平台时**：在 `platform/services/` 里实现 OpenCodeAdapter，用 HTTP 或官方 SDK 调用 `POST /session`、`POST .../message` 等，完成“创建会话 → 发任务 prompt → 取生成代码”的流程。

更细的接口列表见 `opencode/specs/project.md` 和 `opencode/packages/web/src/content/docs/zh-cn/server.mdx`。
