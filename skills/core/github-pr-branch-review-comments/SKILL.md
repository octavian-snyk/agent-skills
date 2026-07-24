---
name: github-pr-branch-review-comments
description: >-
  Transform branch-change-reviewer findings into polite, question-oriented Conventional Comments
  and an approval verdict. Draft in plain language (what the code does before why it matters; no
  insider jargon); treat lint suppressions and tooling workarounds explicitly
  (what/why/required/alternatives). With a GitHub PR number or URL, consume existing PR threads via
  github-pr-comment-analysis in reviewer mode. Without a PR, run on the current branch diff only.
  Collapse overlapping findings and dedupe against answered threads when PR context exists. Use when
  the user wants draft review comment text (not auto-posted) plus approve-now vs wait-for-author.
---

# GitHub PR Branch Review Comments

Extension of **`branch-change-reviewer`**. Turn diff findings into **draft review comments** in [Conventional Comments](https://conventionalcomments.org/) form — question-first, polite, actionable — plus a **Review verdict** (approvable now vs author answers needed first).

The point of this skill is the **output**: a self-contained review the human can act on **without asking follow-up questions**. Every artifact must describe **what the code was and what changed** in plain language before any opinion. Do **not** post to GitHub unless the user explicitly asks.

**Two modes**, chosen from inputs:

| Mode | Trigger | PR threads | Default artifact |
| --- | --- | --- | --- |
| **PR** | PR number or URL given | Yes — reviewer-mode intake | `$ARTIFACTS/pr-<PR>/pr_branch_review_comments_<PR>.md` |
| **Branch-only** | No PR given | Skipped | `$ARTIFACTS/<sanitized-branch>/branch_review_comments_<sanitized-branch>.md` |

## When to Use

When the user wants draft Conventional Comments from a branch diff (with or without a PR), overlapping findings merged per root cause, and an approve-now vs wait-for-author call.

## When Not to Use

Branch review without comment drafting (**`branch-change-reviewer`**), thread analysis without new diff comments (**`github-pr-comment-analysis`**), GitLab PRs (**`gitlab`** + **`gitlab-mr-comment-analysis`**), or code fixes (contributor / TDD skills).

## Inputs

- **PR mode** when a GitHub PR number or URL is present. Optional: existing `branch-change-reviewer` artifact, explicit output path, `meaningful_id` override (default `pr-<PR>`).
- **Branch-only mode** otherwise. Optional: target branch (default `origin/main`), head branch (default: current checkout), existing review artifact.
- Do **not** auto-discover a PR in branch-only mode.

## First read

- Repository `AGENTS.md`.
- **`branch-change-reviewer`** — review standards and finding shape.
- [CONVENTIONAL-COMMENTS.md](CONVENTIONAL-COMMENTS.md) — labels, decorations, tone, examples.
- **PR mode only:** synced **`GITHUB-ACCESS.md`** and **`github-pr-comment-analysis`**; run `check_skill_prereqs.sh github` and `check_skill_config.sh github` when `gh` is needed.

> **Paths:** never hardcode an install root. Resolve synced helpers at run time — GitHub helpers via `GITHUB-ACCESS.md` (or `agent_config.py --github-scripts-dir`), artifact paths via `resolve_artifact_path.py`. Reference helpers by bare name (`gh-fetch`, `bootstrap_github_artifact.py`, `apply_pr_thread_groups.py`).

## Reviewer-side conversation model (PR mode only)

You are on the **reviewer** side: your draft comments are new questions/suggestions; existing PR threads are the **author's** answers. Use `github-pr-comment-analysis` transport (`gh-fetch pr <PR> --full`) but read threads as *reviewer asked → author replied*, not as fix tasks.

For each existing thread: identify the original concern, classify author replies (`answered_sufficiently` | `partially_answered` | `unanswered` | `needs_code_change`), and do not re-ask what the author already addressed unless the current diff contradicts the reply. Overlap with a new finding → merge into one follow-up or mark `covered_by_existing_thread`. Threads still `unanswered` / `partially_answered` on a material concern count toward **wait-for-author** even if this diff adds nothing new.

## Workflow

### 1. Resolve scope

- **PR mode:** from the GitHub repo root, `gh-fetch pr <PR> --full`; record `pr_number`, canonical URL, head/base branches and SHAs, author, changed files, thread counts. Bootstrap/refresh grouped threads (`bootstrap_github_artifact.py --fetch --pr <PR>`, then `apply_pr_thread_groups.py --fetch --pr <PR> --artifact <review_pr_<PR>.md>`) in **reviewer mode**. Diff scope = PR `base...head`.
- **Branch-only mode:** `git status --short --branch` for head; accept optional target (default `origin/main`); `git fetch`; diff scope = `origin/<target>...<head>`.

### 2. Ensure branch-change-reviewer findings exist

Prefer a current review artifact (`$ARTIFACTS/pr-<PR>/review_<head>.md` or `$ARTIFACTS/<branch>/review_<branch>.md`; then user path; then legacy root-level). If missing or stale, run **`branch-change-reviewer`** against the resolved scope **without writing code** (remote diff is enough; checkout optional). Its `## Findings` are the source of truth for new comments; cross-check the PR thread table before drafting.

### 3. Build change context — the core of the output (mandatory)

Write `## Change context` **before** collapse and comments. This is what removes follow-up questions. **Do not skip it because the diff is small.**

**Read one level deeper than the diff.** Changed files are often wiring (CI config, thin adapters, flags). Trace what the diff invokes or affects and describe **observable outcomes**, not just moved lines. Ask *"if I owned this area, what would I need to know before nitpicking?"* and answer it here. Infer relevance from PR title, changed paths, and repo context — do not hardcode domain checklists.

**Mechanical summary (every PR)** — use counts and names, not vague prose:

- **Files changed** — count and list
- **What was → what is** — the prior behavior/shape and the new one for each meaningful change (new/removed/renamed jobs, steps, functions, params, dependency edges)
- **Pattern** — what existing code was copied or extended
- **Complexity** — simple/localized vs moderate vs complex, one line why

**Downstream / outcome semantics (when behavior may extend beyond the changed file):** trace calls, imports, and config until you can state what changes for users/operators/dependents, how variants interact (envs, flags, channels, branches), the blast radius if it fails, and whether conflict/regression is plausible (say why if not). Put outcome semantics **before** wiring/style concerns.

If `branch-change-reviewer` findings stop at the diff surface, **backfill** change context (and its findings) from traced code before drafting.

### 4. Plain language and explainability (mandatory)

The human may not have the diff open or share your vocabulary. **State what the code does or changed before why it matters or what you'd prefer.**

- **Theme/subject titles** must be understandable without the diff — describe observable behavior, not shorthand (prefer *"Docs line drops its trailing newline when a Tip follows"* over *"Docs/Tip newline coupling"*). Abstract names (coupling, fragile, drift) go in discussion only after the mechanism is clear.
- **Every comment discussion** follows **what → so what → ask**: concrete behavior from the diff (cite file/line), then impact, then the question/next step. If you can't explain the mechanism plainly, re-read the diff before drafting.
- **Lint suppressions / tooling workarounds** (`//nolint`, `eslint-disable`, `# noqa`, `#[allow]`, threshold bumps, `skip`): never bury as a one-line nit. Record **what rule fired**, **why it was added**, **whether it's required** (fails without it vs stylistic), and **alternatives** — in a comment when worth awareness, or in `## Change context` when clearly required and non-controversial. Never leave the what/why/required framing only in the reviewer artifact.
- **Self-check** each title/subject cold as the author: would I know which lines this refers to, what the code does without the diff, and (for tooling) why it was added and whether I can remove it? If any answer is no, rewrite.

### 5. Collapse findings (and dedupe in PR mode)

Merge findings that would repeat the same conversation: same underlying risk/missing guard, same proposed architectural change, same test gap across files, or severity variants of one defect. **PR mode also:** findings matching an open `issue_*` on the same root cause → `follow_up_on: issue_*`; matching a thread the author `answered_sufficiently` → omit unless the diff contradicts. Keep comments separate when root causes, blocking severity, or anchors genuinely differ. Record decisions in `## Collapse map`.

### 6. Draft Conventional Comments

One comment per collapsed group, per [CONVENTIONAL-COMMENTS.md](CONVENTIONAL-COMMENTS.md):

- **Label:** default `question`; `suggestion` when you can name a concrete improvement; `issue` for clear defects (pair with a suggested direction); `nitpick` for style. At most two decorations (`(blocking)`/`(non-blocking)`, `(test)`, `(security)`, `(ux)`).
- **Subject:** one plain-language sentence on observable behavior; `?` for questions.
- **Discussion:** what → so what → ask, with diff evidence.
- **Anchors:** primary file/line (+ secondary when collapsed across files), `Related thread:` in PR mode, `Blocks approval: yes|no`.

Severity map: high/bug/regression → `issue`/`question`, `(blocking)` when merge should wait; medium/architecture/test gap → `question`/`suggestion`, `(non-blocking)`/`(test)`; low/style → `nitpick`/`question`, `(non-blocking)`.

### 7. Review verdict

`## Review verdict` is a recommendation for the human, not an automatic GitHub action. Pick exactly one: **`approve_now`** (no blocking questions), **`comment_only`** (non-blocking questions, merge reasonable), **`wait_for_author`** (blocking/material questions first), **`request_changes`** (confirmed defect/regression).

Rules: any blocking comment → at least `wait_for_author` (`request_changes` for confirmed bug); no blocking + no material questions → `approve_now` (say so if non-blocking nits remain). **PR mode also:** any prior `unanswered`/`partially_answered` material thread → at least `wait_for_author`; all relevant threads `answered_sufficiently` + no blocking new comments → `approve_now`.

### 8. Write the output artifact

Resolve the default path with `resolve_artifact_path.py` (explicit user path wins). Lead the artifact with a **metadata block** so runs are reproducible and comparable, then the sections below.

```markdown
---
generated_at: <UTC ISO-8601, e.g. 2026-07-24T07:15:00Z>
mode: pr | branch-only
head_branch: <head>
head_commit: <short-sha> (<full-sha>)
target_branch: <target>              # branch-only
merge_base: <short-sha>              # branch-only, when derivable
pr: <PR> — <url>                     # PR mode
base_sha: <sha>   head_sha: <sha>    # PR mode
source_review_artifact: <path>
---

# Branch review comment suggestions

## Summary
- Mode; scope (PR link or `<head>` vs `<target>`); source review artifact
- Findings count → suggested comment count after collapse (and dedupe in PR mode)
- PR mode: prior open thread count and author-reply summary

## Change context
- Mechanical summary: files changed; what was → what is (counted, named); pattern; complexity
- Downstream / outcome semantics (when derivable): user/operator/dependent impact; variant interaction; blast radius; conflict/regression — answer implicit "so what?" here, not in follow-ups

## Thread intake (reviewer mode)
- PR mode: issue_* → reviewer concern → author reply → status → overlaps new finding?
- Branch-only: skipped — no PR context

## Collapse map
- finding → comment_01 (theme); PR mode: finding → skipped (covered by issue_02)

## Suggested review comments

### comment_01 — <plain-language theme: what the code does/changes>
- Post on: `path/file.ext` (~line N) · Source findings: … · Related thread: issue_03 (PR) · Blocks approval: yes|no

\`\`\`text
question (non-blocking,test): <subject — observable behavior, not jargon>

<what the code does or changed — 1–2 sentences>
<why it might matter>
<suggested next step or question>
\`\`\`

## Review verdict
- Mode · Recommendation (one value) · Suggested GitHub action (Approve|Comment|Request changes, or N/A branch-only)
- Rationale (2–4 sentences) · Blocking before approval (ids or none) · Non-blocking optional (ids or none) · Confidence (high|medium|low + why)

## Posting notes
- Do not post automatically unless the user asks
- PR mode: one thread per comment_*; reply in Related thread when set
- Branch-only: comments ready to paste when a PR is opened
```

Populate the metadata block from live git/PR state — e.g. `date -u +%Y-%m-%dT%H:%M:%SZ`, `git rev-parse --short HEAD` / `git rev-parse HEAD`, `git merge-base origin/<target> HEAD`, and PR `base`/`head` SHAs from `gh-fetch`. Mirror to screen: lead with **verdict** (one line) and **mode**, then a 2–4 sentence **Change context**, then comment count and artifact path.

## Validation

- Metadata block present and populated (generated_at, mode, head_commit; target/merge_base or pr/base_sha/head_sha per mode).
- `## Change context` present with mechanical summary (counts, named what-was→what-is, complexity) and, when relevant, traced downstream semantics — not diff-surface only.
- Plain language: themes/subjects describe observable behavior; every discussion is what → so what → ask.
- Lint/tooling suppressions carry what/why/required/alternatives in a comment or Change context — not only in the reviewer artifact.
- Mode matches inputs; every comment traces to a finding or a justified `follow_up_on: issue_*`.
- PR mode: threads fetched `--full`; author replies classified; no duplicate questions when `answered_sufficiently` unless the diff contradicts.
- `## Collapse map` documents merges/skips; `## Review verdict` has exactly one recommendation.
- Branch-only: no GitHub fetch required. PR mode, no findings + open author threads → at least `wait_for_author`.

## Outputs / artifacts

- PR mode: `$ARTIFACTS/pr-<PR>/pr_branch_review_comments_<PR>.md` (+ optional `review_pr_<PR>.md` from thread intake)
- Branch-only: `$ARTIFACTS/<sanitized-branch>/branch_review_comments_<sanitized-branch>.md`
- Optional `review_<branch>.md` when `branch-change-reviewer` ran this session
- On-screen: mode, verdict, comment count, artifact full path

## Companion Skills

- **`branch-change-reviewer`** — diff findings consumed here (both modes)
- **`github-pr-comment-analysis`** + **`GITHUB-ACCESS.md`** — PR mode thread intake
- repository overlay skills — only when the user asks for repo-specific fixes after comments are drafted

## Safety Notes

- Do not post to GitHub unless the user explicitly requests it.
- Do not modify application code, tests, or config.
- Do not invent findings beyond the review artifact and diff evidence.
- Strip tokens from quoted CI or API snippets.
