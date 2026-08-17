# Security policy

## Supported version

Security fixes are applied to the latest `0.2.x` release. Compatibility is
currently limited to DeepSeek Harness `0.1.0-rc.5` at the commit recorded in
[`harness-lock.json`](harness-lock.json).

## Report a vulnerability

Use GitHub's private security-advisory flow for this repository. Do not include
API keys, MemoryOS bearer tokens, private memory contents, or live session logs
in a public issue.

Include the plugin version, DSH commit, operating system, affected tool profile,
and a minimal redacted reproduction. You should receive an acknowledgement
within seven days.

## Trust boundaries

- The plugin reads the MemoryOS bearer token from the configured environment;
  it never writes the token to its usage ledgers.
- Provider credentials remain owned by DSH and are not read by this Bundle.
- The default tool profile is read-only. Cross-session writes require an
  explicit profile and fixed repository scope.
- The local control-state file stores only the enabled flag and whether the
  first-use notice has been shown. It stores no memory contents or credentials.
- A malformed control-state file fails closed: memory tools stay unavailable
  until the state is repaired or the user successfully enables MemoryOS.
- MemoryOS should remain bound to loopback unless a separately authenticated
  deployment has been reviewed.
- Git plugin installation can execute package lifecycle code. Audit and pin an
  exact commit for high-assurance environments.
