# TEST_CASES.md

Shared memory for QA Agent Flamme. Read by all four agents (Zenith, Flare,
Amber, Copper) as part of the startup protocol (AGENTS.md, section 7).

Registry of test cases: the single source of truth for testCaseId allocation
and test case status. It is an index, not the test case content.

## What goes here

Inclusion test: "Does this test case have (or need) a testCaseId?" If yes ->
it gets exactly one row here.

Boundaries (what does NOT go here):
- Full test case content (steps, gherkin, testData) -> amberToCopper payload.
- Product bugs -> GitLab.
- Harness quirks -> KNOWN_ISSUES.md.
- Why a rule exists -> DECISIONS.md.

Content is ASCII-only and in English (machine layer). Exception: `title`
mirrors the payload title and may be in Spanish, but stays ASCII (Rule 8).

## Rules

1. IDs: global counter TC-NNN. Next ID = max(testCaseId) + 1, computed only
   from the applied file. TC-000 is the example row; real IDs start at TC-001.
2. Reservation: Amber reserves an ID by proposing a `proposed` row. That row
   MUST be applied before another Amber session starts.
3. Append-only: rows are never deleted or reordered. `status` is the only
   mutable cell. A testCaseId is never reused.
4. Writes are a human approval gate (AGENTS.md, section 2): agents propose,
   Benito applies.
5. Atomicity: `-> automated` is applied in the merge commit of the spec;
   `automated -> deprecated` in the same commit that removes the spec's test().
6. Duplicates / contradictions: Amber reports them, never resolves them.
   Compared against active statuses (proposed, approved, automated, manual).
   A match against a `rejected` row is reported as info only.
7. title MUST match the amberToCopper payload title.
8. Cells: ASCII only, no `|` inside a cell, `check` max 100 characters.
9. Commit message per transition: TEST_CASES: TC-NNN <from> -> <to> (<reason>)

## Status lifecycle

```
proposed -> approved -> automated -> deprecated
proposed -> approved -> manual    -> deprecated
manual   -> approved
proposed -> rejected
approved -> deprecated
```

- proposed: ID reserved by Amber; awaiting human review.
- approved: accepted by Benito; ready for automation.
- automated: spec merged to main (Rule 5).
- manual: approved, executed by hand on purpose. Not terminal: can return to
  approved if it is later chosen for automation.
- rejected: never approved. Terminal.
- deprecated: was valid, now retired. Terminal.

Terminal statuses never transition. A retired case that is needed again gets
a new testCaseId (Rule 3).
To automate a manual case: manual -> approved -> automated (Copper only
acts on approved).

## Columns

| Column     | Values                                                    |
| ---------- | --------------------------------------------------------- |
| testCaseId | TC-NNN. Primary key.                                      |
| status     | proposed / approved / automated / manual / rejected / deprecated |
| feature    | camelCase feature token, same as the spec file name (login -> login.spec.ts). |
| title      | Same as the payload title (Rule 7).                       |
| check      | <condition> -> <expected>, max 100 characters.            |
| testType   | positive / negative / edge (amberToCopper schema).        |
| severity   | high / medium / low. Never empty; absent in payload -> medium. |
| issueId    | GitLab issue id, or `none` if there is no source bug.     |
| date       | YYYY-MM-DD. Reservation date. Immutable.                  |

## Registry

Ascending by testCaseId; new rows at the bottom. TC-000 is a permanent example
row (deprecated, so it never counts as active and is never automated).

| testCaseId | status     | feature | title                                  | check                                      | testType | severity | issueId  | date       |
| ---------- | ---------- | ------- | -------------------------------------- | ------------------------------------------ | -------- | -------- | -------- | ---------- |
| TC-000     | deprecated | login   | Login exitoso con credenciales validas | valid credentials -> redirect to dashboard | positive | high     | BIRD-042 | 2026-10-19 |
