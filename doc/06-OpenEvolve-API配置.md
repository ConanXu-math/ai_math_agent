# OpenEvolve 的 API 配置

OpenEvolve 通过 **环境变量** 和 **YAML 配置文件** 使用 LLM（任意 OpenAI 兼容接口）。API Key 不写死在代码里，也不单独存到一个“凭证文件”。

---

## 一、API Key 放哪

- **推荐：环境变量**  
  运行前在终端里设置：
  ```bash
  export OPENAI_API_KEY="你的key"
  ```
  用 Gemini 时文档里也写的是同一条变量：
  ```bash
  export OPENAI_API_KEY="your-gemini-api-key"  # 用 OPENAI_API_KEY 即可
  ```
  OpenEvolve 会读 `OPENAI_API_KEY`，没在配置文件里写 key 时就用它。

- **可选：写在配置里（仍建议用环境变量引用）**  
  在某个任务的 `config.yaml` 里可以写：
  ```yaml
  llm:
    api_key: "${OPENAI_API_KEY}"   # 从环境变量读，不把 key 写死在文件里
    api_base: "https://api.openai.com/v1"
    primary_model: "gpt-4o"
  ```
  支持 `${VAR}` 解析，未设置会报错，适合用 env 统一管理 key。

- **小结**：Key 只存在于 **当前 shell 的环境变量**（或你在配置里写的 `${OPENAI_API_KEY}` 指向的那里），**没有**类似 OpenCode 的 `~/.local/share/opencode/auth.json` 那种“OpenEvolve 专用凭证文件”。

---

## 二、模型和端点放哪

在**每个示例/任务自己的 config 文件**里，一般是项目下的 `config.yaml`（例如 `examples/function_minimization/config.yaml`）。主要写在 `llm` 下：

| 配置项 | 含义 | 示例 |
|--------|------|------|
| `llm.api_base` | API 端点（OpenAI 兼容） | `https://api.openai.com/v1`、`http://localhost:11434/v1`（Ollama）、`https://generativelanguage.googleapis.com/v1beta/openai/`（Gemini） |
| `llm.primary_model` | 主模型名 | `gpt-4o`、`gemini-2.5-flash`、`qwen2.5:7b`（Ollama） |
| `llm.secondary_model` | 辅模型（可选） | 同上 |
| `llm.api_key` | 可选，一般用 `${OPENAI_API_KEY}` | 见上 |

示例（Gemini）：

```yaml
llm:
  api_base: "https://generativelanguage.googleapis.com/v1beta/openai/"
  primary_model: "gemini-2.5-flash"
  # api_key 不写则用环境变量 OPENAI_API_KEY
```

示例（本地 Ollama，通常不需 key）：

```yaml
llm:
  api_base: "http://localhost:11434/v1"
  primary_model: "qwen2.5:7b"
```

---

## 三、可选：API 端点用环境变量

除 `OPENAI_API_KEY` 外，OpenEvolve 在**没有配置文件**或未在配置里指定时会用：

- `OPENAI_API_BASE`：默认 `https://api.openai.com/v1`，可改成自建或代理地址。

有配置文件时以 `config.yaml` 里的 `llm.api_base` 为准。

---

## 四、和 OpenCode 的区别

| 项目 | API Key 存放位置 |
|------|------------------|
| **OpenCode** | `/connect` 存的在 `~/.local/share/opencode/auth.json`；或环境变量（由 OpenCode 读） |
| **OpenEvolve** | 仅环境变量 `OPENAI_API_KEY`（或 config 里 `${OPENAI_API_KEY}`），没有单独凭证文件 |

所以：**OpenEvolve 的 API 是“环境变量 + 每个任务自己的 config.yaml”**，key 不在某个固定“OpenEvolve 的配置文件”里，而在你 export 的 shell 环境或引用的 env 里。
