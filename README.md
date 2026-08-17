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
- Full, compact, progressive, explain, and delta context modes share one
  repository-scoped MemoryOS service.
- An explicit write profile can capture atomic, source-backed decisions across
  sessions and resolve updates with `supersede`, `keep_both`, or `reject`.
- Provider-exact input/output/cache usage stays separate from estimated
  MemoryOS component attribution.
- Defaults remain read-only. Evaluation writes and controlled context eviction
  require explicit opt-in.

## Install

### 1. Run MemoryOS

This Bundle is the DSH adapter, not the database. Start MemoryOS 2.3 locally and
keep its bearer token private. From a MemoryOS source checkout:

```console
python -m memoryos --data-dir ./data serve --no-open
```

### 2. Install the DSH Bundle

Use a release tag for reproducibility:

```console
dsh plugin --profile memoryos add github:tianhao8687/dsh-memoryos#v0.1.18
dsh --profile memoryos --dump-config
```

For higher-assurance deployments, replace the tag with the exact audited commit
SHA. Git installs execute package lifecycle code; review and pin third-party
plugins before installation.

### 3. Enable read-only memory

```powershell
$env:MEMORYOS_ENABLED = '1'
$env:MEMORYOS_BASE_URL = 'http://127.0.0.1:8000'
$env:MEMORYOS_AUTH_TOKEN = '<local-memoryos-token>'
$env:MEMORYOS_CONDITION = 'msc_context_only'
$env:MEMORYOS_BUDGET_TOKENS = '512'
$env:MEMORYOS_MAX_CONTEXT_CALLS = '1'
$env:MEMORYOS_RESPONSE_FORMAT = 'deepseek-compact'
dsh --profile memoryos
```

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

## Architecture

| Cordis component | Mounted when | Model-visible effect |
|---|---|---|
| `dsh-memoryos/usage` | Always | None; records attempts and Provider usage |
| `dsh-memoryos` | `MEMORYOS_ENABLED=1` | Registers the selected memory tools |
| `dsh-memoryos/resume` | A resume session id is configured | Replaces the headless runner for controlled continuation |

The Bundle communicates only with the configured loopback MemoryOS HTTP
service. SQLite, migrations, retrieval, Current Truth, conflict relations, and
Context Compiler logic remain in MemoryOS. See [architecture and coupling](docs/ARCHITECTURE.md).

## Model experience

### What the model sees

With the plugin disabled, the model receives no MemoryOS tools or text. With it
enabled, DSH sends the selected tool schemas. Retrieved context becomes visible
only after the model calls a memory tool.

### Token effect

Input tokens normally increase because schemas, tool results, and an additional
model turn are real input. Compact mode bounds this overhead; it does not claim
zero cost. In the latest update/eviction campaign, the three write sessions
recorded `1,794 / 7,779 / 103,687` total schema-estimate / visible-memory-estimate
/ Provider-exact input tokens.

### KV-cache effect

Enabling a tool changes the Provider-visible request and therefore its cache
key. The usage collector itself contributes no prompt text or tool schema.

## Tested outcomes

- Packaged installation against the locked DSH RC5 profile: 23/23 contract and
  real Loader/HMR tests passed in an offline container.
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
| `MEMORYOS_ENABLED` | `0` | Mount memory tools when set to `1` |
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
