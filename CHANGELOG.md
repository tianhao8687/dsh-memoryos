# Changelog

## 0.2.0 - 2026-08-17

- Add the model-callable `memoryos_control` tool so users can type natural
  language such as “关闭 OS”, “开启 OS”, or ask for current status.
- Dynamically remove all context, explain, and write tools while MemoryOS is
  off, while retaining only the small control tool needed to turn it back on.
- Persist the switch and one-time onboarding state with atomic local writes;
  malformed state fails closed instead of silently enabling memory.
- Health-check the local MemoryOS service before re-enabling its tools.
- Add a one-time, first-success notice explaining that MemoryOS is working and
  can be disabled or restored by typing a request to the model.
- Keep the strict `no_memory` evaluation arm completely schema-free, including
  the control tool.
- Align the usage-ledger default condition with the now-enabled
  `msc_progressive` default so unconfigured runs are not mislabeled baseline.
- Add contract and Loader/HMR coverage for disable, restart, re-enable,
  health-check failure, corrupt state, and strict-baseline composition.

## 0.1.18 - 2026-08-17

- Add atomic cross-session write tools with stable semantic keys.
- Expose `supersede`, `keep_both`, and `reject` conflict strategies.
- Preserve structured MemoryOS errors through the DSH bridge and prevent
  candidate churn while a conflict is pending.
- Add Provider-attempt write-token attribution with explicit estimated/exact
  sources.
- Add evaluation-only controlled context eviction and its audit ledger.
- Preserve write keys and confirmed source content in compact rendering.
- Validate Memory Update and Context Eviction end to end with DeepSeek Harness.

## Earlier 0.1.x work

- Split usage collection from conditional memory tools so the baseline remains
  schema-free.
- Add compact context-only and progressive/explain delivery.
- Normalize Provider-visible request hashing for cache comparisons.
- Add request-before-dispatch usage guards and installed Loader/HMR tests.
