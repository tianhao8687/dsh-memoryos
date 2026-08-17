# dsh-memoryos

[![CI](https://github.com/tianhao8687/dsh-memoryos/actions/workflows/ci.yml/badge.svg)](https://github.com/tianhao8687/dsh-memoryos/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![](https://img.shields.io/badge/powered_by-dsh-4D6BFE?style=flat-square&logo=deepseek&logoColor=white)](https://github.com/deepseek-ai/deepseek-harness)

[English](README.md) | **[中文用户：查看通俗版说明](README.zh-CN.md)**

Evidence-first long-term project memory for DeepSeek Harness (DSH), powered by
[MemoryOS](https://github.com/tianhao8687/MemoryOS).

`dsh-memoryos` is a plain-JavaScript DSH Bundle. It adds scoped project-memory
tools and exact Provider-usage collection without moving MemoryOS persistence,
retrieval, truth resolution, or context compilation into the Agent process.

> Compatibility is intentionally pinned to DeepSeek Harness `0.1.0-rc.5` at
> commit `47f943859bef60e4160492346772ded9b24f765a`. DSH is a developer preview;
> re-run Loader and contract acceptance before upgrading it.

## Why this plugin

- A true `no_memory` arm exposes zero MemoryOS tool schemas while retaining the
  same model-invisible usage collector.
- Users can type “关闭 OS”, “开启 OS”, or ask whether MemoryOS is active; the
  model calls a dedicated control tool and the choice survives DSH restarts.
- Full, compact, progressive, explain, and delta context modes share one
  repository-scoped MemoryOS service.
- An explicit write profile can capture atomic, source-backed decisions across
  sessions and resolve updates with `supersede`, `keep_both`, or `reject`.
- Provider-exact input/output/cache usage stays separate from estimated
  MemoryOS component attribution.
- Defaults remain read-only. Evaluation writes and controlled context eviction
  require explicit opt-in.

## Evidence-first differentiation

Many DSH memory plugins already have useful unit, integration, smoke, or package
verification. `dsh-memoryos` does not claim that having tests is unique. Its
focus is a stricter layer of **controlled real-Agent evidence** around the memory
claims themselves:

- a true `no_memory` baseline with zero model-visible MemoryOS schemas;
- normal development conversations where the Agent must write memory through
  the plugin rather than controller-side seeding or repair;
- hard restarts of the Agent/MCP/MemoryOS process chain followed by fresh-session
  recall from persistent storage;
- wrong-scope negative controls that must recover none of the target canaries;
- Current Truth update tests where a newer fact supersedes an older fact and a
  fresh session receives only the active truth;
- controlled context-eviction A/B tests proving the original turn is absent from
  retained history before comparing `no_memory` with MemoryOS recall;
- Provider-exact input/output/cache usage recorded separately from estimated
  MemoryOS schema and visible-memory attribution;
- explicit claim boundaries: a passing memory mechanism is not relabeled as a
  generalized coding-success or token-efficiency improvement without matching
  evidence.

The goal is not to win by feature count. It is to make each memory claim
reproducible, falsifiable, and scoped to the evidence that actually passed.

## Install

### 1. Run MemoryOS

This Bundle is the DSH adapter, not the database. Start MemoryOS 2.3 locally and
keep its bearer token private. From a MemoryOS source checkout:

```console
python -m memoryos --data-dir ./data serve --port 8000 --no-open
```

### 2. Install the DSH Bundle

Use a release tag for reproducibility:

```console
dsh plugin --profile memoryos add github:tianhao8687/dsh-memoryos#v0.2.0
dsh --profile memoryos --dump-config
```

For higher-assurance deployments, replace the tag with the exact audited commit
SHA. Git installs execute package lifecycle code; review and pin third-party
plugins before installation.

### 3. Launch with bounded read-only memory

```powershell
$env:MEMORYOS_BASE_URL = 'http://127.0.0.1:8000'
$env:MEMORYOS_AUTH_TOKEN = '<local-memoryos-token>'
$env:MEMORYOS_CONDITION = 'msc_context_only'
$env:MEMORYOS_BUDGET_TOKENS = '512'
$env:MEMORYOS_MAX_CONTEXT_CALLS = '1'
$env:MEMORYOS_RESPONSE_FORMAT = 'deepseek-compact'
dsh --profile memoryos
```

An intentionally installed profile starts with MemoryOS on unless its persisted
state says otherwise. After the first successful context fetch, the Agent tells
the user once that MemoryOS has started working and that it can be turned off or
on in chat.

The Bundle never selects or rewrites `agent-default-model`. Configure the model
and Provider in DSH. It does not read `DEEPSEEK_API_KEY`; DSH owns that secret.

## Modes

| Condition | Model-visible memory surface | Intended use |
|---|---|---|
| `no_memory` | None | Matched baseline; usage collection only |
| `legacy_full` | Full legacy context | Compatibility experiments |
| `msc_full` | Minimum Sufficient Context in one response | General resolved project context |
| `msc_progressive` | Compact index plus selective explain | Evidence-heavy or multi-record work |
| `msc_context_only` | One argument-free compact context call | Bounded DeepSeek coding sessions |
| `msc_delta` / `msc_delta_core` | Full context followed by delta | Long sessions with changing context |

The optional `cross-session-write` tool profile adds `memory_propose` and
`memory_confirm`. Every proposal needs one stable semantic key, one independently
updateable fact, and a conversation excerpt. The repository scope is fixed by
the controller. Keep the default `read-only` profile for ordinary use unless
the write policy has been reviewed.

## Turn MemoryOS on or off in chat

No desktop shortcut or shell command is required. Speak to the Agent normally:

```text
关闭 OS
开启 OS
OS 现在开着吗？
```

The model must call `memoryos_control`; the plugin does not merely match and
pretend to obey the text. A normal disable persists the choice and dynamically
removes every MemoryOS context, explain, and write schema. One small control
schema remains so a later “开启 OS” can work. Re-enabling first checks the local
MemoryOS health endpoint and only restores the memory tools if it succeeds.

The strict `MEMORYOS_CONDITION=no_memory` evaluation arm is different: the
entire component is absent, so it has zero MemoryOS schemas, including no
control tool. Restart that baseline with another condition to enable memory.

## Architecture

| Cordis component | Mounted when | Model-visible effect |
|---|---|---|
| `dsh-memoryos/usage` | Always | None; records attempts and Provider usage |
| `dsh-memoryos` | Any condition except strict `no_memory` | Always exposes control; dynamically exposes the selected memory tools while on |
| `dsh-memoryos/resume` | A resume session id is configured | Replaces the headless runner for controlled continuation |

The Bundle communicates only with the configured loopback MemoryOS HTTP
service. SQLite, migrations, retrieval, Current Truth, conflict relations, and
Context Compiler logic remain in MemoryOS. See [architecture and coupling](docs/ARCHITECTURE.md).

## Model experience

### What the model sees

In ordinary off mode, the model receives only `memoryos_control`, not memory
context or write tools. In strict `no_memory`, it receives no MemoryOS schema at
all. When enabled, DSH sends the selected schemas; retrieved context becomes
visible only after the model calls a memory tool.

### Token effect

Input tokens normally increase because schemas, tool results, and an additional
model turn are real input. Compact mode bounds this overhead; it does not claim
zero cost. Ordinary off mode still sends the small control schema; use strict
`no_memory` when an A/B arm must have zero MemoryOS schema tokens. In the latest
update/eviction campaign, the three write sessions
recorded `1,794 / 7,779 / 103,687` total schema-estimate / visible-memory-estimate
/ Provider-exact input tokens.

### KV-cache effect

Enabling a tool changes the Provider-visible request and therefore its cache
key. The usage collector itself contributes no prompt text or tool schema.

## Tested outcomes

- Packaged installation against the locked DSH RC5 profile: 27/27 contract and
  real Loader/HMR tests passed offline, including persistent natural-language
  control, strict zero-schema baseline, and dynamic tool removal/restoration.
- Full plugin acceptance: 14/14 hidden validations passed; 13/14 strict mode
  protocols passed.
- Memory Update: PostgreSQL 17 was superseded by 18; a fresh session returned
  only 18.
- Context Eviction A/B: after the original turn was proven absent from active
  history, `no_memory` answered “unknown” and MemoryOS recovered `Glacier-47`.
- Cross-session v1 remains a strict 2/3 campaign, despite all recall, baseline,
  and wrong-scope isolation arms passing. It is not relabeled as 3/3.
- Coding evaluations show useful single-task efficiency signals but do not yet
  prove generalized repair-success improvement.

Read [tests, failures, fixes, and claim boundaries](docs/TESTS_AND_RESULTS.md).

## Configuration

| Environment variable | Default | Meaning |
|---|---|---|
| `MEMORYOS_ENABLED` | `1` | Initial state only when no persisted control state exists; `0` starts in ordinary off mode |
| `MEMORYOS_CONTROL_ENABLED` | `1` | Expose the model-callable on/off/status control tool outside strict `no_memory` |
| `MEMORYOS_ONBOARDING_NOTICE` | `1` | Ask the Agent to relay the first-success notice once |
| `MEMORYOS_STATE_FILE` | OS config directory | Optional per-profile path for persistent switch/onboarding state |
| `MEMORYOS_BASE_URL` | `http://127.0.0.1:8000` | Local MemoryOS endpoint |
| `MEMORYOS_AUTH_TOKEN` | none | Local MemoryOS bearer token |
| `MEMORYOS_CONDITION` | `msc_progressive` | Context-delivery condition |
| `MEMORYOS_BUDGET_TOKENS` | `6000` | MemoryOS response budget |
| `MEMORYOS_MAX_CONTEXT_CALLS` | unlimited | Per-session context-call ceiling; `0` means unlimited |
| `MEMORYOS_RESPONSE_FORMAT` | `json` | `json`, `deepseek-compact`, or progressive compact mode |
| `MEMORYOS_TOOL_PROFILE` | `read-only` | `read-only` or explicit `cross-session-write` |
| `MEMORYOS_REPOSITORY` | none | Fixed repository scope; required for writes |
| `MEMORYOS_TASK` | none | Controller-owned task description |
| `MEMORYOS_TIMEOUT_MS` | `30000` | Local MemoryOS request timeout |

Usage-ledger and controlled-eviction variables are evaluation infrastructure;
see [`cordis.patch.yml`](cordis.patch.yml) and the architecture document before
enabling them.

## Verify and remove

```console
node --test tests/contract.test.mjs tests/loader-composition.test.mjs
dsh --profile memoryos --dump-config
dsh plugin --profile memoryos remove dsh-memoryos
```

The Loader/HMR test runs automatically when `DSH_TEST_PROFILE_DIR` points to an
installed RC5 profile. Local contributor scripts pack a tarball before install;
a direct directory install becomes a `link:` dependency and is unsupported.

## Known limitations

- Process-wide enablement means concurrent enabled and disabled Agents should
  use separate DSH launches.
- The switch controls subsequent requests. Memory already copied into the
  current chat transcript cannot be retracted; start a new Session when a fully
  clean context is required.
- The default state path is shared by local launches. Set a distinct
  `MEMORYOS_STATE_FILE` for profiles that need independent switch state.
- The plugin is intentionally coupled to RC5 Cordis lifecycle and event surfaces.
- Profiles without `headless-runner` can emit a non-fatal missing-entry warning
  for the optional resume overlay during `--dump-config`; memory tools, usage
  collection, and Loader/HMR composition still pass. The overlay is only needed
  for controlled continuation benchmarks.
- The controlled history-eviction hook is evaluation-only.
- Memory guidance remains evidence, not authority; verify code-related facts in
  the checkout and run tests.

See [SECURITY.md](SECURITY.md), [CONTRIBUTING.md](CONTRIBUTING.md), and the
[MIT license](LICENSE).