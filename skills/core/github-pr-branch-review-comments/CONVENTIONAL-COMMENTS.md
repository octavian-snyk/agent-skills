# Conventional Comments reference

Canonical spec: [conventionalcomments.org](https://conventionalcomments.org/)

This skill uses Conventional Comments for **draft GitHub PR review text** derived from **`branch-change-reviewer`** findings. Default to **question-oriented**, **polite** phrasing.

## Format

```text
<label> [decorations]: <subject>

[discussion]
```

- **label** — single word such as `question`, `suggestion`, `issue`
- **decorations** — optional, parenthesized, comma-separated: `(non-blocking)`, `(test,security)`
- **subject** — main message; for `question`, prefer ending with `?`
- **discussion** — supporting context, evidence, and suggested next step

## Labels (preferred)

| Label | Use for |
| --- | --- |
| `question` | **Default.** Uncertainty, clarification, or “could we verify…?” — non-accusatory |
| `suggestion` | Concrete improvement with clear rationale |
| `issue` | Specific problem; pair with direction in discussion |
| `nitpick` | Trivial preference; should be non-blocking |
| `thought` | Non-blocking idea for later |
| `praise` | Sincere positive note (optional, at most one per PR when warranted) |

## Decorations (use sparingly)

| Decoration | Meaning |
| --- | --- |
| `(blocking)` | Should be resolved before merge |
| `(non-blocking)` | Author may defer or handle in follow-up |
| `(if-minor)` | Resolve only if the fix stays small |
| `(test)` | Testing / coverage angle |
| `(security)` | Security-sensitive |
| `(ux)` | User-facing behavior |

Use **at most two** decorations per comment.

## Tone rules (this skill)

1. **Question-first** — prefer `question` unless the finding is an clear defect (`issue`) or a specific fix (`suggestion`).
2. **Polite** — “Could we…?”, “Would it make sense…?”, “I might be missing context, but…”
3. **Specific** — cite file, behavior, or test gap from the review finding.
4. **Collaborative** — assume good intent; ask before accusing.
5. **Actionable** — discussion ends with a verifiable next step when possible.
6. **Plain language** — subjects and themes describe **what the code does**, not compressed reviewer jargon.
7. **Mechanism before opinion** — discussion opens with what changed, then why it matters, then the ask.

### Discussion structure (required)

```text
<what the code does or changed — concrete, 1–2 sentences>
<why it might matter to users, operators, or maintainers>
<question, suggestion, or next step>
```

The PR author (or the human pasting your draft) should not need to ask “what do you mean?” or “why is that needed?”

## Severity mapping

| branch-change-reviewer severity | Label | Decorations |
| --- | --- | --- |
| high | `issue` or `question` | `(blocking)` when appropriate |
| medium | `question` or `suggestion` | `(non-blocking)` or domain tag |
| low | `question` or `nitpick` | `(non-blocking)` or `(if-minor)` |

## Examples

**Testing gap (collapsed from two related findings):**

```text
question (non-blocking,test): Could we cover the empty-input path for this handler?

The validation change in `handler.ts` also affects `batch.ts`, but I only see happy-path tests updated. Would extending the existing suite (or adding one shared case) help catch regressions if either caller passes an empty list?
```

**Architecture (single finding):**

```text
question (non-blocking): Would extracting this parsing logic into a shared helper reduce duplication between the two call sites?

Both branches now repeat the same normalization steps. If you agree the behavior should stay identical, a small shared function might make future changes easier to test in one place.
```

**Possible regression (blocking):**

```text
issue (blocking): Might this change return `undefined` when the cache is cold?

The early return skips the fallback that `loader.ts` relied on before. Could we confirm whether callers handle a missing entry, or restore the fallback for the empty-cache case?
```

**Style / nit (low severity):**

```text
nitpick (if-minor): Could we align this name with the `UserSession` type used elsewhere?

Purely for consistency — happy to ignore if you prefer keeping the local alias.
```

**Spacing / layout (mechanism first — not jargon):**

```text
question (non-blocking,ux): Should the Docs line keep its own trailing newline when a Tip row is added after it?

Right now Docs omits `\n` when a tip is present, and Tip adds `\n` at the end of its value — so blank lines between footer rows depend on pairs of fields rather than a single spacer. That works for Docs → Tip → ID today; would empty spacer entries in `body` be easier if more optional rows are added later?
```

**Lint suppression (what / why / required / alternatives):**

```text
question (non-blocking): Was the `gocyclo` nolint the intended fix for the new tip branches in `RenderError`?

The tip logic adds several `if` branches and pushes cyclomatic complexity over the repo’s min-complexity threshold (15), so lint fails without the suppression. It’s not needed for runtime correctness — extracting e.g. `readErrorTip(ctx)` would also bring complexity back under the limit. Fine to keep the nolint if you prefer the flat layout; I wanted to confirm that was deliberate.
```

## Anti-patterns

- ❌ `This is wrong.` → ✅ `question: Could this throw when the slice is empty?`
- ❌ Stacking five decorations → ✅ one label + zero to two decorations
- ❌ Separate comments for the same root cause → ✅ one collapsed comment with multiple anchors in discussion
- ❌ False praise → ✅ omit `praise` when nothing is genuinely notable
- ❌ Theme: “newline coupling / maintainability as fields grow” → ✅ Theme: “Docs drops trailing newline when Tip follows”
- ❌ Subject assumes reader has diff open → ✅ Subject states observable behavior first
- ❌ `nolint` mentioned only in findings, no what/why/required → ✅ comment or Change context explains rule, trigger, and alternatives
- ❌ Abstract pattern name before mechanism → ✅ what the code does, then the concern
