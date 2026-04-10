# Agent Skills

A collection of PR workflow skills for AI coding agents. Skills are packaged instructions that extend agent capabilities.

Skills follow the [Agent Skills](https://skills.sh) format.

## Available Skills

### pr-document-writer

Generate PR titles (conventional commit format) and descriptions by analyzing the current branch's diff against the base branch. Automatically detects project PR templates or uses a sensible default.

Use when:

- "Write a PR"
- "Describe my changes"
- "Open a pull request"
- "Generate PR description"

Features:

- Conventional commit title generation (`feat`, `fix`, `refactor`, etc.)
- Auto-detects `.github/pull_request_template.md` and other common template locations
- Can create or update PRs via GitHub tools when available

### pr-scorer

Score a PR across three quality dimensions — like Core Web Vitals, but for PRs. Produces a detailed `scores.md` report with actionable feedback.

Use when:

- "Score my PR"
- "Rate my changes"
- "How good is this PR?"
- "Check PR readiness"

Scoring categories:

- **Code Quality** (always scored) — readability, organization, error handling, DRY, complexity
- **Non-Functional Requirements** (conditional) — performance, security, scalability, accessibility
- **Test Coverage** (always scored) — test presence, edge cases, assertion quality

### kt-doc

Generate comprehensive Knowledge Transfer documentation for any frontend module. Dispatches three parallel exploration agents to analyze UI flows, data/API layers, and cross-cutting concerns, then synthesizes findings into a 12-section KT document.

Use when:

- "Document this module"
- "Create KT"
- "Knowledge transfer"
- "Onboarding doc"
- "What does this module do"

Features:

- Auto-detects framework (Next.js, Nuxt, Angular, Svelte, React, Vue, etc.)
- Produces non-technical documentation readable by product owners, new developers, and coding agents
- Covers user flows, business rules, API dependencies, cross-module interactions, analytics, and tech debt

## Installation

Refer to the [skills.sh docs](https://skills.sh/docs) for setup instructions.

```
npx skills add CW-Codewalnut/agent-skills
```

## Usage

Skills are automatically available once installed. The agent will use them when relevant tasks are detected.

```
Write a PR description for my changes
```

```
Score my PR
```

```
/kt-doc src/features/checkout
```

## Skill Structure

Each skill contains:

- `SKILL.md` — Instructions for the agent
- `assets/` — Supporting templates and references

## License

MIT
