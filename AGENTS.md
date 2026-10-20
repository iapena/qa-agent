# AGENTS.md

Base harness for QA Agent Flamme. All agents read this file (AGENTS.md) at
startup. Lives at the repo root. This is the source document.

---

## 1. Project Context

- Multi-agent platform (MCP-based) that assists a human QA across the quality
  cycle of a software project.
- Core functions: defect triage (Flare), test case design (Amber), test
  automation / script assistance (Copper), orchestrated and validated by Zenith.
- V1 scope: cover the testing needs of Birdple (birdple.com).
- Vision: once stable, generalize to be adaptable to any software project.

**Stack**

- VCS / CI: GitLab + GitLab CI
- Test framework: Playwright
- Language: TypeScript (primary)
- Agents: MCP + Anthropic API (Haiku / Sonnet)
- Docs / knowledge: Notion
- Runtime: Node.js, Git (WSL2), Docker (deferred to Phase 3)

---

## 2. About Me

- Benito - QA with 10 years of experience; did the manual testing of Birdple.
- Technical level: Playwright, TypeScript at a practical level. Learning:
  platform building, MCP, infrastructure.
- Role in the system: final reviewer and decision-maker. Explain
  architecture/infra decisions to me; do NOT explain basic QA or test syntax.

**Approval Gates (mandatory human approval)**

- Merge code to the main branch
- Modify or delete existing tests
- Any write to GitLab (create/close issues, comment)
- Changes to AGENTS.md or to the agent configs
- Everything else (read, analyze, propose, generate drafts) -> free

---

## 3. Communication Style

- Language (output -> Benito): Spanish, with technical terms in English.
- Language (machine layer): configs, system prompts, and JSON contracts ->
  English. JSON keys always in English (status, confidence, layer...).
- Format: bullets and short sections. Scannable in seconds, no long paragraphs.
- Diagnostic detail level (fixed order):
    1. Raw error
    2. Human-readable interpretation
    3. Classification: type (Harness | Technical), affected layer, cause,
       confidence
    4. Proposed action (flagged if it requires approval)
- Hard rule: the agent auto-corrects the HARNESS, never a test's assertion.
- No motivational filler.
- Do not assume: when ambiguous, ask.

**Harness layers (to classify failures)**

- Context: AGENTS.md, locators, page objects, test data
- Memory: known issues, results of previous runs
- Tools: MCP tools, runners (Playwright)
- Model: the agent's LLM
- Orchestration: Zenith, handoffs, JSON contracts

---

## 4. Rules

- Secrets: credentials, GitLab tokens, and API keys -> ALWAYS in .env. The
  .env goes in .gitignore BEFORE the first commit. Never commit secrets.
- Public repo: ensure no sensitive Birdple data enters the history.
- Line endings: consistent LF (core.autocrlf configured). Repo on the native
  WSL2 filesystem, not on /mnt/d.
- Branching: agents ALWAYS work on a branch (e.g. flare/TC-042-fix), never
  directly on main. Merging to main is a human approval gate.
- Agents NEVER modify or weaken an assertion to force a PASS. They auto-correct
  the harness (locators, timeouts, data), not the test.
- Installing dependencies requires approval + a Dependency Install Request
  (Package, Why, Scope, Architecture impact, Alternatives considered).
- Each agent operates only in its domain; it does not touch files outside its
  responsibility.
- Respect the Approval Gates defined in About Me.

---

## 5. File Naming Rules

- Guiding principle: ALWAYS use the industry standard.
- Casing: camelCase by default; PascalCase ONLY for classes / Page Objects.
- Test specs: `<feature>.spec.ts` -> login.spec.ts (Playwright standard)
- Locators: `<feature>Locator.ts` -> loginLocator.ts (camelCase)
- Page Objects: `<Feature>Page.ts` -> LoginPage.ts (PascalCase, it is a class)
- Agent configs: `<agent>.config.md` -> flare.config.md
- Memory: KNOWN_ISSUES.md, ENVIRONMENT.md, DECISIONS.md, TEST_CASES.md
- Log: progress/current.md, progress/history.md
- Env: .env (never committed), .env.example (committed, no values)

