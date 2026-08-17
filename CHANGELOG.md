# Changelog

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
