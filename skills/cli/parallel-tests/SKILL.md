---
name: cli-parallel-tests
description: >-
  Use for broad parallel validation in the CLI product repository: run documented test or CI-equivalent
  scripts from package.json or Turbo, optionally split suites across workers when separate scripts exist,
  then review failures with repository-technical-analysis plus cli-technical-analysis.
---

# CLI Product Broad Test Execution

Use this skill when the goal is **wide** automated validation for the **CLI product** repository rather than a single targeted test file.

## When to Use

Use this skill when:

- the user asks to run “all tests”, “CI locally”, or “full suite before merge”
- you need parallel runners for disjoint script groups (for example packaging vs integration) **and** the repo exposes separate scripts for those groups
- follow-up triage of failures is expected after the run

## When Not to Use

Do not use this skill when:

- a single package or file needs a narrow `pnpm test path` style command (prefer direct invocation plus `tdd`)
- the repository is not the CLI product source tree
- the user has not authorized subagents and you rely on a **two-runner split** that only makes sense in parallel

## First Read

- Read `AGENTS.md`.
- Parse **`package.json` `scripts`** and `turbo.json` (if any) to find canonical `test`, `lint`, `test:unit`, `test:integration`, or CI-equivalent scripts. **Do not invent script names.**
- If the user provides a task or analysis artifact, read it first for environment assumptions.

## Workflow

1. From the repository root, install dependencies using the repo’s documented flow (for example `pnpm install` with the lockfile committed).
2. Identify one or two **documented** primary test drivers:
   - Prefer a single `pnpm test` (or `turbo run test`) when that is what CI uses.
   - If `package.json` distinguishes `test:unit` and `test:integration` (or workspace equivalents), you may run them in separate shells or subagents.
3. When subagents are authorized **and** two disjoint scripts exist, spawn two workers with non-overlapping ownership:
   - Worker A: first script group
   - Worker B: second script group
4. Otherwise run scripts **sequentially** in dependency order (build before tests when documented).
5. Capture exit codes, failing test names, and stderr highlights for each run.
6. After completion, use `repository-technical-analysis` with `cli-technical-analysis` to interpret failures, flaky infrastructure, or missing prerequisites.
7. Preserve durable notes such as `Frequent Failure Clusters` or `CI Parity Gaps` when reruns confirm them.

## Validation

- When splitting disjoint suites locally, run **`scripts/check_skill_prereqs.sh parallel-tests`**. If GNU `parallel` is missing, **ask the user** to install using the **OS-appropriate** suggestion from the helper before falling back to sequential runs only.
- Align script choice with CI workflow files when available (`.github/workflows`, `.gitlab-ci.yml`).
- Prefer the same Node/pnpm versions CI documents (`.node-version`, `packageManager` field).

## Outputs / Artifacts

- Console-ready summary of each suite
- Optional pointers into `analysis_*.md` when deeper follow-up is produced

## Companion Skills

- `cli-technical-analysis`
- `repository-technical-analysis`
- `cli-contributor` when fixes are ready to implement

## Safety Notes

- Do not run destructive scripts or publish tasks unless the user explicitly asks.
- Redact secrets from test logs before writing artifacts.
