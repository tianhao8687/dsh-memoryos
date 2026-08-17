# dsh-memoryos

[![CI](https://github.com/tianhao8687/dsh-memoryos/actions/workflows/ci.yml/badge.svg)](https://github.com/tianhao8687/dsh-memoryos/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![](https://img.shields.io/badge/powered_by-dsh-4D6BFE?style=flat-square&logo=deepseek&logoColor=white)](https://github.com/deepseek-ai/deepseek-harness)

[English](README.md) | [简体中文](README.zh-CN.md)

> 一句话说明：让 DeepSeek Agent 换一个会话、重启一次，仍然知道这个项目以前
> 做过什么决定；决定变了，它会优先拿到新的，而不是继续照着旧结论做。

`dsh-memoryos` 是 DeepSeek Harness（DSH）的长期项目记忆插件，记忆能力由
[MemoryOS](https://github.com/tianhao8687/MemoryOS) 提供。

它不是给模型“凭空加智商”，而是给模型一份能跨会话保存、按项目隔离、可以更新、
有来源可查的项目记忆。它尤其适合需要连续工作几天、经历多个 Session，或者经常要
把任务交给新 Agent 继续做的项目。

## 它到底有什么用

| 真实场景 | 没有长期记忆 | 使用 dsh-memoryos |
|---|---|---|
| 重启后继续项目 | Agent 不知道上次定了什么，要重新解释 | Agent 可以主动查询上次确认过的决定 |
| 技术决定发生变化 | 旧版和新版信息容易混在一起 | 旧记录会被标记为已替换，当前结论只保留新版 |
| 对话太长，早期消息被挤出上下文 | Agent 会忘记早期的重要要求 | 只要已经写入 MemoryOS，原聊天不在窗口里也能找回 |
| 同时维护多个仓库 | 记忆容易串项目 | 每次读取和写入都固定在指定仓库范围内 |
| 想知道插件到底花了多少 Token | 很难区分模型、工具和重试开销 | 分开记录工具 Schema、模型实际看到的记忆和 Provider 精确输入 |
| 临时不想使用长期记忆 | 要改环境变量并重启 | 直接对 Agent 说“关闭 OS”；要恢复就说“开启 OS” |

适合保存的内容包括：

- 已确定的数据库版本、API 约定和兼容性要求；
- 为什么选择或放弃某个实现方案；
- 关键代码位置、已完成的验证和仍未解决的问题；
- 下一次 Session 接手时必须知道的项目事实。

不建议保存密码、API Key、大段日志、未经验证的猜测，或者可以随时从代码里快速
查到的普通信息。

## 用了以后，Agent 会怎么工作

1. DSH 把 `memory_context` 工具交给 Agent。
2. Agent 认为需要项目历史时，自己调用这个工具；用户不需要每次都输入“请回忆”。
3. 插件只向本机 MemoryOS 查询当前仓库的记忆。
4. MemoryOS 先处理新旧冲突，再把尽量短的“当前有效结论”返回给 Agent。
5. Agent 继续查看代码、修改和运行测试。记忆只是证据，不代替代码核对。

第一次成功取到记忆上下文时，Agent 会主动告诉你：

> MemoryOS 已开始工作，正在为当前项目提供跨会话记忆。你随时可以直接说
> “关闭 OS”；需要恢复时说“开启 OS”即可。

这条提示只出现一次。它表示插件确实完成了一次有效调用，不只是“安装命令跑完了”。

默认模式是**只读**，插件不会擅自把所有对话都存起来。只有明确开启
`cross-session-write` 后，Agent 才会获得 `memory_propose` 和
`memory_confirm` 两个写入工具。模型根据当前对话提出候选记忆，再经过确认写入；
遇到冲突时可以替换旧结论、保留两者或拒绝新结论。

## 哪些情况最值得装

- 中等或长期编程任务，需要多次启动 Agent 才能完成；
- 项目有很多不能只靠代码猜出的历史决定；
- 多个 Agent 或多人轮流接手同一仓库；
- 想做严格的有记忆/无记忆 A/B 测试；
- 需要知道记忆工具、重试和模型请求分别用了多少 Token。

以下情况收益通常不大：

- 一次对话就能完成的简单修改；
- 项目没有需要跨会话保留的信息；
- 只是希望插件自动提高所有题目的修复成功率；
- 不愿意运行本地 MemoryOS 服务。

## 快速安装

### 1. 启动 MemoryOS

这个仓库是 DSH 适配器，真正保存和整理记忆的是 MemoryOS 2.3。从 MemoryOS
源码目录启动本地服务：

```console
python -m memoryos --data-dir ./data serve --port 8000 --no-open
```

### 2. 安装插件

当前版本锁定 DeepSeek Harness `0.1.0-rc.5`，对应 commit
`47f943859bef60e4160492346772ded9b24f765a`。

```console
dsh plugin --profile memoryos add github:tianhao8687/dsh-memoryos#v0.2.0
dsh --profile memoryos --dump-config
```

DSH 仍处于 developer preview。升级 DSH 前，请重新运行 Loader 和契约测试。
Git 插件安装可能执行包生命周期代码，高可信环境建议审阅后固定到准确 commit SHA。

### 3. 第一次使用的推荐配置

先从“短上下文、每个 Session 最多调用一次”开始：

```powershell
$env:MEMORYOS_BASE_URL = 'http://127.0.0.1:8000'
$env:MEMORYOS_AUTH_TOKEN = '<本地-memoryos-token>'
$env:MEMORYOS_CONDITION = 'msc_context_only'
$env:MEMORYOS_BUDGET_TOKENS = '512'
$env:MEMORYOS_MAX_CONTEXT_CALLS = '1'
$env:MEMORYOS_RESPONSE_FORMAT = 'deepseek-compact'
dsh --profile memoryos
```

这套配置的目标不是塞给模型越多历史越好，而是先给它一份够用、可执行、当前有效的
项目摘要。模型和 Provider 仍由 DSH 配置；插件不会读取或改写
`DEEPSEEK_API_KEY`。

只要你是有意安装这个插件，首次启动默认就是开启状态；如果你上次说过“关闭 OS”，
插件会记住这个选择，重启后仍保持关闭。

## 直接打字开关，不需要快捷方式

在和 Agent 的正常对话里直接说：

```text
关闭 OS
开启 OS
OS 现在开着吗？
```

模型会调用 `memoryos_control` 工具执行真实开关，不是只回复一句“好的”。

- 说“关闭 OS”后，插件会立即撤下读取、解释和写入记忆的工具，后续请求不再使用
  MemoryOS；只留下一个很小的控制工具，保证你以后还能说“开启 OS”。
- 说“开启 OS”后，插件先检查本机 MemoryOS 服务是否正常；检查成功才恢复记忆工具，
  服务没启动时不会假装开启。
- 开关状态会原子写入本地状态文件，关闭 DSH 或重启电脑后仍然有效。
- 已经进入当前聊天记录的旧记忆无法倒着删除。如果要完全干净的上下文，关闭后再新建
  Session。

严格 A/B 测试的 `MEMORYOS_CONDITION=no_memory` 是例外：它连控制工具都不加载，
MemoryOS Schema 数量为零。这个模式不能在当前进程里靠聊天重新开启，需要换回其他模式
并重启；这样才能保证无记忆基线没有额外 Schema。

## 记忆模式怎么选

| 模式 | 人话解释 | 建议用途 |
|---|---|---|
| `no_memory` | 完全不给模型记忆工具 | 做公平的无记忆基线 |
| `msc_context_only` | 一次拿到短小的当前结论 | **普通用户建议从这里开始** |
| `msc_progressive` | 先看目录，必要时再展开证据 | 多条记录或复杂历史 |
| `msc_full` | 一次返回较完整的项目记忆 | 需要完整上下文的受控实验 |
| `msc_delta` / `msc_delta_core` | 先拿完整内容，之后只拿变化 | 很长且持续变化的 Session |
| `legacy_full` | 旧版完整上下文 | 兼容性测试，不建议新项目默认使用 |

如果只是日常使用，优先选择 `msc_context_only`。只有在确实缺少证据时，再尝试
Progressive 或 Full；不要因为预算上限很大就把所有历史都塞进模型。

## 实际效果和测试结果

以下不是演示文案，而是已经跑过的公开验收：

- **真实安装可用**：打包后安装到锁定的 DSH RC5 Profile，在断网环境中 27/27
  通过；验收覆盖自然语言持久开关、动态撤下/恢复工具、健康检查、损坏状态防误开启以及
  严格零 Schema 基线。
- **主要插件功能可用**：14/14 隐藏验证通过，13/14 严格模式协议通过。
- **记忆可以更新**：Session A 确认 PostgreSQL 17，Session B 改成 18，硬重启后
  Session C 只把 18 当成当前版本。
- **聊天窗口忘了，长期记忆还在**：原消息被确认移出活动上下文后，无记忆 Agent
  回答“不知道”，MemoryOS Agent 找回了 `Glacier-47`。
- **项目隔离有效**：跨会话测试里的错误仓库范围没有读到目标项目的记忆。
- **结果并非全胜**：第一版跨会话严格验收仍然是 2/3，其中一项中文连续文本的
  来源词匹配失败；这个失败没有被改写成通过。
- **没有夸大成功率**：编程 A/B/C 测试看到过单题 Token 和成本改善，但目前不能
  证明它普遍提高代码修复成功率。

完整数据、失败过程和修复记录见
[测试、失败、修复与结论边界](docs/TESTS_AND_RESULTS.md)。

## Token 会不会增加

会。只要给模型增加工具 Schema、工具结果或额外一轮模型调用，输入 Token 就会增加。
这个插件的目标是让增加的 Token 换来真正有用的跨会话信息，并用 Compact 模式控制
开销，而不是宣称“用了记忆反而零成本”。普通“关闭 OS”仍会发送那个很小的控制
Schema；只有严格 `no_memory` A/B 基线才是零 MemoryOS Schema Token。

最新 Memory Update / Context Eviction 测试的三个写入 Session 合计记录了：

| 指标 | Token | 它代表什么 |
|---|---:|---|
| `write_tool_schema_tokens` | 1,794 | 写入工具 Schema 的估算量 |
| `memory_write_visible_tokens` | 7,779 | 写入过程中模型实际看到的 MemoryOS 内容估算量 |
| `provider_input_tokens` | 103,687 | Provider 返回的精确输入量 |

前两项是 `unicode-heuristic-v1` 估算，只有第三项是 Provider 精确值，三者不能混在
一起宣称“节省了多少”。增加工具也会改变 Provider 可见请求，因此可能改变 KV Cache
命中；模型不可见的用量采集器本身不会增加提示词或工具 Schema。

## 常用配置

| 环境变量 | 默认值 | 人话说明 |
|---|---|---|
| `MEMORYOS_ENABLED` | `1` | 只决定“没有旧状态文件时”的初始状态；设为 `0` 可首次以普通关闭模式启动 |
| `MEMORYOS_CONTROL_ENABLED` | `1` | 在非 `no_memory` 模式下给模型开关/查询状态工具 |
| `MEMORYOS_ONBOARDING_NOTICE` | `1` | 第一次成功取到上下文时，让 Agent 告诉用户 OS 已开始工作 |
| `MEMORYOS_STATE_FILE` | 系统配置目录 | 可选；为不同 DSH Profile 指定各自的开关状态文件 |
| `MEMORYOS_BASE_URL` | `http://127.0.0.1:8000` | 本机 MemoryOS 服务地址 |
| `MEMORYOS_AUTH_TOKEN` | 无 | MemoryOS 本地访问令牌 |
| `MEMORYOS_CONDITION` | `msc_progressive` | 选择上面的记忆模式 |
| `MEMORYOS_BUDGET_TOKENS` | `6000` | 一次最多返回多少记忆；建议先用 512 |
| `MEMORYOS_MAX_CONTEXT_CALLS` | 不限 | 每个 Session 最多查几次；`0` 表示不限 |
| `MEMORYOS_RESPONSE_FORMAT` | `json` | 返回 JSON、DeepSeek compact 或 progressive compact |
| `MEMORYOS_TOOL_PROFILE` | `read-only` | 默认只读；写入需显式选择 `cross-session-write` |
| `MEMORYOS_REPOSITORY` | 无 | 固定项目范围；开启写入时必须设置 |
| `MEMORYOS_TASK` | 无 | 当前任务说明，由控制器提供 |
| `MEMORYOS_TIMEOUT_MS` | `30000` | 等待本地 MemoryOS 的最长时间 |

Usage ledger 和受控上下文淘汰属于评测能力。普通用户不需要开启；如果要做实验，请先
阅读 [`cordis.patch.yml`](cordis.patch.yml) 和[架构说明](docs/ARCHITECTURE.md)。

## 数据和安全

- 记忆数据库、冲突处理、检索和 Context Compiler 都在本地 MemoryOS 服务中。
- 插件只访问你配置的 MemoryOS 地址，不负责保存 DeepSeek API Key。
- MemoryOS 返回的内容会进入后续模型请求，所以**不要把密码或密钥写进记忆**。
- 开关状态文件只保存“是否开启”和“首次提示是否已经显示”，不保存记忆正文或密钥。
- 状态文件损坏时采用安全关闭：不会悄悄把记忆工具重新打开。
- 仓库范围由控制器固定，模型不能在写入时随意切换到另一个项目。
- 默认只读；写入和评测专用的上下文淘汰都需要明确开启。

## 验证和卸载

```console
node --test tests/contract.test.mjs tests/loader-composition.test.mjs
dsh --profile memoryos --dump-config
dsh plugin --profile memoryos remove dsh-memoryos
```

当 `DSH_TEST_PROFILE_DIR` 指向已经安装的 RC5 Profile 时，Loader/HMR 测试会走真实
DSH Loader。开发时应先打 tarball 再安装；直接安装目录会变成不受支持的 `link:`
dependency。

## 目前的限制

- 插件目前有意锁定 DSH RC5，不保证兼容其他版本。
- 开关是进程级的；要同时跑有记忆和无记忆 Agent，请分别启动 DSH 进程。
- 默认状态文件由本机启动共享。多个 Profile 要保留各自开关状态时，请给它们设置不同的
  `MEMORYOS_STATE_FILE`。
- “关闭 OS”只影响后续请求，不能删除已经进入当前 Session 聊天记录的记忆文本。
- 不包含 `headless-runner` 的 Profile 在 `--dump-config` 时可能显示一条可选 resume
  overlay 的 missing-entry 警告。这不会阻止记忆工具、用量采集或 Loader/HMR 工作。
- 受控历史淘汰只用于测试，不是生产环境的聊天压缩方案。
- 记忆可能过期或记录错误。涉及代码的结论仍然必须查看当前代码并运行测试。

另见 [SECURITY.md](SECURITY.md)、[CONTRIBUTING.md](CONTRIBUTING.md) 和
[MIT License](LICENSE)。
