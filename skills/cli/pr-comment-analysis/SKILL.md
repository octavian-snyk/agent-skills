---
name: cli-pr-comment-analysis
description: >-
  Analyze GitLab merge request comments for the CLI product repo by consuming grouped MR issues and
  adding repository-specific technical analysis. Use when given an MR IID or URL, pairing
  repository-technical-analysis with cli-technical-analysis for each grouped issue and cli-contributor
  for proposed fixes. Agent- and IDE-agnostic.
---

# CLI Product PR Comment Analysis

Skill id uses **PR** (pull request) naming; GitLab still exposes these as **merge requests (MR)**. Upstream transport (`gitlab`, `gitlab-mr-comment-analysis`) remains MR-specific.

Use this skill from the **CLI product** repository root when unresolved MR review threads need deep, code-grounded responses.

It layers repo-specific conclusions on top of the generic GitLab MR analysis workflow (`gitlab`, `gitlab-mr-comment-analysis`).

## When to Use

Use this skill when:

- grouped unresolved comments exist for a merge request in this CLI repository and local code inspection is required
- the user wants verdicts, risk notes, and proposed changes tied to this repository’s layout and tooling

## When Not to Use

Do not use this skill when:

- plain MR fetch or grouping is enough (`gitlab`, `gitlab-mr-comment-analysis` only)
- work happens outside the CLI product repository
- no GitLab MR context exists for the request

## First Read

- Read `AGENTS.md` before commands.
- Read `gitlab` to normalize MR identity, links, and threads; read `gitlab-mr-comment-analysis` for grouped issues and scaffolds.
- Pair `repository-technical-analysis` with `cli-technical-analysis` for each grouped item.
- Use `cli-contributor` to draft concrete code-level responses or patches after analysis.
- If `review_mr_<MR>.md` or `analysis_mr_<MR>.md` exists, reuse it before repeating upstream fetch work.

## Inputs

Prefer, in order:

1. Artifacts already produced by `gitlab-mr-comment-analysis` (`work_plan_mr_<MR>.md`, grouped issue files)
2. Normalized MR context from `gitlab`
3. Raw MR IID or URL (must flow through `gitlab` then `gitlab-mr-comment-analysis` first)

## Workflow

1. Start at the CLI product repository root.
2. Ensure grouped issues exist; if not, run upstream `gitlab` → `gitlab-mr-comment-analysis` before local enrichment.
3. For each grouped issue in the work plan, inspect relevant packages using `repository-technical-analysis` **and** `cli-technical-analysis` (scripts, workspace boundaries, CLI entrypoints).
4. Add technical verdicts, risks, and prerequisites to the grouped-issue analysis Markdown using evidence from the tree.
5. When code changes are appropriate, record proposed diffs or commands via `cli-contributor` conventions (tests-first when fixing regressions).
6. If `multi-spawn-agent` is explicitly authorized, parallelize independent grouped issues with disjoint file ownership.
7. Finish with a short on-screen summary plus paths to updated analysis files.

## Validation

- Thread conclusions should cite file paths, tests, or configs checked.
- Prefer reproducible commands named in `package.json` over ad hoc invocations.

## Companion Skills

- `gitlab`, `gitlab-mr-comment-analysis`
- `repository-technical-analysis`, `cli-technical-analysis`, `cli-contributor`
- `multi-spawn-agent` only when authorized

## Safety Notes

- Do not post to GitLab automatically unless the user asks; produce review-ready text instead.
- Strip tokens from quoted API or CI snippets.

## Outputs / Artifacts

- Enriched grouped-issue Markdown files produced by `gitlab-mr-comment-analysis`
- Short summary of per-thread verdicts and file paths for handoff
