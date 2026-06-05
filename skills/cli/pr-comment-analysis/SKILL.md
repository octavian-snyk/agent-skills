---
name: cli-pr-comment-analysis
description: >-
  Analyze GitHub pull request review threads for the CLI product repo by consuming grouped PR issues
  from `github-pr-comment-analysis` and adding repository-specific technical analysis. Use when given a PR number or URL, pairing
  repository-technical-analysis with cli-technical-analysis for each grouped issue and cli-contributor
  for proposed fixes. Agent- and IDE-agnostic.
---

# CLI Product PR Comment Analysis

GitHub hosts **pull requests (PR)**; use **`github`** for transport (`gh`, GitHub MCP, or `gh api`).

Use this skill from the **CLI product** repository root when unresolved PR review threads need deep, code-grounded responses.

It layers repo-specific conclusions on top of **`github`** transport plus **`github-pr-comment-analysis`** grouped-comment sections inside **`$ARTIFACTS/<meaningful_id>/review_pr_<PR>.md`** (or **`$ARTIFACTS/…/analysis_pr_<PR>.md`** when that file is the session artifact; legacy root-level files remain valid when already present).

## When to Use

Use this skill when:

- grouped unresolved comments exist for a pull request in this CLI repository and local code inspection is required
- the user wants verdicts, risk notes, and proposed changes tied to this repository’s layout and tooling

## When Not to Use

Do not use this skill when:

- plain PR fetch or listing comments is enough (`github` only, without grouped-comment workflow)
- work happens outside the CLI product repository
- no GitHub PR context exists for the request

## First Read

- Read `AGENTS.md` before commands.
- Read `github` to normalize repository identity, canonical PR number, URLs, and review or review-comment threads; read `github-pr-comment-analysis` for grouped issues and scaffolds.
- Pair `repository-technical-analysis` with `cli-technical-analysis` for each grouped item.
- Use `cli-contributor` to draft concrete code-level responses or patches after analysis.
- If `$ARTIFACTS/…/review_pr_<PR>.md`, `$ARTIFACTS/…/analysis_pr_<PR>.md`, or a legacy root-level equivalent exists, reuse it before repeating upstream fetch work.

## Inputs

Prefer, in order:

1. Main artifact `$ARTIFACTS/<meaningful_id>/review_pr_<PR>.md` or `$ARTIFACTS/…/analysis_pr_<PR>.md` (or legacy root-level file) whose `## Grouped unresolved comments` / `### issue_*` sections were produced or refreshed by `github-pr-comment-analysis`
2. Normalized PR context from `github`
3. Raw PR number or URL — resolve via `github`, run **`github-pr-comment-analysis`** when grouped sections are missing inside the main artifact

## Workflow

1. Start at the CLI product repository root.
2. Ensure the main artifact (default `$ARTIFACTS/pr-<PR>/review_pr_<PR>.md` or `analysis_pr_<PR>.md` per repository `ARTIFACTS.md`; reuse legacy root paths when already present) contains up-to-date grouped subsections; if not, run upstream `github` → `github-pr-comment-analysis` before enriching each `### issue_*` block.
3. For each `### issue_*` under `## Grouped unresolved comments`, inspect relevant packages using `repository-technical-analysis` **and** `cli-technical-analysis`.
4. Add technical verdicts, risks, and prerequisites **inside that subsection** using evidence from the tree.
5. When code changes are appropriate, record proposed diffs or commands via `cli-contributor` conventions (tests-first when fixing regressions)—still within the same subsection unless the user directs otherwise.
6. If `multi-spawn-agent` is explicitly authorized, parallelize independent grouped issues with **disjoint `### issue_*` subsection ownership** in the single main artifact (no parallel edits to the same subsection).
7. Finish with a short on-screen summary plus the **full path** to the single main artifact (e.g. `$ARTIFACTS/pr-336/review_pr_336.md`).

## Validation

- Thread conclusions should cite file paths, tests, or configs checked.
- Prefer reproducible commands named in `package.json` over ad hoc invocations.

## Companion Skills

- `github`, `github-pr-comment-analysis`
- `repository-technical-analysis`, `cli-technical-analysis`, `cli-contributor`
- `multi-spawn-agent` only when authorized

## Safety Notes

- Do not post to GitHub automatically unless the user asks; produce review-ready text instead.
- Strip tokens from quoted API or CI snippets.

## Outputs / Artifacts

- Enriched main artifact under `$ARTIFACTS/<meaningful_id>/` (grouped subsections only), produced jointly with `github-pr-comment-analysis`; legacy root-level paths when already the working file
- Short summary of per-thread verdicts and the artifact’s full path
