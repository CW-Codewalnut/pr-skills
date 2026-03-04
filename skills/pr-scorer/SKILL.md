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

| Criteria                 | Weight | What to look for                                                                                                    |
| ------------------------ | ------ | ------------------------------------------------------------------------------------------------------------------- |
| Readability & Naming     | 20%    | Are variables, functions, classes named clearly? Can someone unfamiliar with the codebase follow the logic?         |
| Code Organization        | 20%    | Are changes in the right files/modules? Is responsibility well-separated? Are new abstractions justified?           |
| Error Handling           | 15%    | Are failure cases handled? Are errors surfaced meaningfully (not swallowed)? Are edge cases accounted for?          |
| DRY & Reusability        | 15%    | Is there unnecessary duplication? Could shared utilities be extracted? Are existing helpers used where appropriate? |
| Complexity Management    | 15%    | Are functions small and focused? Is nesting reasonable? Are conditionals clear? Could anything be simplified?       |
| Consistency & Patterns   | 10%    | Do the changes follow the existing codebase conventions? Are patterns used consistently?                            |
| Comments & Documentation | 5%     | Are complex decisions explained? Are public APIs documented? Is there any dead/commented-out code?                  |

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

| Criteria                | Weight | What to look for                                                                                                               |
| ----------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------ |
| Performance             | 25%    | Are there unnecessary loops, redundant API calls, N+1 queries, unoptimized operations? Are large datasets handled efficiently? |
| Security                | 25%    | Is user input validated/sanitized? Are secrets handled properly? Are auth checks in place? Is data exposure minimized?         |
| Scalability             | 15%    | Will this work under load? Are there bottlenecks? Is resource usage bounded?                                                   |
| Backwards Compatibility | 15%    | Are existing APIs/contracts preserved? Are breaking changes flagged and justified? Are migrations provided?                    |
| Observability           | 10%    | Is there adequate logging for debugging? Are errors traceable? Are metrics/events tracked where useful?                        |
| Accessibility           | 10%    | (Frontend only) Are semantic elements used? Is keyboard navigation supported? Are ARIA attributes present where needed?        |

Adjust the weights dynamically based on what's relevant. If there are no frontend changes, redistribute the Accessibility weight across the other criteria. If there are no API changes, redistribute Backwards Compatibility weight. The total should always sum to 100%.

### 3. Test Coverage (Always Scored)

Evaluates the testing quality and coverage introduced or affected by the PR. This isn't about running a coverage tool — it's about analyzing the diff to assess whether the changes are well-tested.

**What to evaluate:**

| Criteria                    | Weight | What to look for                                                                                   |
| --------------------------- | ------ | -------------------------------------------------------------------------------------------------- |
| Test Presence               | 25%    | Are there test files in the diff? Do new features/fixes have corresponding tests?                  |
| Happy Path Coverage         | 20%    | Are the primary use cases tested? Do tests verify the main behavior introduced by the changes?     |
| Edge Case Coverage          | 20%    | Are boundary conditions tested? Null/empty inputs? Error states? Race conditions where relevant?   |
| Assertion Quality           | 15%    | Are assertions specific and meaningful? Do they check actual behavior, not just "no error thrown"? |
| Test Structure              | 10%    | Are tests organized logically? Are test names descriptive? Is setup/teardown clean?                |
| Integration vs Unit Balance | 10%    | Is there an appropriate mix? Are unit tests used for logic and integration tests for workflows?    |

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

### Step 2: Get the latest commit and full diff

Fetch the diff between the current branch and the base branch, along with the latest commit info:

```bash
git log -1 --format="%H %s"
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

Before scoring anything, take time to deeply understand the changes. Scoring without understanding leads to generic, useless feedback.

Think through:

- **What** changed? (files, functions, components, APIs)
- **Why** might it have changed? (bug fix, new feature, refactor, performance)
- **How** do the changes work together? (trace the logic across files)
- **What's the scope?** (small targeted fix or broad refactor?)
- **What's the tech stack?** (language, framework, testing libraries)
- **Are there breaking changes?** (API changes, schema migrations, removed exports)
- **What files are test files vs production code?** (identify by path conventions like `__tests__/`, `*.test.*`, `*.spec.*`, `test/`, etc.)

Group related changes into logical units. Understanding the intent behind the changes is critical for fair scoring — a deliberate tradeoff should be scored differently from an oversight.

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

**How to score fairly:**

- Score what's in the diff, not what's missing from the entire codebase. If the existing codebase has no tests and this PR also adds none, that's still a low test score — but note the context.
- Give credit for meaningful improvements. A PR that adds error handling to a previously unhandled path deserves recognition even if it doesn't handle every edge case.
- Be proportional to scope. A 5-line bug fix shouldn't be penalized for not adding comprehensive test suites — but it should have a targeted test for the fix.
- Consider intent. A refactoring PR that doesn't add features shouldn't lose Test Coverage points for not adding new tests (as long as existing tests still pass/are updated).
- Don't double-penalize. If a function has poor naming AND poor error handling, score each criterion independently — don't let one issue drag down the other.

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
- The top 2-3 most actionable findings — the things that would most improve the score
- Let the user know that the full detailed report is in `scores.md`

Keep the chat summary concise — the detailed report is in the file. The chat message is the "at a glance" view, like seeing your Lighthouse scores before diving into the full report.
