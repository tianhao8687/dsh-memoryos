# dsh-memoryos

[![CI](https://github.com/tianhao8687/dsh-memoryos/actions/workflows/ci.yml/badge.svg)](https://github.com/tianhao8687/dsh-memoryos/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![](https://img.shields.io/badge/powered_by-dsh-4D6BFE?style=flat-square&logo=deepseek&logoColor=white)](https://github.com/deepseek-ai/deepseek-harness)

[English](README.md) | [简体中文](README.zh-CN.md)

面向 DeepSeek Harness（DSH）的证据优先项目长期记忆插件，由
[MemoryOS](https://github.com/tianhao8687/MemoryOS) 提供后端能力。

`dsh-memoryos` 是纯 JavaScript DSH Bundle。它把有 scope 的项目记忆工具和
Provider 精确用量采集接入 DSH，但不会把 MemoryOS 的持久化、检索、Truth 解析或
Context Compiler 复制进 Agent 进程。

> 当前有意锁定 DeepSeek Harness `0.1.0-rc.5`，commit 为
> `47f943859bef60e4160492346772ded9b24f765a`。DSH 仍是 developer preview；
> 升级前必须重跑 Loader 与契约验收。

## 为什么使用它

- 真正的 `no_memory` 对照不暴露任何 MemoryOS 工具 Schema，同时保留相同且模型不可见的用量采集器。
- Full、Compact、Progressive、Explain 和 Delta 模式共享同一个 repository scope MemoryOS 服务。
- 显式写入 Profile 能跨 Session 保存原子化、有对话证据的决定，并用 `supersede`、`keep_both` 或 `reject` 处理更新。
- Provider 精确输入/输出/缓存用量与 MemoryOS 组件估算严格分开。
- 默认保持只读；评测写入和受控上下文淘汰都必须显式开启。

## 安装

### 1. 启动 MemoryOS

本仓库只是 DSH 适配器，不是数据库。从 MemoryOS 2.3 源码目录启动本地服务：

```console
python -m memoryos --data-dir ./data serve --no-open
```

### 2. 安装 DSH Bundle

使用 Release tag 保持可复现：

```console
dsh plugin --profile memoryos add github:tianhao8687/dsh-memoryos#v0.1.18
dsh --profile memoryos --dump-config
```

高可信环境应把 tag 换成审计过的准确 commit SHA。Git 插件安装可能执行包生命周期代码，请先审阅再固定版本。

### 3. 开启只读记忆

```powershell
$env:MEMORYOS_ENABLED = '1'
$env:MEMORYOS_BASE_URL = 'http://127.0.0.1:8000'
$env:MEMORYOS_AUTH_TOKEN = '<本地-memoryos-token>'
$env:MEMORYOS_CONDITION = 'msc_context_only'
$env:MEMORYOS_BUDGET_TOKENS = '512'
$env:MEMORYOS_MAX_CONTEXT_CALLS = '1'
$env:MEMORYOS_RESPONSE_FORMAT = 'deepseek-compact'
dsh --profile memoryos
```

Bundle 不会选择或改写 `agent-default-model`，模型与 Provider 仍由 DSH 配置。插件不读取 `DEEPSEEK_API_KEY`，该密钥由 DSH 管理。

## 模式

| 条件 | 模型可见记忆面 | 用途 |
|---|---|---|
| `no_memory` | 无 | 匹配基线，只保留用量采集 |
| `legacy_full` | 完整旧版上下文 | 兼容性实验 |
| `msc_full` | 单次 Minimum Sufficient Context | 一般已解析项目上下文 |
| `msc_progressive` | 紧凑索引 + 按需 Explain | 多记录或证据密集任务 |
| `msc_context_only` | 一次无参数紧凑调用 | 有界 DeepSeek 编程 Session |
| `msc_delta` / `msc_delta_core` | Full 后接 Delta | 上下文变化的长任务 |

可选 `cross-session-write` Profile 增加 `memory_propose` 和 `memory_confirm`。每次 proposal 必须只有一个可独立更新事实、一个稳定 semantic key 和一段对话证据，repository scope 由控制器固定。普通使用保持默认 `read-only`。

## 架构

| Cordis 组件 | 挂载条件 | 模型可见影响 |
|---|---|---|
| `dsh-memoryos/usage` | 始终 | 无；记录请求尝试与 Provider usage |
| `dsh-memoryos` | `MEMORYOS_ENABLED=1` | 注册选定的记忆工具 |
| `dsh-memoryos/resume` | 配置 resume session id | 为受控续跑替换 headless runner |

Bundle 只访问配置的 loopback MemoryOS HTTP 服务。SQLite、迁移、检索、Current Truth、冲突关系和 Context Compiler 仍属于 MemoryOS。详见[架构与耦合](docs/ARCHITECTURE.md)。

## 模型实际体验

### 模型能看到什么

插件关闭时，模型看不到 MemoryOS 工具或文本。开启后，DSH 发送选定工具 Schema；只有模型调用工具后，返回的上下文才会进入后续请求。

### Token 影响

输入 Token 通常会增加，因为 Schema、工具结果和新增模型轮次都是真实输入。Compact 模式只负责限制开销，不声称零成本。最新 Update/Eviction campaign 的三个写入会话合计记录为 `1,794 / 7,779 / 103,687`：分别是 Schema 估算、模型可见 MemoryOS 估算和 Provider 精确输入。

### KV Cache 影响

增加工具会改变 Provider 可见请求，因此会改变缓存 key；用量采集器本身不增加提示词或工具 Schema。

## 已测试结果

- 在离线容器中把打包产物安装到锁定的 DSH RC5 Profile 后，23/23 契约与真实 Loader/HMR 测试通过。
- 全功能验收：14/14 隐藏验证通过，13/14 严格模式协议通过。
- Memory Update：PostgreSQL 17 被 18 正确 supersede，新 Session 只返回 18。
- Context Eviction A/B：原始消息确认离开活动历史后，无记忆回答“不知道”，MemoryOS 恢复 `Glacier-47`。
- Cross-session v1 的严格结果仍是 2/3；虽然全部 recall、baseline 和错 scope 隔离臂都通过，也不改写成 3/3。
- 编程测试出现过单题效率收益，但尚未证明普遍提高修复成功率。

详见[测试、失败、修复与结论边界](docs/TESTS_AND_RESULTS.md)。

## 配置

| 环境变量 | 默认值 | 含义 |
|---|---|---|
| `MEMORYOS_ENABLED` | `0` | 设为 `1` 时挂载记忆工具 |
| `MEMORYOS_BASE_URL` | `http://127.0.0.1:8000` | 本地 MemoryOS 地址 |
| `MEMORYOS_AUTH_TOKEN` | 无 | 本地 MemoryOS bearer token |
| `MEMORYOS_CONDITION` | `msc_progressive` | 上下文交付模式 |
| `MEMORYOS_BUDGET_TOKENS` | `6000` | MemoryOS 响应预算 |
| `MEMORYOS_MAX_CONTEXT_CALLS` | 不限 | 每 Session 调用上限；`0` 表示不限 |
| `MEMORYOS_RESPONSE_FORMAT` | `json` | JSON、DeepSeek compact 或 progressive compact |
| `MEMORYOS_TOOL_PROFILE` | `read-only` | `read-only` 或显式 `cross-session-write` |
| `MEMORYOS_REPOSITORY` | 无 | 固定 repository scope；写入时必需 |
| `MEMORYOS_TASK` | 无 | 控制器提供的任务说明 |
| `MEMORYOS_TIMEOUT_MS` | `30000` | 本地 MemoryOS 超时 |

Usage ledger 和受控淘汰变量属于评测基础设施，开启前请阅读 [`cordis.patch.yml`](cordis.patch.yml) 和架构文档。

## 验证与移除

```console
node --test tests/contract.test.mjs tests/loader-composition.test.mjs
dsh --profile memoryos --dump-config
dsh plugin --profile memoryos remove dsh-memoryos
```

当 `DSH_TEST_PROFILE_DIR` 指向已安装的 RC5 profile 时，Loader/HMR 测试会真实运行。开发脚本会先打 tarball 再安装；直接目录安装会变成 `link:` dependency，不受支持。

## 已知限制

- 开关是进程级的；并发的开启/关闭 Agent 应使用不同 DSH 进程。
- 插件有意耦合 RC5 Cordis 生命周期与事件面。
- 不包含 `headless-runner` 的 Profile 在 `--dump-config` 时可能对可选 resume overlay 输出非致命的 missing-entry 警告；记忆工具、用量采集和 Loader/HMR 组合仍可通过。该 overlay 仅用于受控续跑评测。
- 受控历史淘汰仅用于评测。
- 记忆是证据，不是跳过代码核对与测试的权威。

另见 [SECURITY.md](SECURITY.md)、[CONTRIBUTING.md](CONTRIBUTING.md) 与 [MIT License](LICENSE)。
