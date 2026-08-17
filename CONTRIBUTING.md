# Contributing

`dsh-memoryos` is a deliberately thin adapter. Persistence, retrieval, Current
Truth, conflict relations, and context compilation belong in MemoryOS; DSH
lifecycle integration and model-visible rendering belong here.

## Development checks

Use Node.js `22.19.0` or a compatible Node 24 release:

```console
node --test tests/contract.test.mjs tests/loader-composition.test.mjs
npm pack --dry-run --ignore-scripts
```

To exercise the real Cordis Loader/HMR path, install the tarball into a DSH
RC5 profile and set `DSH_TEST_PROFILE_DIR` to that profile directory before
running the same test command.

## Change rules

- Keep `no_memory` free of MemoryOS schemas and prompt text.
- Keep the default profile read-only.
- Do not add model-, repository-, benchmark-, dependency-, answer-, or fixed
  step-specific guidance to the plugin.
- Preserve Provider-exact totals separately from estimated component counts.
- Add a contract test for every model-visible schema or rendering change.
- Re-run installed-profile Loader/HMR acceptance after any DSH surface change.
- Document failed evaluations and limitations alongside positive results.

Open a focused pull request with the compatibility target, test evidence, and a
short explanation of whether the change affects the model-visible request.
