# DECISIONS.md
 
Shared memory for QA Agent Flamme. Read by all four agents (Zenith, Flare,
Amber, Copper) as part of the startup protocol (AGENTS.md, section 7).
 
Lightweight ADR (Architecture Decision Record) log: one record per locked-in
design decision. Records capture the "why", not just the rule, so a future
agent or human does not relitigate a settled choice.
 
Entries are PROPOSED by agents; writing to this file is a human approval gate
(AGENTS.md, section 2). Records are append-only and immutable: a decision is
never rewritten or deleted. When a decision changes, mark the old record
Superseded and add a new one.
 
## Status values
 
- Accepted: the decision is in force (default).
- Superseded by ADR-xxx: replaced by a later record; kept for history.
- Deprecated: no longer applies and not replaced.

## Entry format
 
```
## ADR-NNN: <title>
- Status: Accepted
- Date: YYYY-MM-DD
- Context: <the problem or force that required a decision>
- Decision: <the rule, in the imperative>
- Consequences: <agents / files / contracts affected>
- Source: <canonical origin; reference, do not copy>
- Alternatives considered: <optional; what was weighed and why rejected>
```
 
Ordering: append-only, oldest first. Content is ASCII-only and in English
(machine layer). Reference canonical sources; do not duplicate rules that
already live in AGENTS.md or a config.
 
## Index
 
| ID      | Title                                              | Status   |
| ------- | -------------------------------------------------- | -------- |
| ADR-001 | gherkin is authoritative over steps                | Accepted |
| ADR-002 | Repo files are plain ASCII; section sign chat-only | Accepted |
 
---
 
## ADR-001: gherkin is authoritative over steps
- Status: Accepted
- Date: 2026-09-07
- Context: An amberToCopper payload carries both gherkin (given/when/then) and steps. The two can diverge, and a generated spec needs a single authoritative structure.
- Decision: When gherkin and steps differ, gherkin is the source of truth for the test structure. steps is human-readable support only.
- Consequences: Affects Copper when mapping the contract to a Playwright spec (given -> setup, when -> action, then -> assertion). No effect on how Amber authors the payload.
- Source: copper.config.md, Inputs / Outputs section (consumed field: gherkin authoritative, steps readable support).

## ADR-002: Repo files are plain ASCII; section sign chat-only
- Status: Accepted
- Date: 2026-09-07
- Context: Non-ASCII glyphs (the section sign, em-dashes, Unicode arrows) can produce mojibake and encoding bugs across editors, terminals, and CI, especially on the WSL2 native filesystem.
- Decision: All committed repo files use plain ASCII. The section sign is used only in chat, never in committed files. Applies to all four agent configs and every committed markdown file.
- Consequences: Reviewers keep every proposed entry ASCII-only. Applies to the four configs, memory files, and contracts.
- Source: project encoding convention (Flamme conventions).
