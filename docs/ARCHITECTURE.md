# Architecture and coupling

## Request path

1. DSH loads `cordis.patch.yml` as the Bundle patch.
2. `dsh-memoryos/usage` attaches to Provider lifecycle events. It records a
   pre-dispatch attempt and, after a successful response, the Provider usage.
3. Outside strict `no_memory`, `dsh-memoryos` registers `memoryos_control` and
   reads the persisted local switch state. When that state is enabled, it also
   registers the tools selected by the condition and tool profile.
4. “关闭 OS” and “开启 OS” are model tool calls, not string substitutions. A
   disable dynamically disposes the memory tool registrations. An enable first
   calls `/api/health`, then restores them and persists the state.
5. A memory tool call is sent to the configured local MemoryOS HTTP service
   with its bearer token.
6. The plugin removes volatile/accounting fields and renders only the selected
   model-visible contract.

The usage collector is mounted in both baseline and treatment. It contributes
no prompt text or tool schema. The entire tools component is absent from strict
`no_memory`, including the control tool.

## Switch states

| State | Model-visible MemoryOS schemas | How it changes |
|---|---|---|
| Enabled | Control plus selected context/explain/write tools | Default after intentional install, or a successful `enable` control call |
| Ordinary off | Control only | A successful `disable` control call, initial `MEMORYOS_ENABLED=0`, or fail-closed state loading |
| Strict `no_memory` | None | Loader omits the component before tool registration |

The control state contains only `enabled`, `onboarding_notice_shown`, and a
schema version. It is atomically replaced at `MEMORYOS_STATE_FILE` (or the OS
configuration-directory default). A malformed file fails closed. State is
process/profile control, not a model-authored memory record. Already-rendered
memory in an existing transcript cannot be removed retroactively.

The first successful `memory_context` result carries a one-time instruction for
the Agent to tell the user that MemoryOS is active and can be switched in chat.
A failed HTTP call does not consume this notice. The notice state is persisted
so it is not repeated after restart.

## Ownership boundary

| Concern | Owner |
|---|---|
| DSH lifecycle, tool registration, Provider events | This Bundle |
| Model-visible compact/progressive rendering | This Bundle |
| Natural-language control tool and local switch state | This Bundle |
| SQLite and migrations | MemoryOS |
| Scope filtering and retrieval | MemoryOS |
| Current Truth, supersedes relations, freshness | MemoryOS |
| Context Atom selection and compilation | MemoryOS |
| A/B/C orchestration and hidden scoring | MemoryOS benchmark runners |

## Intentional DSH coupling

The Bundle depends on RC5 Cordis component IDs, tool registration, session
surface, Provider request and completion events, Loader patches, and the DSH
DeepSeek wire shape used for evidence hashing. These are adapter responsibilities,
not generic MemoryOS core behavior. `harness-lock.json` freezes the exact target.

An upgrade is accepted only after:

- contract tests pass;
- `dsh --profile <name> --dump-config` contains the Bundle patch;
- an installed-profile Loader/HMR test passes;
- baseline tools remain absent and usage remains mounted;
- enabled read-only and explicit write schemas match their frozen contracts.

## Anti-over-coupling rules

The plugin contains no task answer, target repository/file, dependency name,
model name, or fixed “edit on step N” rule. One-resolved-record auto-expansion,
offline fallback wording, atomic write keys, and conflict resolution are
state-based behaviors that apply across repositories and models.

Evaluation-only surfaces are opt-in:

- `cross-session-write` adds the two write tools;
- controlled context eviction requires a nonzero evaluation history limit;
- usage guards require a controller-owned guard file.

The production default remains read-only and does not evict history.

Natural-language control is deliberately generic: the tool schema maps only
explicit enable, disable, and status intent to a small state machine. It contains
no task-, repository-, model-, language-, or benchmark-specific heuristic. The
strict baseline remains independently enforceable at Loader composition time,
so the convenience controller cannot contaminate zero-schema A/B measurements.

## Security and accounting

Tool scope is fixed by the adapter, not supplied by the model. Write proposals
must cite conversation evidence. MemoryOS tokens are component estimates using
`unicode-heuristic-v1`; Provider total input remains the exact usage returned by
DeepSeek. Raw prompts, bearer tokens, and Provider keys are not written to the
evidence ledgers. The switch-state file stores no MemoryOS payload or secret.
