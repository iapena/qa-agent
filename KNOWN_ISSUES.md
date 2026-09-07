# KNOWN_ISSUES.md

Shared memory for QA Agent Flamme. Records harness quirks: non-obvious 
behaviors of the test environment or harness that bit an agent once and are
worth remembering. This is NOT a product bug tracker (those live in GitLab);
it is about how the testing setup itself behaves.

All four agents read this file at startup (AGENTS.md, section 7). Entries are
PROPOSED by agents and written only after human approval (AGENTS.md, section 2).

## How to use

- One block per issue, newest at the bottom. IDs are incremental: KI-001, KI-002, ...
- Reference an issue from code or logs by its ID, e.g. `// see KI-003`.
- When an issue is fixed, set `Status: resolved` (keep the entry for history;
  do not delete it).

## Entry format

```
### KI-NNN: <short title>
- Status: open | resolved  (resolved entries are kept for history, never deleted)
- Date: YYYY-MM-DD
- Discovered by: <agent> (Zenith | Flare | Amber | Copper)
- Layer: Context | Memory | Tools | Model | Orchestration
- Symptom: what was observed (the flaky / false result).
- Workaround: what resolves it.
- Scope: where it applies (environment, feature, viewport, ...).
```

Layer vocabulary is the harness layer set from AGENTS.md, section 3, shared
across the whole system (same terms used in each agent's Obstacles Encountered).

---

## Issues

### KI-000: [EXAMPLE - remove when the first real issue is added]
- Status: resolved
- Date: 2026-09-05
- Discovered by: Copper
- Layer: Context
- Symptom: The login submit selector `#submit` becomes `#submit-btn` on a mobile viewport, so the mobile run fails to find the element.
- Workaround: Use a resilient locator in `loginLocator.ts` (role- or text-based) that matches both desktop and mobile, instead of the raw id.
- Scope: login feature; mobile viewport only.

<!-- Real entries start at KI-001. -->
