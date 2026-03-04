---
name: pr-scorer
description: Score a Pull Request on three core metrics — Code Quality, Non-Functional Requirements, and Test Coverage — and generate a detailed scores.md report. Use this skill whenever the user wants to score a PR, evaluate PR quality, review PR health, rate their changes, get a PR scorecard, check PR readiness, assess code quality of a branch, or asks anything related to PR scoring, PR metrics, PR grading, or code review scores — even if they just say "score my PR", "rate my changes", "how good is this PR", or "evaluate my branch".
---

# PR Scorer

Score a pull request across three core quality dimensions — like Core Web Vitals, but for PRs. Each category is scored out of 100 with detailed justification, producing a rich `scores.md` report the author can act on immediately.

## Scoring Categories

### 1. Code Quality (Always Scored)

Evaluates the craftsmanship and maintainability of the code changes. This is the baseline health check that applies to every PR regardless of what it does.

**What to evaluate:**

| Criteria                 | Weight | What to look for                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------------ | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Readability & Naming     | 20%    | Are variables, functions, classes named clearly and descriptively? Can someone unfamiliar with the codebase follow the logic without jumping between files? Are abbreviations avoided? Do boolean variables read as questions (`isReady`, `hasPermission`)? Are function names verbs that describe what they do? Are there any misleading names — variables that suggest one type but hold another?                                               |
| Code Organization        | 20%    | Are changes in the right files/modules? Is responsibility well-separated — does each file/function do one thing? Are new abstractions justified and not premature? Are imports organized and minimal (no unused imports)? Is there clear separation between business logic, data access, and presentation? Are new files placed in the correct directory following existing project conventions?                                                  |
| Error Handling           | 15%    | Are ALL failure cases handled — not just the obvious ones? Are errors surfaced meaningfully with context (not swallowed silently or re-thrown without added info)? Are edge cases like null, undefined, empty arrays, empty strings, 0, and negative numbers accounted for? Are async operations wrapped in proper error handling? Do error messages help the developer debug the issue? Are there any catch blocks that silently swallow errors? |
| DRY & Reusability        | 15%    | Is there any duplicated logic — even across different files? Could repeated patterns be extracted into shared utilities? Are existing helpers and utilities in the codebase being used where appropriate instead of reinventing them? Are magic numbers and hardcoded strings extracted into named constants? Could any new code be written as a reusable function or module?                                                                     |
| Complexity Management    | 15%    | Are functions small and focused (ideally under 30 lines)? Is nesting kept to 3 levels or fewer? Are conditionals clear and not deeply nested — could guard clauses or early returns simplify them? Are there any god functions that do too many things? Could complex conditionals be extracted into well-named helper functions? Are ternaries readable or should they be if/else? Is cyclomatic complexity reasonable?                          |
| Consistency & Patterns   | 10%    | Do the changes follow the existing codebase conventions for formatting, naming, file structure, and patterns? Are similar operations done the same way throughout the diff? Are there places where the new code contradicts patterns established elsewhere in the same PR? Are language/framework idioms followed?                                                                                                                                |
| Comments & Documentation | 5%     | Are complex decisions, non-obvious logic, and workarounds explained with comments? Are public APIs documented with parameter types, return values, and usage examples? Is there any dead or commented-out code that should be removed? Are TODO/FIXME/HACK comments present without linked issues or timelines? Are JSDoc/docstrings present for exported functions?                                                                              |

### 2. Non-Functional Requirements (Conditionally Scored)

This category applies only when the changes touch areas where NFRs are relevant. Not every PR has NFR implications — a typo fix in a README doesn't need a performance review. The skill should detect when NFRs matter and score accordingly.

**When NFRs apply** — score this category when the diff includes any of:

- API endpoint changes or additions
- Database queries or schema changes
- Authentication/authorization logic
- File I/O or network calls
- Frontend rendering logic or component changes
- Infrastructure or deployment configuration
- Dependency additions or upgrades
- Data processing or transformation pipelines
- Caching or state management changes

**When NFRs don't apply** — skip this category (mark as N/A) for:

- Documentation-only changes
- Pure test additions with no production code
- Code style/formatting-only changes
- Comment or typo fixes
- Renaming/refactoring with no behavioral change

**What to evaluate (when applicable):**

