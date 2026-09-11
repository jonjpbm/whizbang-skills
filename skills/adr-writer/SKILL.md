---
name: adr-writer
description: Use whenever you are about to add a new library, service, or dependency, introduce a pattern other code will need to follow, pick between two or more real options for something structural (a database tier, a state layout, a monitoring approach), or make any other call that would be expensive to reverse — stop and propose an Architecture Decision Record (ADR) before acting on it, even if the user never says the word "ADR" or asks for documentation. Also use whenever the user directly asks to write, draft, update, or supersede an ADR, or asks "should this be an ADR." Covers the should-this-be-an-ADR gate (so trivial or already-decided tweaks don't get flagged), detecting an existing repo's ADR convention, matching its format exactly, and recording supersession.
---

# ADR writer

An Architecture Decision Record (ADR) is a short, permanent note. It answers
one question: why did we choose this, over what alternatives, at this point
in time. Its job is to save the next reader — often a future you — from
re-litigating a decision or reversing it by accident.

## 1. Gate: should this even be an ADR?

Most decisions are not ADR material. Before proposing one, check that the
decision fails **all** of these skip conditions:

- **Tiny** — reversible in minutes, no one else is affected.
- **Self-contained** — nothing outside this one file or function depends on
  the choice.
- **Single-developer, no handover** — only relevant if someone else, or
  future-you in six months, needs the reasoning.
- **Already covered by policy** — a style guide, a linter rule, or an earlier
  ADR already settled this.
- **Temporary** — a workaround you expect to remove, not a standing decision.

If the decision fails to clear even one of the skip conditions, it likely
deserves a record. If you are unsure, ask the user in one sentence rather
than guessing.

For example, a config tweak that an earlier ADR already named as expected,
routine work (say, that ADR explicitly calls out threshold or parameter
tuning as something to adjust freely) is already covered by policy — it does
not need an ADR of its own. Swapping in a new dependency or picking between
two real infrastructure options, by contrast, usually clears every skip
condition: something else depends on it, it is not obviously reversible, and
nothing existing has already decided it.

## 2. Trigger: propose before, not after

Stop and propose an ADR at the moment a decision is being made, not after
the fact, when you are about to:

- Add a new dependency (package, service, external API).
- Introduce a pattern other code will need to follow (a new module
  boundary, a new state-management approach, a new naming or layout
  convention).
- Make a call that is expensive to reverse (a schema choice, a pricing
  tier, a data-boundary decision, anything that would require a rebuild or
  a migration to undo).

Proposing means: name the decision in one or two sentences, say an ADR
should record it, and ask whether to write one now — don't silently write
code that embodies an unrecorded decision.

## 3. Detect the repo's ADR convention

Never assume a template. Check what a repo already does before writing
anything:

```
git ls-files | grep -iE '(^|/)(adr|adrs|decisions?)(/|$)'
```

This finds an existing ADR directory (`docs/adr/`, `docs/decisions/`,
`adr/`, etc.) and its numbering scheme. If files exist, open the two or
three most recent ones and copy their exact structure — headers, field
labels, capitalization, section order — rather than reconstructing it from
memory or from a generic idea of what an ADR looks like. Small format
inconsistencies between ADRs in the same repo are a paper cut for every
future reader.

If the grep finds nothing, there is no convention yet: ask the user where
ADRs should live, or default to the template below.

## 4. Default template (when a repo has no convention yet)

`docs/decisions/`, one file per decision, 4-digit sequential numbers
starting at `0001`:

```
# NNNN — Title

**Status:** Accepted
**Date:** YYYY-MM-DD

## Context

## Decision

## Rationale

## Consequences

## References        (optional — include only when there are real sources to cite)
```

Field notes:

- **Status** starts as `Accepted` unless the user wants a `Proposed` review
  stage first — ask rather than assume.
- **Date** is the day the decision was made, not the day the file was
  written.
- **Context** states the problem and any constraints — enough that a
  reader with no memory of the discussion understands what was being
  decided and why it mattered.
- **Decision** is the outcome, stated plainly, in one or two sentences,
  bolded if it is a specific value (a tier, a version, a boundary).
- **Rationale** is why this option won over the alternatives — cite real
  numbers, prices, or docs where they exist, not vibes.
- **Consequences** are the trade-offs accepted, including any that are
  inconvenient. A decision with no listed downside was not examined
  closely enough.
- **References** is optional — only add it when there are real links or
  sources to cite, not as a placeholder.

Do not add sections beyond this list (no Implementation Plan, no
Verification checklist, no ID-coded bullets) unless the user asks for one,
or an existing convention in the repo already uses them. A new required
section imposed on only the newest ADR breaks consistency with the rest.

## 5. When a decision has several real alternatives

If a decision has three or more genuinely live options each worth a
pros/cons comparison, the plain prose Rationale section can get dense.
MADR's worked examples show one way to lay that out as a table:
https://adr.github.io/madr/examples.html

Use this only as a secondary structuring option inside the existing
Rationale section (a table of options) — never as a replacement top-level
template.

## 6. Supersession

When a new decision replaces an old one:

1. Write the new ADR normally, with the next sequential number.
2. In the new ADR's Context or Rationale, name the ADR it supersedes and
   say why (e.g. "Supersedes ADR 0004: the constraint that decision was
   built around no longer holds").
3. Go back to the old ADR and update its `**Status:**` line to point
   forward, e.g. `**Status:** Superseded by ADR 0012`. Leave the rest of
   the old ADR unchanged — it is a historical record, not something to
   rewrite.
4. If the repo keeps a summary table of decisions (often in `AGENTS.md`,
   `CLAUDE.md`, or the README), update the row to point at the new ADR —
   the table is a pointer, the ADR holds the reasoning.

Never delete or silently edit an old ADR's Decision or Rationale to match
the new choice — that erases the history the whole record exists to
preserve.
