---
name: cli-caretaker
description: Research a CLI Ask Caretaker shift and write an evidence-backed report with advised actions for the CLI Ask queue, support triage, Datadog signals, alerts, main-branch CI failures, redirected requests, and PR asks. Use when the user asks for CLI caretaker analysis, Ask or support triage, a caretaker report or handoff, or advice on whether an item should be answered, escalated, converted, reviewed, or closed. Exclude the separate On Caller role and on-call incident work. Keep Jira, Slack, GitHub, CircleCI, and Datadog interactions read-only unless the user explicitly requests a specific external action.
---

# CLI Ask Caretaker

Protect the CLI team's sprint focus by researching the queue and producing a
technical-analysis-style report with evidence and advised actions.

## When to Use

Use for a CLI caretaker shift, Ask or support triage, or caretaker handoff.

## When Not to Use

Do not use for an unrelated repository, the separate CLI `On Caller` role,
on-call rotations or handoffs, paging or incident command, or deep bug
investigation after initial triage.

## Scope Boundary

- Model only `Roles and Responsibilities > CLI Ask Caretaker`, including its
  Ask handling, support triage, alert-channel monitoring, and `main` CI triage.
- Ignore `Roles and Responsibilities > On Caller` and its included on-call
  guide. Do not absorb on-call duties merely because they appear on the same
  Confluence page.
- Treat alerts as caretaker research and routing only; do not assume incident
  response or on-call ownership.

## Source of Truth

