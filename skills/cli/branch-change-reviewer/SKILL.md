---
name: cli-branch-change-reviewer
description: Use with branch-change-reviewer when reviewing branch diffs in the CLI product repository. Adds CLI monorepo evidence through cli-technical-analysis and formats ordinary review findings with caveman-review. Use for read-only CLI branch reviews covering regressions, architecture, package boundaries, configuration precedence, CI parity, and tests.
---

# CLI Branch Change Reviewer

Apply CLI-specific evidence and terse output on top of
`branch-change-reviewer`; inherit all base scope, artifact, parallel-review,
and safety rules.

## When to Use

Use for branch-diff reviews in the CLI product repository.

## When Not to Use

- Work outside the CLI repository
- Implementation or fix requests
- PR thread grouping without a branch review; use `cli-pr-comment-analysis`

## First Read

1. Load `branch-change-reviewer`.
2. Load `cli-technical-analysis`; it supplies the required
   `repository-technical-analysis` partner.
3. Load `caveman-review`. If unavailable, format each ordinary finding as one
   actionable line containing location, problem, and fix or verification.
4. Load `github-pr-comment-analysis` only when unresolved PR comments are part
   of the requested branch review.

## Workflow

1. Follow `branch-change-reviewer` end to end.
2. When PR comments are in scope, use `github-pr-comment-analysis` to refresh
   and group unresolved threads. Use its single main PR artifact as the branch
   review output; do not create a parallel branch-review artifact.
3. Apply `cli-technical-analysis` to changed packages and affected callers.
4. When review is parallelized, split by independent CLI package or surface
   and require each worker to use `cli-technical-analysis` and
   `caveman-review`.
5. Preserve the base report structure. Format ordinary findings as:

   ```text
   - High — packages/foo/src/bar.ts:L42: 🔴 bug: null result reaches `email`. Guard before access.
   ```

6. Use normal prose for security or architectural findings when one line would
   hide necessary rationale, then resume terse findings.

## Validation

Use base validation plus the smallest relevant `package.json` or Turbo command
selected by `cli-technical-analysis`. Caveman formatting never replaces
evidence.

## Outputs / Artifacts

Use the base screen output and review artifact unchanged.

## Companion Skills

- `branch-change-reviewer` (required base)
- `cli-technical-analysis` (CLI evidence)
- `repository-technical-analysis` (shared evidence workflow)
- `github-pr-comment-analysis` (optional unresolved PR threads)
- `caveman-review` (finding format)
- `multi-spawn-agent` (authorized parallel review)

## Safety Notes

Inherit base safety. Never expose tokens or CLI credential values.