| Criteria                | Weight | What to look for                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ----------------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Performance             | 25%    | Are there unnecessary loops, redundant API calls, or N+1 queries? Are database queries indexed and selective (no `SELECT *` without reason)? Are large datasets paginated or streamed instead of loaded into memory? Are there synchronous operations that could be parallelized? Are expensive computations cached or memoized where appropriate? Are there unnecessary re-renders in frontend components? Are objects or arrays being copied/spread unnecessarily in hot paths?                                                              |
| Security                | 25%    | Is ALL user input validated and sanitized at the boundary — not just at the point of use? Are secrets, tokens, and credentials kept out of code and config files committed to the repo? Are auth checks present on every endpoint/route that needs them? Is data exposure minimized — are API responses returning only the fields the client needs? Are SQL queries parameterized? Is file upload validated for type and size? Are CORS, CSP, and other security headers considered? Are there any hardcoded credentials, API keys, or tokens? |
| Scalability             | 15%    | Will this work when data volume grows 10x–100x? Are there unbounded queries, unbounded loops, or unbounded in-memory collections? Is resource usage (memory, connections, file handles) bounded and cleaned up? Are there potential race conditions under concurrent access? Are database transactions scoped tightly? Is connection pooling used where applicable?                                                                                                                                                                            |
| Backwards Compatibility | 15%    | Are existing APIs, contracts, and interfaces preserved? Are breaking changes to public APIs clearly documented and justified? Are database schema changes backwards-compatible (additive only) or do they include proper migrations? Are deprecated features marked and given a migration path? Will existing clients/consumers break?                                                                                                                                                                                                         |
| Observability           | 10%    | Is there structured logging at appropriate levels (not just console.log)? Are errors logged with enough context to trace and reproduce? Are key business events tracked with metrics or events? Can an on-call engineer diagnose a failure in production from the logs this code produces? Are request IDs or correlation IDs propagated?                                                                                                                                                                                                      |
| Accessibility           | 10%    | (Frontend only) Are semantic HTML elements used instead of generic divs/spans? Is keyboard navigation fully supported (tab order, focus management, escape to close)? Are ARIA attributes present and correct where semantic HTML isn't sufficient? Do images have alt text? Is color contrast sufficient? Are form inputs labeled? Do interactive elements have visible focus indicators?                                                                                                                                                     |

Adjust the weights dynamically based on what's relevant. If there are no frontend changes, redistribute the Accessibility weight across the other criteria. If there are no API changes, redistribute Backwards Compatibility weight. The total should always sum to 100%.

### 3. Test Coverage (Always Scored)

Evaluates the testing quality and coverage introduced or affected by the PR. This isn't about running a coverage tool — it's about analyzing the diff to assess whether the changes are well-tested.

**What to evaluate:**

| Criteria                    | Weight | What to look for                                                                                                                                                                                                                                                                                                                                                                        |
| --------------------------- | ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Test Presence               | 25%    | Are there test files in the diff? Does every new feature, bug fix, or behavioral change have a corresponding test? Are new exported functions/endpoints/public methods covered? If no tests are present and production code was changed, this criterion should score very low.                                                                                                          |
| Happy Path Coverage         | 20%    | Are the primary use cases tested end-to-end? Do tests verify the actual output/behavior, not just that the function runs? Are all major code paths through new logic exercised? Do tests cover the typical inputs a real user would provide?                                                                                                                                            |
| Edge Case Coverage          | 20%    | Are boundary conditions tested — empty inputs, null/undefined, zero, negative numbers, maximum values, empty arrays/objects? Are error states tested — what happens when dependencies fail, when input is malformed, when permissions are denied? Are concurrent/race conditions considered where relevant? Are off-by-one scenarios covered?                                           |
| Assertion Quality           | 15%    | Are assertions specific — checking exact values, specific error types, precise state changes? Or are they weak — just checking truthiness, checking for no error, checking array length without verifying contents? Do assertions validate behavior and outcome, not implementation details? Are error assertions checking the right error type/message, not just "it threw something"? |
| Test Structure              | 10%    | Are tests organized with clear describe/context blocks? Are test names descriptive enough to understand the scenario without reading the test body? Is setup/teardown clean and using proper beforeEach/afterEach hooks? Are test utilities and fixtures well-organized? Is there unnecessary test duplication that could be parameterized?                                             |
| Integration vs Unit Balance | 10%    | Is there an appropriate mix for the type of changes? Are pure logic functions unit-tested? Are workflows, API routes, and multi-component interactions integration-tested? Are external dependencies mocked at the right boundary — not too deep (testing mocks) and not too shallow (missing integration issues)?                                                                      |

## Workflow

### Step 1: Identify the base branch

Determine the base branch that the current branch will be merged into.

Run these git commands to gather context:

```bash
git branch --show-current
git remote show origin | grep "HEAD branch"
git log --oneline -5
```

Use `git remote show origin` to find the default branch (usually `main` or `master`). If the repo has a common base like `develop` or `staging`, infer from the branch naming convention or recent merge targets.

If the base branch cannot be confidently determined (e.g., ambiguous remote setup, multiple candidates), ask the user which branch they intend to merge into. Don't guess — getting the base branch wrong means the diff will be wrong, which means the entire scoring will be wrong.

### Step 2: Get the full diff

Fetch the diff between the current branch and the base branch:

```bash
git diff <base-branch>...HEAD --stat
git diff <base-branch>...HEAD
```

Use the three-dot diff (`...`) because it shows only the changes introduced on the current branch since it diverged from the base — this is what a PR reviewer will actually see.

