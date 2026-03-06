# Codex CLI config.toml 配置参考手册

> 基于官方文档和 GitHub 仓库整理。

---

## 配置文件位置

| 级别 | 路径 | 说明 |
|---|---|---|
| 用户级 | `~/.codex/config.toml` | 个人默认配置 |
| 项目级 | `.codex/config.toml`（项目根目录） | 项目级覆盖（需要项目被信任） |
| 系统级 | `/etc/codex/config.toml`（Unix） | 系统范围默认值 |

## 配置优先级（从高到低）

1. CLI 命令行标志和 `--config` 覆盖
2. Profile 配置档案（`--profile <name>`）
3. 项目级 `config.toml`（最近的文件优先）
4. 用户级 `~/.codex/config.toml`
5. 系统级 `config.toml`
6. 内置默认值

---

## 通用配置项

| 键名 | 类型 | 可选值 / 示例 | 说明 |
|---|---|---|---|
| `model` | string | `"azure/gpt-5.3-codex"` | 默认使用的 AI 模型 |
| `model_provider` | string | `"openai"` / `"oss"` / 自定义 ID | 模型提供商 ID，对应 `[model_providers.xxx]` |
| `model_reasoning_effort` | string | `"none"` / `"low"` / `"medium"` / `"high"` / `"xhigh"` | 推理强度 |
| `plan_mode_reasoning_effort` | string | 同上 | Plan 模式专用推理强度覆盖 |
| `approval_policy` | string | `"untrusted"` / `"on-request"` / `"never"` | 命令审批策略 |
| `sandbox_mode` | string | `"read-only"` / `"workspace-write"` / `"danger-full-access"` | 沙箱访问级别 |
| `model_catalog_json` | string | JSON 文件路径 | 自定义模型目录，启动时加载 |
| `preferred_auth_method` | string | `"apikey"` | 首选认证方式 |
| `web_search` | string | `"live"` / `"cached"` | 网页搜索模式 |
| `token_threshold` | int | 例如 `50000` | 触发历史自动压缩的 token 阈值 |
| `context_window_tokens` | int | 例如 `128000` | 当前模型可用的上下文窗口 token 数 |
| `sqlite_home` | string | 目录路径 | SQLite 状态数据库存储目录 |
| `builtin_instructions_replacement` | string | 自定义指令文本 | 替换默认的 `AGENTS.md` 指令 |
| `notification` | string | 钩子命令 | Agent 完成时的通知钩子 |

---

## 命令审批策略详解

| 值 | 说明 |
|---|---|
| `"untrusted"` | 不受信任的命令弹出确认（推荐） |
| `"on-request"` | 由模型判断是否需要审批 |
| `"never"` | 自动审批所有命令（⚠️ 谨慎使用） |
| `"on-failure"` | **已弃用**，请使用 `"on-request"` |

支持精细化拒绝策略：

```toml
approval_policy = { reject = { ... } }
```

---

## 模型提供商配置

```toml
[model_providers.<提供商ID>]
```

| 键名 | 类型 | 示例 | 说明 |
|---|---|---|---|
| `name` | string | `"MSPBots AI Gateway"` | 提供商显示名称 |
| `base_url` | string | `"https://api.example.com/v1"` | API 基础 URL |
| `env_key` | string | `"OPENAI_API_KEY"` | 读取 API Key 的环境变量名 |
| `wire_api` | string | `"responses"` / `"chat"` | API 通信协议格式 |
| `requires_openai_auth` | bool | `false` | 是否需要 OpenAI 官方认证 |
| `request_max_retries` | int | `3` | HTTP 请求最大重试次数 |
| `stream_max_retries` | int | `2` | SSE 流中断重试次数 |
| `stream_idle_timeout_ms` | int | `30000` | SSE 流空闲超时（毫秒） |
| `query_params` | table | `{ api-version = "2024-02-01" }` | 额外查询参数（如 Azure 的 api-version） |
| `http_headers` | table | `{ X-Custom = "value" }` | 自定义 HTTP 请求头 |
| `env_http_headers` | table | `{ Authorization = "AUTH_TOKEN" }` | 从环境变量读取的 HTTP 请求头 |

### 示例

```toml
[model_providers.mspbots-gateway]
name = "MSPBots AI Gateway"
base_url = "https://aigateway-sandbox.mspbots.ai/v1"
env_key = "OPENAI_API_KEY"
wire_api = "responses"
requires_openai_auth = false
request_max_retries = 3
stream_max_retries = 2
stream_idle_timeout_ms = 30000
```

---

## 平台配置

### Windows

