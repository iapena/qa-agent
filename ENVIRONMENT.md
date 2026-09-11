# ENVIRONMENT.md

Shared memory for QA Agent Flamme. Read by all four agents (Zenith, Flare,
Amber, Copper) as part of the startup protocol (AGENTS.md, section 7).

Operational knowledge of the Birdple project: what an agent needs to know
about the environment it runs against. Not product bugs, not harness quirks.

## What goes here

Inclusion test: "Is this information about how the Birdple environment is or
behaves, that an agent needs in order to work, and that does not change per
task?" If yes -> it belongs here.

Boundaries (what does NOT go here):
- Real credentials -> .env (never committed).
- Product bugs -> GitLab.
- Harness quirks (flaky locators, timings as a test problem) -> KNOWN_ISSUES.md.

Content is ASCII-only and in English (machine layer). This file holds the DATA
(URLs, account refs, observed behavior). The RULES that govern that data are
referenced, not copied: "staging/local only, never prod" lives in AGENTS.md
(section 8); secrets handling in AGENTS.md (section 4).

## Environments

baseURL for each environment resolves from .env / playwright.config.ts, never
hardcoded in specs (copper.config.md, section 9).

- local: development environment used in the current phase. Birdple runs on
  localhost.
  - baseURL: env-driven. Value: (none documented yet)
- staging: the test environment agents run against.
  - baseURL: env-driven. Value: (none documented yet)
- prod: DO NOT RUN TESTS HERE. Production Birdple.
  - Never target this environment. Staging/local only (AGENTS.md, section 8).
  - baseURL: (none documented yet)

## Test accounts

Roles and references only. Real credentials live in .env, never here. A ref is
the NAME of an env var (e.g. STAGING_TEST_USER); the value resolves from .env.

- (none documented yet)

## Known environment behavior

Observed, non-per-task behavior of the environment (e.g. slowness at certain
hours, post-deploy downtime windows). If a recurring behavior turns into a
harness problem, it graduates to KNOWN_ISSUES.md.

- (none documented yet)

## Data setup requirements

Preconditions that must exist before a run (e.g. seed data X before test Y).

- (none documented yet)