If the diff is extremely large (thousands of lines), use `--stat` first to understand the scope, then read the most important changed files selectively. For very large diffs, focus on:

- New files (they tell the story of what was added)
- Modified core files (business logic, not config/lock files)
- Deleted files (what was removed and why)

Skip noisy files like lock files, auto-generated code, or minified assets — they add nothing to the scoring.

### Step 3: Analyze the diff thoroughly

Before scoring anything, deeply understand the changes. Scoring without understanding leads to generic, useless feedback.

You **must** determine all of the following before proceeding to scoring:

- **What** changed? (files, functions, components, APIs)
- **Why** might it have changed? (bug fix, new feature, refactor, performance)
- **How** do the changes work together? (trace the logic across files)
- **What's the scope?** (small targeted fix or broad refactor?)
- **What's the tech stack?** (language, framework, testing libraries)
- **Are there breaking changes?** (API changes, schema migrations, removed exports)
- **What files are test files vs production code?** (identify by path conventions like `__tests__/`, `*.test.*`, `*.spec.*`, `test/`, etc.)
- **What new public API surface was added?** (exported functions, new endpoints, public methods)
- **What error paths exist in the new code?** (every conditional, every try/catch, every early return)
- **What inputs does the new code accept?** (user input, API payloads, query params, file uploads — and how are they validated?)
- **What dependencies were added or changed?** (new packages, version bumps, removed deps)
- **Are there any leftover artifacts?** (debug logs, commented-out code, TODO/FIXME without linked issues, temporary workarounds)

Group related changes into logical units. Understanding the intent behind the changes is critical for fair scoring — a deliberate tradeoff that is documented with a comment should be scored differently from an unexplained shortcut.

### Step 4: Determine NFR applicability

Before scoring, decide whether the Non-Functional Requirements category applies to this PR. Review the diff through the lens of the NFR applicability rules defined above.

If NFRs apply:

- Note which specific NFR criteria are relevant (not all sub-criteria apply to every PR)
- Adjust weights to redistribute from non-applicable sub-criteria
- All three categories will be scored and the overall score is the weighted average of all three

If NFRs don't apply:

- Mark NFRs as "N/A" in the report
- The overall score becomes the weighted average of Code Quality and Test Coverage only
- Briefly explain why NFRs were not scored (e.g., "This PR contains only documentation changes — NFR scoring is not applicable")

### Step 5: Score each category

For each applicable category, go through every criterion in the evaluation table and assign a score. Be specific and evidence-based — every score needs justification rooted in the actual diff, not generic advice.

**Scoring guidelines:**

| Range  | Label             | Meaning                                                                        |
| ------ | ----------------- | ------------------------------------------------------------------------------ |
| 90–100 | Excellent         | Exemplary code. Minor or no issues. Ready to merge as-is.                      |
| 75–89  | Good              | Solid work with minor improvements possible. Merge-ready with optional polish. |
| 50–74  | Needs Improvement | Noticeable issues that should be addressed before merging.                     |
| 25–49  | Poor              | Significant problems. Requires substantial rework.                             |
| 0–24   | Critical          | Fundamental issues. Should not be merged in current state.                     |

**How to score:**

- **Score what's in the diff.** The diff is the only evidence. Do not give credit for code that exists elsewhere in the codebase unless it's part of the diff.
- **Be proportional to scope.** A 5-line bug fix doesn't need a comprehensive test suite — but it should have a targeted test for the fix.
- **Consider intent.** A refactoring PR that doesn't add features shouldn't lose Test Coverage points for not adding new tests (as long as existing tests still pass/are updated).
- **Don't double-penalize.** If a function has poor naming AND poor error handling, score each criterion independently — don't let one issue drag down the other.
- **Every score must have justification** rooted in the actual diff — cite specific files, functions, and line ranges. No score should appear without an explanation.

### Step 6: Calculate the overall score

Compute the overall PR score as a weighted average:

**When all three categories apply:**

- Code Quality: 40%
- Non-Functional Requirements: 30%
- Test Coverage: 30%

**When NFRs are N/A:**

- Code Quality: 55%
- Test Coverage: 45%

Round the final score to the nearest integer.

### Step 7: Generate the scores.md report

Read the template at `assets/scores-template.md` (relative to this skill's directory) and fill it in with the scoring results.

Write the completed report to `scores.md` in the repository root directory.

### Step 8: Present a summary

After writing `scores.md`, present a brief summary in the chat:

- The three category scores (or two if NFR is N/A) as a quick visual
- The overall score with its label
- The top 2-3 most critical findings — the specific issues that **must** be fixed
- Let the user know that the full detailed report is in `scores.md`

Keep the chat summary concise — the detailed report is in the file. The chat message is the "at a glance" view, like seeing your Lighthouse scores before diving into the full report.

**Do not include the commit hash or commit message anywhere in the summary or the report.**
