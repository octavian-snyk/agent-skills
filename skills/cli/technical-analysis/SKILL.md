---
name: cli-technical-analysis
description: >-
  Use with repository-technical-analysis when investigating the CLI product repository. Adds
  TypeScript/JavaScript monorepo anchors, package-script-based repro commands, CI parity hints,
  optional Slack MCP context when configured, and artifact expectations. Agent- and IDE-agnostic
  except Slack supplement (Cursor Slack plugin).
---

# CLI Product Technical Analysis

Use this skill as a repo-specific overlay on `repository-technical-analysis` when investigating issues in the **CLI product** source tree.

## When to Use

Use this skill when investigation-first work happens in the CLI repository and the user needs:

- commands aligned with pnpm workspaces and Turbo when present
- narrowing reproduction paths for CLI entrypoints, packaged binaries, or adjacent SDK surfaces
- analysis artifacts consistent with shared schema in `ARTIFACTS.md`

## When Not to Use

Do not use this skill when:

- the investigation target is outside the CLI product repository
- generic `repository-technical-analysis` suffices without repo-local anchors
- the task is only GitLab, Jira, or CircleCI transport with no local code path

## First Read

- Read `AGENTS.md` and the root `README.md` or `CONTRIBUTING.md` when present.
- Inspect `package.json` scripts to choose the smallest command that reproduces the signal.
- Load `repository-technical-analysis` for the shared investigation structure. Use `circleci` when the failure or metrics live in CircleCI. Use this file only for this CLI repo’s specifics.
- When local code, CI, Jira, or GitHub context is thin, check whether the **Cursor Slack plugin** (hosted MCP at `https://mcp.slack.com/mcp`) is connected; if so, treat Slack as an optional supplement — not a substitute for repo evidence.
- If `$ARTIFACTS/<meaningful_id>/task_<issue>.md`, `review_mr_<MR>.md`, or `analysis_mr_<MR>.md` exists (or legacy root-level equivalents), read it first and reuse links and prior hypotheses.

## Workflow

1. Confirm repository root and package manager from lockfiles and `package.json`.
2. Prefer targeted **tests** or **lint/typecheck** commands tied to the affected package path.
3. For cross-package behavior, use Turbo filters (`turbo run … --filter=…`) when `turbo.json` exists.
4. Capture reproduction as a **shell transcript**: cwd, exact script, exit code, and relevant log lines.
5. When behavior depends on **installed or project-level CLI configuration**, follow the product’s documented paths and precedence; never paste secrets into artifacts or chat output.
6. When analysis produces or extends an artifact (see repo `ARTIFACTS.md`), keep durable sections such as fastest repro, known false leads, and CI gaps. Use `$KNOWLEDGE/analysis_<name>.md` for general reference; use `$ARTIFACTS/<meaningful_id>/analysis_<name>.md` for ticket/session work.
7. **Optional Slack context** — when repo-local evidence is insufficient and Slack MCP tools are available (Cursor **Settings → MCP** or marketplace Slack plugin connected):
   - Prefer **search** over broad channel history: Jira/GitHub keys (`CLI-####`, `pr-####`), error strings, subsystem names, incident ids, or named teammates from the task.
   - Resolve people with user search when the task names an email or display name; scope message search with `from:<@USER_ID>` when useful.
   - Read **threads** for reproduction steps, rollout timing, config changes, or prior fixes; cite channel + thread date + short snippet in the artifact.
   - Do **not** use Slack when local repro already explains the failure, when the user did not imply team/incident context, or when MCP is unavailable — say which case applies and continue without it.
   - Redact tokens, customer data, and credential values from artifacts and chat; link to Slack messages instead of pasting secrets.
8. **When the task includes approved code changes** — after validation passes, shrink the diff: review the full change set (`git diff`), drop out-of-scope edits and debug noise, minimize the patch without changing behavior, and respect monorepo scope—do not shrink by stripping tests or cross-package fixes the investigation required. Re-run the same repro/validation if production code changes materially.

## Validation

- Re-run the smallest repro command after each hypothesis change when practical.
- Align local commands with CI job names when workflows are visible under `.github/`, `.gitlab-ci.yml`, or when CircleCI config or API metadata names jobs and workflows.
- When the task includes approved code changes, complete workflow **step 8** (shrink the diff) after validation passes.
- When Slack MCP was used, note search queries and whether conclusions from Slack are confirmed vs speculative alongside repo evidence.

## Outputs / Artifacts

May produce or enrich:

- `$ARTIFACTS/<meaningful_id>/analysis_<relevant_name>.md` for ticket/session-scoped artifacts
- `$KNOWLEDGE/analysis_<relevant_name>.md` for general knowledge reference (extend existing paths in place)

## Companion Skills

- `repository-technical-analysis` (required partner)
- `circleci` when investigation needs CircleCI pipeline, workflow, or job facts from the API
- `diagnose` when debugging concrete failures before broad analysis
- **Cursor Slack plugin** (optional) when team discussion, incident threads, or informal repro notes may fill gaps left by code, CI, Jira, and GitHub — MCP last, after local and bundled transport per `AGENTS.md`

## Safety Notes

- Do not exfiltrate tokens, API keys, or full credential files; refer to redacted values only.
- Stop when reproduction requires undisclosed credentials or signing keys.
- Slack MCP requires workspace admin approval and OAuth; if tools are missing or auth fails, state that and proceed with repo/CI sources only — do not ask the user to paste full Slack exports unless they offer.
