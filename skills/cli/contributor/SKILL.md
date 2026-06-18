---
name: cli-contributor
description: >-
  Use with tdd and repository-technical-analysis when implementing changes in the CLI product
  repository. Adds repo-local TypeScript/JavaScript monorepo conventions, package-script-first
  validation, merge request summaries against the default branch, and layout anchors. Agent- and
  IDE-agnostic.
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
- If the user provides `$ARTIFACTS/<meaningful_id>/task_<issue>.md`, `review_mr_<MR>.md`, or `analysis_mr_<MR>.md` (or legacy root-level equivalents), read it first and reuse repository context, links, assumptions, and open questions.

## Repo Workflow

- Prefer **pnpm** when `packageManager` or lockfiles indicate pnpm; otherwise follow the repo’s documented package manager.
- Discover validation from **`package.json` `scripts`** (for example `lint`, `typecheck`, `test`, `build`). Run the narrowest script that still validates the change.
- For monorepos, use **Turbo** (`turbo run …`) when `turbo.json` exists and the task spans packages; scope with filters instead of running unrelated workspaces.
- Keep changes scoped; avoid drive-by refactors across packages unless the task requires it.
- When summarizing merge requests, compare the current branch to the remote default branch (usually `origin/main`) unless the user names another base.
- **Before finishing**, shrink the diff: review the full change set (`git diff`), drop out-of-scope edits, debug noise, and redundant abstractions, and minimize the patch without changing behavior; respect monorepo scope—do not shrink by stripping tests or cross-package fixes the task required. Re-run the same validation if production code changes materially.

## Validation

- After substantive edits, run **lint** and **typecheck** scripts when the repo defines them.
- Run **tests** relevant to touched packages before finishing; use `tdd` for regression-first fixes.
- If CI duplicates a local script name, prefer the same script locally to match CI behavior.
- Record noisy or flaky commands once under durable sections in artifacts (see `../ARTIFACTS.md` patterns) when you find a faster reliable alternative.
- After validation passes, complete the **Before finishing** diff-shrink step in **Repo Workflow** above.

## Merge Request Summaries

When asked to prepare an MR description:

1. Inspect committed changes on the current branch against the agreed base (`origin/main` unless stated).
2. If `.gitlab/merge_request_templates/` exists, start from the appropriate template and fill every section.
3. Summarize what changed and why in engineer-oriented language; link issues or tickets when known.
4. Call out risk, rollout notes, and follow-up work only when grounded in the diff or discussion.

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
- `circleci` for CircleCI pipeline and job context
- `git` for branch and diff inspection

## Safety Notes

- Do not invent package scripts; always confirm in `package.json` or workspace docs.
- Stop and ask when auth, tokens, or signing are required but missing.
