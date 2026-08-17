# Tests, results, and claim boundaries

The executable cases and full machine-readable runners live in
[MemoryOS](https://github.com/tianhao8687/MemoryOS/tree/main/benchmarks/context_efficiency).
This page is the public plugin-level result index.

## Frozen results

| Evaluation | Result | Supported conclusion |
|---|---|---|
| Packaged RC5 Loader/HMR smoke | 27/27 tests passed after installing the `0.2.0` tarball in a network-disabled container | The bundle composes through the real locked DSH Loader; ordinary off retains only control, strict `no_memory` has zero MemoryOS schemas, dynamic HMR restoration works, and usage remains mounted |
| Natural-language persistent control | PASS: exact Chinese triggers are present; disable/status/health-checked enable execute; state survives restart; corrupt state fails closed; first-success notice is one-time | Users can control MemoryOS by speaking to the model without a shortcut; this proves the tool contract and lifecycle, not that every model will obey every paraphrase |
| Full plugin acceptance | 14/14 hidden validations; 13/14 strict protocols | Install, disable, Full, Progressive, Explain, Delta, usage, cache evidence, isolation, and Loader paths work |
| Medium A/B/C v1 | A and C passed; C used 16.20% fewer input tokens and 21.85% lower cost than A; B remained unpassed at 298 attempts | One-task Progressive efficiency and focus signal; no success-rate claim |
| Medium A/B/C v2 | A/B/C all produced no patch | Earlier diagnosis did not become implementation; no quality gain |
| Medium A/B/C v3 | A/B/C all produced no patch; B cost 23.0% less than A; C cost most | Full context showed a single-task reasoning-cost signal; Progressive long reasoning remained |
| Cross-session v1 | Strict 2/3; every recall, no-memory abstention, and wrong-scope isolation arm passed | Hard-restart recall and scope isolation work; one cross-language lexical source gate remains failed |
| Memory Update | PASS | PostgreSQL 17 becomes superseded, 18 becomes active, and a fresh session returns only 18 |
| Context Eviction A/B | PASS | After proven active-history eviction, baseline abstains and MemoryOS recovers `Glacier-47` |

A separate Requests pair had MemoryOS pass while the baseline failed. Later
MarkupSafe and Seaborn pairs did not pass, so that result is retained as an
individual case rather than generalized success evidence.

## Latest long-term-memory accounting

The update and eviction live-r4 campaign used `dsh-memoryos 0.1.18`,
`deepseek-v4-flash`, and zero Provider retries.

| Write session | Schema estimate | Visible MemoryOS estimate | Provider-exact input |
|---|---:|---:|---:|
| Update A | 598 | 2,593 | 34,061 |
| Update B | 598 | 2,593 | 35,287 |
| Eviction MemoryOS A | 598 | 2,593 | 34,339 |
| Total | 1,794 | 7,779 | 103,687 |

The complete campaign made 24 attempts and recorded 214,165 input, 3,743
output, and 2,206 reasoning tokens. The first two columns use the declared
`unicode-heuristic-v1` estimate; only the Provider input column is exact.

## Incidents and fixes

| Incident | Root cause | Fix/status |
|---|---|---|
| Full mode consumed 54,003,040 input tokens without passing | Wrong anchoring plus unrestricted continuation | Add compact/progressive alternatives and a controller-owned 130% request-before-dispatch guard; do not promote Full as universally better |
| A hidden scorer became visible to one continued Agent | Scoring workspace was inside Agent authority | Quarantine the entire run and move hidden scoring outside the Agent root |
| Cold/warm hashes differed for equivalent requests | Random IDs, UI metadata, and absolute paths entered the hash | Hash only the Provider-visible wire projection and remove volatile persona paths |
| Repeated proposal/409 conflict storm | Broad records, hidden strategy schema, lost structured errors, no pending state | Require atomic keys; expose strategies and errors; block candidate churn until resolution |
| Progressive reasoning spiked after correct retrieval | Agent continued historical/upstream/dependency investigation; reasoning was replayed in later input | Compress payloads and add state-based action-ready/offline fallback; issue is reduced but not claimed solved |
| Chinese recall missed continuous text | FTS5 `unicode61` lacked useful word boundaries | Add bounded CJK n-gram/LIKE fallback in MemoryOS with scope and history guards |
| “Eviction” could still leave the source in active context | Filler alone did not force a 1M-window model to forget | Add audited, evaluation-only complete-turn surface replacement and sentinel checks |
| A zero-schema ordinary off mode could not be re-enabled from chat | With no model-visible control capability, the Agent has nothing executable behind “开启 OS” | Split states: ordinary off keeps only `memoryos_control`; strict A/B `no_memory` remains the separate zero-schema Loader condition |
| Re-enable could be reported while the backend was unavailable | Restoring schemas alone did not prove MemoryOS could serve requests | Require `/api/health` to return `ok=true` before mounting memory tools; failures remain off |

## What is not proven

- Generalized coding-repair success-rate improvement.
- Lower total tokens or cost on every task.
- Superiority of Progressive over Full for every model or repository.
- Compatibility with a DSH release other than the locked RC5 target.
- That controlled evaluation eviction should be a production compaction policy.

The defensible current claim is narrower: the plugin is functional, scoped,
measured, and capable of real cross-session update/recall behavior, with useful
single-task efficiency signals and publicly documented failures.