- Refresh the [CLI Support page](https://snyksec.atlassian.net/wiki/spaces/CLI/pages/2120417317/Support)
  with `confluence` when practical, but read only the `CLI Ask Caretaker` scope
  and its support-triage guidance. Live guidance wins over this summary.
- Use the [support/ask decision diagram](https://miro.com/app/board/uXjVKD3VYxU=/)
  as a secondary workflow source. Prefer newer Confluence or Jira policy when
  they conflict.
- Review the [CLI Support Board](https://snyksec.atlassian.net/jira/software/c/projects/CLI/boards/715)
  and the `Triage Status` section of the
  [CLI Support Dashboard](https://snyksec.atlassian.net/jira/dashboards/10913).
- Use Jira transport per `JIRA-ACCESS.md`. Use connected Slack tools for
  `#ask-cli`, `#cli-alerts`, and `#hammerhead-alerts` when available.
- Use the connected Datadog app and its skill guides for relevant observability
  evidence when available.

## Inputs

- Accept an optional shift window, queue snapshot, issue list, or handoff.
- Keep Jira, Slack, GitHub, CircleCI, and Datadog access read-only by default.
- Treat a request to research, review, triage, prepare, or run the caretaker
  report as authorization to read evidence only—not to comment, transition,
  close, assign, message, reply, or submit a PR review.
- Perform an external action only when the user explicitly asks for that
  specific interaction. Confirm destructive, ambiguous, or bulk changes.

## Workflow

1. Research current work through read-only access:
   - every Ask in the Asks swimlane outside `Done`
   - new support requests in dashboard `Triage Status`
   - relevant CLI and Hammerhead alerts
   - CI/CD failures on `main` in the
     [snyk/cli CircleCI pipelines](https://app.circleci.com/pipelines/gh/snyk/cli),
     using `circleci` read-only
   - requests redirected to the team outside `#ask-cli`
2. For alerts, CI failures, errors, or performance symptoms with observable
   signals, research Datadog read-only:
   - list Datadog skill guides, then load the best matching guide before using
     related tools; common matches include logs, incidents and alerting, and
     change tracking
   - start with the item identifier, service, error text, and narrowest useful
     time window; inspect relevant monitors, incidents, logs, metrics, traces,
     and changes
   - record queries, time ranges, result links, and whether evidence confirms
     or only suggests the diagnosis
   - skip Datadog when the item has no observability signal or existing evidence
     already answers the caretaker question
3. Classify each Ask and advise the matching branch:
   - Simple question, remark, or update: draft the answer and advise closure. If
     the answer is unclear, advise requesting help in `#ask-cli` and tracking
     the follow-up.
   - Feature request: advise asking the reporter to create an Aha! entry and
     closing the Ask after the request is captured.
   - Customer-reported bug: advise asking the reporter to create or link a
     Zendesk ticket with logs, screenshots, and reproducible steps, then close
     the Ask and continue through the support process.
   - Non-customer bug: advise creating a bug in the CLI/IDE Jira project,
     linking it from the Ask, and prioritizing it through KLO/Cooldown planning.
   - Deeper documentation problem rather than a product bug: advise a
     documentation tech-debt ticket; if an answer is time-sensitive, advise
     marking it as a `Cycle <X> Cooldown candidate` for team discussion.
   - PR ask: inspect PR evidence read-only and advise review and closure steps.
     Do not submit a review or close the Ask.
4. Apply the 30-minute gate. If an Ask needs more than 30 minutes, advise the
   appropriate tracked-work branch above instead of continued Ask-channel
   investigation.
5. Perform initial SUP triage in 5–10 minutes and within 1–3 days according to
   priority:
   - decide whether CLI can fix it; otherwise advise assigning the owning team
     and moving the ticket to that team's project
   - for `Highest (Critical)`, advise `Backlog`, pulling it into the current
     sprint, and fixing it; otherwise advise `Backlog` for a confirmed CLI bug
   - treat red SUP items as breached due dates and yellow SUP items as breached
     triage dates; surface both prominently in the report
   - identify feature requests; advise `Customer Need` with a short reason
   - sanity-check priority and obvious missing information
   - advise `Won't Fix` with a reason for a bug CLI will not address
   - do not investigate the issue, design a solution, or promise an exact ETA
     during initial triage
6. During sprint-planning work, flag whether roughly 30% of team capacity is
   reserved for support and whether support SLOs are at risk.
7. Write the report. Include a reminder to update Slack `@ask-cli-caretaker`
   when appropriate; do not update it.
8. If the user separately requests a specific external action, perform only
   that action and record the resulting URL.

## Validation

- Cover every visible non-Done Ask and new support-triage item.
- Record the decision branch for source type, CLI ownership, feature/bug type,
  customer origin, and Critical priority when applicable.
- Keep work over 30 minutes out of the Ask queue.
- Cite the evidence supporting each recommendation.
- Leave Jira, Slack, GitHub, CircleCI, and Datadog unchanged unless a specific
  action was explicitly requested.
- Exclude On Caller responsibilities from findings and advised actions.

## Outputs

Write non-trivial research to a user-provided path or, by default,
`$ARTIFACTS/cli-caretaker-YYYY-MM-DD/analysis_cli_caretaker.md`. Resolve the
path per `ARTIFACTS.md`, read an existing same-day report first, and extend it
instead of creating a duplicate.

Report each item with:

- identifier and source
- classification: answer, needs help, PR review, customer support, CLI bug,
  feature request, alert, or CI failure
- advised action and rationale
- owner, blocker, and follow-up when known
- link to the Ask, ticket, PR, Datadog evidence, alert, or CI run

Finish with queue counts, evidence gaps, prioritized advised actions, and a
share-ready caretaker handoff. Clearly separate observed state from advice.

## Companion Skills

- Use `confluence` to refresh the support guidance.
- Use `circleci` read-only for `main` pipeline checks in `gh/snyk/cli`.
- Use `cli-branch-change-reviewer` read-only for PR asks requiring code review;
  do not post its findings to GitHub.
- Use `cli-technical-analysis` only after work has left initial triage and a
  deeper investigation is explicitly requested.
- Use the connected Datadog app's skill discovery first, then load the relevant
  Datadog guide before querying telemetry.

## Safety Notes

- Do not paste customer data, credentials, private logs, or Salesforce content
  into chat or artifacts.
- Never infer write authorization from a request to research, triage, review,
  prepare, run, or report.
- Draft comments and messages inside the report; do not send them unless the
  user explicitly requests sending.
