---
name: cli-contributor
description: >-
  Use with tdd and repository-technical-analysis when implementing changes in the CLI product
  repository. Adds repo-local TypeScript/JavaScript monorepo conventions, package-script-first
  merge request summaries against the default branch, layout anchors, testable design (dependency
  injection, no hidden globals), and optional Slack MCP context when configured. Agent- and
  IDE-agnostic except Slack supplement (Cursor Slack plugin).
---

# CLI Product Contributor

Use this skill as a repo-specific overlay layered on top of `tdd` and `repository-technical-analysis` when working in the **CLI product** source repository (terminal or agent-facing CLI codebase).

## When to Use

Use this skill when the user is developing or fixing work inside that CLI repository and needs:

- package-manager and script discipline aligned with this repo
- repo-local lint, typecheck, and test commands
- MR description guidance that matches team templates
- implementation notes that stay compatible with CI and release expectations

## When Not to Use

Do not use this skill when:

- the task is outside the CLI product repository
- generic `tdd` or `repository-technical-analysis` is enough and no repo-local rules apply
- the task is pure transport (GitLab fetch only, Jira only, CircleCI fetch only) without local code changes

## First Read

- Read `AGENTS.md` at the repository root before editing.
- Read `package.json`, `pnpm-workspace.yaml` or `turbo.json` when present to choose commands; do not guess script names that are not declared.
- Load `tdd` for test-first flow and `repository-technical-analysis` for investigation framing. Literal search: synced **`LITERAL-CODE-SEARCH.md`**. Use `circleci` when pipeline or job status from CircleCI is needed for fixes or MR notes. Keep this skill for this CLI repo’s local rules only.
- When ticket, MR, or design context is missing from artifacts and Jira/GitHub, see **Slack context** below if the **Cursor Slack plugin** (`https://mcp.slack.com/mcp`) is connected.
- If the user provides `$ARTIFACTS/<meaningful_id>/task_<issue>.md`, `review_mr_<MR>.md`, or `analysis_mr_<MR>.md` (or legacy root-level equivalents), read it first and reuse repository context, links, assumptions, and open questions.

## Design principles

Testability is a **primary objective** (see **Contributor design principles** in repo `AGENTS.md`):

- **Inject dependencies** — pass collaborators via constructor args, function parameters, or explicit factory/context objects; avoid `import { singleton }` from deep inside logic.
- **Avoid globals** — no module-level mutable caches, registries, or config reads at import time; wire dependencies at CLI entrypoints, command handlers, or test setup.
- **Side effects at the edge** — keep parsing, orchestration, and I/O in thin layers; core logic should accept interfaces/types you can fake in Jest/Vitest (or the repo’s test runner).
- **Tests first-class** — prefer injecting test doubles over sprawling `vi.mock`/`jest.mock` trees; new behavior should be coverable with package-scoped tests.
- For new behavior or regressions, pair with **`tdd`**: red-green-refactor on public interfaces using injected fakes.

## Repo Workflow

- Prefer **pnpm** when `packageManager` or lockfiles indicate pnpm; otherwise follow the repo’s documented package manager.
- Discover validation from **`package.json` `scripts`** (for example `lint`, `typecheck`, `test`, `build`). Run the narrowest script that still validates the change.
- For monorepos, use **Turbo** (`turbo run …`) when `turbo.json` exists and the task spans packages; scope with filters instead of running unrelated workspaces.
- Keep changes scoped; avoid drive-by refactors across packages unless the task requires it.
- When summarizing merge requests, compare the current branch to the remote default branch (usually `origin/main`) unless the user names another base.
- **Before finishing**, shrink the diff: review the full change set (`git diff`), drop out-of-scope edits, debug noise, and redundant abstractions, and minimize the patch without changing behavior; respect monorepo scope—do not shrink by stripping tests or cross-package fixes the task required. Re-run the same validation if production code changes materially.

## Slack context

Optional supplement when **Slack MCP tools** are available (Cursor **Settings → MCP** or marketplace Slack plugin connected). Use **after** local code, artifacts, Jira, and GitHub — not instead of them. Pair with **`cli-technical-analysis`** for deeper investigation-style Slack search.

**Skip** when the task is fully specified in code and artifacts, or when MCP is missing or auth fails — state which and continue.

**Contributor use cases:**

- reviewer or teammate guidance in threads (approach, naming, rollout constraints)
- design decisions or “ship / hold” discussion not captured in Jira or the MR
- incident or hotfix threads that justify scope, risk, or follow-up in the MR summary
- repro or customer-impact notes from team channels when shaping tests or fix scope

**Tool ladder** (same Slack MCP server as `cli-technical-analysis`):

1. **`slack_search_users`** — resolve email or display name to `USER_ID` when the task names a person.
2. **`slack_search_public_and_private`** — search by Jira key (`CLI-####`), MR/PR ref, branch name, or `from:<@USER_ID>` (`limit` 10–20, `sort` `timestamp`, `sort_dir` `desc`, `include_context` `true`).
3. **`slack_read_thread`** — read threads behind promising hits for review feedback or rollout notes.

Prefer search over channel history. Redact secrets; cite channel + date + short snippet in MR notes or artifacts — do not paste tokens or customer data.

## Validation

- After substantive edits, run **lint** and **typecheck** scripts when the repo defines them.
- Run **tests** relevant to touched packages before finishing; use `tdd` for regression-first fixes.
- When acceptance tests fail, suggest `TEST_SNYK_IGNORE_LIST` as the selective CI-unblock option documented in `CONTRIBUTING.md`, especially for blocking specs outside CLI scope; do not use it to hide a CLI-owned regression.
- If CI duplicates a local script name, prefer the same script locally to match CI behavior.
- Record noisy or flaky commands once under durable sections in artifacts (see `../ARTIFACTS.md` patterns) when you find a faster reliable alternative.
- After validation passes, complete the **Before finishing** diff-shrink step in **Repo Workflow** above.

## Merge Request Summaries

When asked to prepare an MR description:

1. Inspect committed changes on the current branch against the agreed base (`origin/main` unless stated).
2. If `.gitlab/merge_request_templates/` exists, start from the appropriate template and fill every section.
3. Summarize what changed and why in engineer-oriented language; link issues or tickets when known.
4. Call out risk, rollout notes, and follow-up work only when grounded in the diff or discussion — including **Slack context** when MCP search found relevant threads and the takeaway is confirmed against the change set.

## Artifact-Aware Behavior

When a local workflow artifact exists:

- prefer `$ARTIFACTS/<meaningful_id>/` for new artifacts per `ARTIFACTS.md`; extend existing root-level files in place
- read it first for durable context
- refresh conclusions against current code and CI signals before reuse
- preserve shared schema sections from `ARTIFACTS.md` when updating the same file

## Outputs / Artifacts

This skill typically adds:

- repo-local command choices and validation paths
- MR description structure for this repository

## Companion Skills

Layer with:

- `tdd` for red-green-refactor implementation
- `diagnose` for hard failures before encoding regressions
- `repository-technical-analysis` for broader codebase reasoning
- `cli-technical-analysis` for investigation-style Slack search and repro-oriented queries
- `circleci` for CircleCI pipeline and job context
- `git` for branch and diff inspection
- **Cursor Slack plugin** (optional) — **Slack context** section; MCP last, after local and bundled transport per `AGENTS.md`

## Safety Notes

- Do not invent package scripts; always confirm in `package.json` or workspace docs.
- Stop and ask when auth, tokens, or signing are required but missing.
- Slack MCP requires workspace admin approval and OAuth; if tools are missing or auth fails, proceed without Slack — do not ask the user to paste full Slack exports unless they offer.
