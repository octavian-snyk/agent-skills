---
name: cli-pr-comment-analysis
description: >-
  Analyze GitHub pull request review threads for the CLI product repo by consuming grouped PR issues
  from `github-pr-comment-analysis` and adding repository-specific technical analysis. Use when given a PR number or URL, pairing
  repository-technical-analysis with cli-technical-analysis for each grouped issue and cli-contributor
  for proposed fixes. Optional Slack MCP context when configured. Agent- and IDE-agnostic except
  Slack supplement (Cursor Slack plugin).
---

# CLI Product PR Comment Analysis

GitHub hosts **pull requests (PR)**; fetch transport per synced **`GITHUB-ACCESS.md`** (`gh`, `gh api`, then GitHub MCP when local tools are insufficient).

Use this skill from the **CLI product** repository root when unresolved PR review threads need deep, code-grounded responses.

It layers repo-specific conclusions on top of **`GITHUB-ACCESS.md`** transport plus **`github-pr-comment-analysis`** grouped-comment sections inside **`$ARTIFACTS/<meaningful_id>/review_pr_<PR>.md`** (or **`$ARTIFACTS/…/analysis_pr_<PR>.md`** when that file is the session artifact; legacy root-level files remain valid when already present).

## When to Use

Use this skill when:

- grouped unresolved comments exist for a pull request in this CLI repository and local code inspection is required
- the user wants verdicts, risk notes, and proposed changes tied to this repository’s layout and tooling

## When Not to Use

Do not use this skill when:

- plain PR fetch or listing comments is enough (**`GITHUB-ACCESS.md`** + `gh`, without grouped-comment workflow)
- work happens outside the CLI product repository
- no GitHub PR context exists for the request

## First Read

- Read `AGENTS.md` before commands.
- Read synced **`GITHUB-ACCESS.md`** to normalize repository identity, canonical PR number, URLs, and review or review-comment threads; read `github-pr-comment-analysis` for grouped issues and scaffolds.
- Pair `repository-technical-analysis` with `cli-technical-analysis` for each grouped item.
- Use `cli-contributor` to draft concrete code-level responses or patches after analysis.
- If `$ARTIFACTS/…/review_pr_<PR>.md`, `$ARTIFACTS/…/analysis_pr_<PR>.md`, or a legacy root-level equivalent exists, reuse it before repeating upstream fetch work.
- When a grouped comment lacks context (design rationale, prior agreement, incident link) and GitHub/Jira do not explain it, see **Slack context** below if the **Cursor Slack plugin** (`https://mcp.slack.com/mcp`) is connected.

## Inputs

Prefer, in order:

1. Main artifact `$ARTIFACTS/<meaningful_id>/review_pr_<PR>.md` or `$ARTIFACTS/…/analysis_pr_<PR>.md` (or legacy root-level file) whose `## Grouped unresolved comments` / `### issue_*` sections were produced or refreshed by `github-pr-comment-analysis`
2. Normalized PR context from **`GITHUB-ACCESS.md`** + `gh`
3. Raw PR number or URL — fetch per **`GITHUB-ACCESS.md`**, run **`github-pr-comment-analysis`** when grouped sections are missing inside the main artifact

## Workflow

1. Start at the CLI product repository root.
2. Ensure the main artifact (default `$ARTIFACTS/pr-<PR>/review_pr_<PR>.md` or `analysis_pr_<PR>.md` per repository `ARTIFACTS.md`; reuse legacy root paths when already present) contains up-to-date grouped subsections; if not, run **`GITHUB-ACCESS.md`** fetch → **`github-pr-comment-analysis`** before enriching each `### issue_*` block.
3. For each `### issue_*` under `## Grouped unresolved comments`, inspect relevant packages using `repository-technical-analysis` **and** `cli-technical-analysis`.
4. Add technical verdicts, risks, and prerequisites **inside that subsection** using evidence from the tree.
5. When code changes are appropriate, record proposed diffs or commands via `cli-contributor` conventions (tests-first when fixing regressions)—still within the same subsection unless the user directs otherwise.
6. When a grouped issue still lacks background after code inspection, run **Slack context** (optional) for that issue before finalizing the verdict or reply draft.
7. If `multi-spawn-agent` is explicitly authorized, parallelize independent grouped issues with **disjoint `### issue_*` subsection ownership** in the single main artifact (no parallel edits to the same subsection).
8. Finish with a short on-screen summary plus the **full path** to the single main artifact (e.g. `$ARTIFACTS/pr-336/review_pr_336.md`).

## Slack context

Optional supplement when **Slack MCP tools** are available (Cursor **Settings → MCP** or marketplace Slack plugin connected). Use **after** GitHub PR threads, local code, and Jira — not instead of them. Pair with **`cli-technical-analysis`** or **`cli-contributor`** for broader Slack search patterns.

**Skip** when the PR thread and code already answer the comment, or when MCP is missing or auth fails — state which and continue.

**PR comment use cases:**

- reviewer references a Slack thread, incident, or offline decision not quoted on the PR
- need the author’s or reviewer’s earlier rationale (`from:<@USER_ID>` + PR or topic keywords)
- rollout, flag, or customer-impact discussion tied to the PR branch or Jira key
- confirming whether feedback was already addressed in team chat before drafting a reply

**Tool ladder:**

1. **`slack_search_public_and_private`** — start with `pr-<PR>`, PR URL slug, branch name, or linked `CLI-####` from the artifact; add reviewer keywords from the comment text.
2. **`slack_search_users`** — when the comment names a person by email or display name; follow with `from:<@USER_ID>`.
3. **`slack_read_thread`** — read threads behind hits that look like the discussion the comment references.

Record in the relevant `### issue_*` subsection: query used, channel + date + one-line snippet, and whether Slack **confirms** or only **suggests** the reply angle. Redact secrets; do not paste tokens or customer data.

## Validation

- Thread conclusions should cite file paths, tests, or configs checked.
- Prefer reproducible commands named in `package.json` over ad hoc invocations.
- When Slack informed a subsection, note the search anchor and label conclusions grounded in Slack as confirmed vs speculative.

## Companion Skills

- **`GITHUB-ACCESS.md`**, `github-pr-comment-analysis`
- `repository-technical-analysis`, `cli-technical-analysis`, `cli-contributor`
- **Cursor Slack plugin** (optional) — **Slack context** section; MCP last, after GitHub and local code per `AGENTS.md`
- `multi-spawn-agent` only when authorized

## Safety Notes

- Do not post to GitHub automatically unless the user asks; produce review-ready text instead.
- Strip tokens from quoted API or CI snippets.
- Slack MCP requires workspace admin approval and OAuth; if tools are missing or auth fails, proceed without Slack — do not ask the user to paste full Slack exports unless they offer.

## Outputs / Artifacts

- Enriched main artifact under `$ARTIFACTS/<meaningful_id>/` (grouped subsections only), produced jointly with `github-pr-comment-analysis`; legacy root-level paths when already the working file
- Short summary of per-thread verdicts and the artifact’s full path
- At the top of the main artifact, state `Analysis date: YYYY-MM-DD` and
  `Analyzed commit: <full PR head SHA>` from normalized PR context; do not use
  the local checkout SHA unless it matches the PR head.