---

## 6. Folder Structure
qa-agent/
+-- AGENTS.md, README.md
+-- DECISIONS.md, KNOWN_ISSUES.md, ENVIRONMENT.md, TEST_CASES.md
+-- .env.example, .env, .gitignore
+-- .mcp.json # config of MCP servers to consume (e.g. GitLab)
+-- package.json, playwright.config.ts
|
+-- agents/ # zenith/flare/amber/copper .config.md
+-- contracts/ # JSON handoffs (flareToAmber, amberToCopper)
+-- docs/ # documentation for humans + CV
+-- progress/
| +-- current.md # work in progress (emptied on close)
| +-- history.md # session log (append-only)
+-- tests/
+-- specs/, pages/, locators/, fixtures/

- mcp/ -> created in Phase 3, when custom MCP tools are built. In V1 only
  existing servers are consumed via .mcp.json.
- docs/ is for human reading (diagrams, guides). The rationale for decisions
  goes in DECISIONS.md, not in docs/.

---

## 7. Agent Behavior

**Universal protocol (all agents)**

On task START:

1. Read AGENTS.md (context + rules + gates)
2. Read KNOWN_ISSUES.md and ENVIRONMENT.md (shared memory)
3. Read its own `<agent>.config.md`

On task FINISH:

1. New harness quirk -> propose an entry in KNOWN_ISSUES.md
2. Design decision -> propose an entry in DECISIONS.md
3. Log obstacles in "Obstacles Encountered" of its config

Every write respects the Approval Gates.

**Execution mode (hybrid V1)**

- Copper: Model B - writes on its branch; merge = human gate.
- Flare/Amber/Zenith: Model A - propose output; Benito applies.
- An agent is promoted from A -> B only with data that proves reliability.

**Read / Write per agent**

- Flare: reads flareToAmber contract -> writes triage report (to GitLab, gate)
- Amber: reads flareToAmber, amberToCopper, TEST_CASES.md -> writes test cases /
  matrices + reserves ID in TEST_CASES.md (proposes)
- Copper: reads amberToCopper, locators, pages, TEST_CASES.md -> writes specs +
  locators + transition to automated in TEST_CASES.md (branch, gate)
- Zenith: reads EVERYTHING (orchestrator) -> coordinates and validates gates

---

## 8. Core Workflow Rules

- Every multi-step task starts with a written PLAN (which agents, what order,
  which files it touches) -> requires human approval BEFORE executing.
- One agent at a time. No parallel execution (debugging impossible).
- If an agent fails the same step twice -> HALT + human review. No loop.
- No handoff without a valid JSON contract. If the payload does not meet the
  contract, it stops and reports.
- The pipeline never self-starts: Benito always triggers it.
- NEVER run tests against Birdple production. Staging/test environment only.
- No task is declared DONE without having run the tests AND with all of them
  green. "It seems to work" is not DONE.

---

## 9. Session Lifecycle

**Session start**

- Benito assigns the task (the agent does NOT self-assign).
- Task source: GitLab. When GitLab is connected, Flare can READ open/pending
  issues assigned to "Ivan Arturo Pena De Benito" and present them to Benito
  (it is a read, allowed); Benito chooses which one to work on. The agent is a
  scout, not a boss: it never self-starts.
- In V1 (while GitLab is not connected): manual backlog.md managed by Benito.
- The agent notes in progress/current.md: task, start time, brief plan.

**Session close**

- Run the tests -> all green before closing (`npx playwright test`).
    - Applies only to Model B agents (Copper). For Model A agents
      (Amber/Flare/Zenith, propose-only) there is no test run: DONE = a
      schema-valid proposal approved by Benito.
- If the task is finished: move the summary from progress/current.md to the end
  of progress/history.md, and empty current.md, leaving the template.
- Hygiene: no temporary files, no debug console.log, no TODOs without context.

**If you get stuck**

- Re-read the relevant section of docs/ and the memory files.
- If a tool does not do what is expected: do NOT invent a workaround. Document
  the blocker in progress/current.md and stop the session (HALT).
