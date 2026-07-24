# github-pr-branch-review-comments

Extension of **`branch-change-reviewer`**. Consumes diff review findings and writes **draft Conventional Comments** plus a **Review verdict** (approve now vs wait for author).

See `SKILL.md` and [CONVENTIONAL-COMMENTS.md](CONVENTIONAL-COMMENTS.md).

## Modes

| Mode | Input | Threads | Output artifact |
| --- | --- | --- | --- |
| **PR** | PR number or URL | `github-pr-comment-analysis` (reviewer mode) | `$ARTIFACTS/pr-<PR>/pr_branch_review_comments_<PR>.md` |
| **Branch-only** | No PR — current branch vs target (default `origin/main`) | Skipped | `$ARTIFACTS/<branch>/branch_review_comments_<branch>.md` |

## Typical flow

**PR mode**

1. `GITHUB-ACCESS.md` — `gh-fetch pr <PR> --full`
2. `github-pr-comment-analysis` — group threads; classify author replies
3. `branch-change-reviewer` — diff review
4. This skill — dedupe, draft comments, verdict

**Branch-only mode**

1. `branch-change-reviewer` — diff review on current branch vs target
2. This skill — collapse findings, draft comments, verdict (no GitHub fetch)

Does **not** post to GitHub unless the user asks.

## Distinction

| Skill | Role |
| --- | --- |
| `branch-change-reviewer` | Diff findings (Markdown) |
| `github-pr-comment-analysis` | PR mode: thread intake |
| `github-pr-branch-review-comments` | Draft comments + approval recommendation |