```toml
[windows]
sandbox = "elevated"    # "elevated"(提升权限）或 "standard"（标准权限）
```

---

## 沙箱写入配置

当 `sandbox_mode = "workspace-write"` 时可用：

```toml
[sandbox_workspace_write]
writable_roots = ["/path/to/dir1", "/path/to/dir2"]
exclude_tmpdir_env_var = false
exclude_slash_tmp = false
```

| 键名 | 类型 | 说明 |
|---|---|---|
| `writable_roots` | array | 额外允许写入的目录列表 |
| `exclude_tmpdir_env_var` | bool | 是否排除 `TMPDIR` 环境变量指向的目录 |
| `exclude_slash_tmp` | bool | 是否排除 `/tmp` 目录 |

---

## 项目信任级别

```toml
[projects.'C:\path\to\project']
trust_level = "trusted"    # "trusted"（受信任）或 "untrusted"（不受信任）
```

被标记为 `trusted` 的项目中，Codex 可以执行更多操作而无需额外确认。

---

## 配置档案（Profiles）

命名配置集，通过 `--profile <名称>` 切换：

```toml
[profiles.fast]
model = "gemini-3-flash-preview"
model_reasoning_effort = "low"

[profiles.deep]
model = "azure/gpt-5.3-codex"
model_reasoning_effort = "xhigh"

[profiles.gemini]
model = "gemini-3.1-pro-preview"
model_reasoning_effort = "high"
```

使用方式：

```bash
codex --profile fast "your prompt"
codex --profile deep "complex refactoring task"
```

---

## MCP 服务器配置

```toml
[mcp_servers.<服务器名称>]
```

| 键名 | 类型 | 说明 |
|---|---|---|
| `command` | string | 启动 MCP stdio 服务器的命令 |
| `args` | array | 传递给命令的参数 |
| `env` | table | MCP 服务器的环境变量 |
| `endpoint` | string | MCP HTTP 服务器端点 URL |
| `launcher_command` | string | MCP 启动器命令 |
| `working_directory` | string | MCP 服务器工作目录 |
| `allow_list` | array | 允许的工具名白名单 |
| `deny_list` | array | 禁止的工具名黑名单 |
| `enabled_tools` | array | 启用的工具列表 |
| `http_headers` | table | HTTP 请求头（从环境变量填充） |
| `timeout` | int | 启动或工具执行超时时间 |
| `disable` | bool | 禁用该服务器但保留配置 |

### 示例

```toml
[mcp_servers.my-server]
command = "npx"
args = ["-y", "@some/mcp-server"]
env = { API_KEY = "MY_API_KEY" }
working_directory = "C:\\Users\\Administrator\\project"
allow_list = ["read_file", "write_file"]
timeout = 30000
```

---

## 功能开关

```toml
[features]
web_search = true
```

---

## 终端 UI 配置

```toml
[tui]
alternate_screen = true    # 是否使用终端备用屏幕模式
```

---

## 模型迁移通知

静默迁移提示通知：

```toml
[notice.model_migrations]
"gpt-5.2" = "gpt-5.3-codex"    # 已确认的迁移记录
```

---

## 完整配置示例

```toml
# ===================
# 模型配置
# ===================
model = "azure/gpt-5.3-codex"
model_reasoning_effort = "medium"
model_provider = "mspbots-gateway"
approval_policy = "never"
model_catalog_json = "C:\\Users\\Administrator\\.codex\\custom_models.json"

[model_providers.mspbots-gateway]
name = "MSPBots AI Gateway"
base_url = "https://aigateway-sandbox.mspbots.ai/v1"
env_key = "OPENAI_API_KEY"
wire_api = "responses"
requires_openai_auth = false
request_max_retries = 3
stream_max_retries = 2
stream_idle_timeout_ms = 30000

# ===================
# 配置档案
# ===================
[profiles.fast]
model = "gemini-3-flash-preview"
model_reasoning_effort = "low"

[profiles.deep]
model = "azure/gpt-5.3-codex"
model_reasoning_effort = "xhigh"

# ===================
# 系统与权限
# ===================
[windows]
sandbox = "elevated"

[projects.'C:\Users\Administrator\Desktop\MPD']
trust_level = "trusted"

# ===================
# 通知
# ===================
[notice.model_migrations]
"gpt-5.2" = "gpt-5.3-codex"
```

---

## CLI 命令行覆盖示例

```bash
# 单次运行覆盖模型
codex --model gemini-3.1-pro-preview "refactor auth module"

# 覆盖多个设置
codex -c model=azure/gpt-5.2 -c model_reasoning_effort=high "debug this"

# 使用配置档案
codex --profile fast "quick fix"
```
