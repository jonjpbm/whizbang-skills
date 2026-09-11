---
name: adr-writer
description: Use whenever you are about to add a new library, service, or dependency, introduce a pattern other code will need to follow, pick between two or more real options for something structural (a database tier, a state layout, a monitoring approach), or make any other call that would be expensive to reverse — stop and propose an Architecture Decision Record (ADR) before acting on it, even if the user never says the word "ADR" or asks for documentation. Also use whenever the user directly asks to write, draft, update, or supersede an ADR, or asks "should this be an ADR."
---

# ADR writer

An ADR is a short, permanent note answering one question: why did we choose
this, over what alternatives, at this point in time — so nobody re-litigates
or reverses it by accident.

## 1. Should this even be an ADR?

Skip it only if the decision is **all** of: tiny (reversible in minutes),
self-contained (nothing else depends on it), single-developer with no
handover need, already covered by policy or an earlier ADR, or temporary. If
even one of those doesn't hold, write it. Unsure? Ask in one sentence rather
than guess.

Example: bumping a config value an earlier ADR already named as free to
tune — skip. Adding a new dependency, or picking between two real infra
options — usually doesn't skip: something depends on it, it isn't obviously
reversible, and nothing has decided it yet.

## 2. Propose before, not after

Stop and propose an ADR the moment you're about to add a dependency,
introduce a pattern other code will follow, or make a call that's expensive
to reverse (a schema, a pricing tier, a data boundary — anything needing a
migration to undo). Name the decision in a sentence, say it should be an
ADR, and ask — don't silently write code that embodies an unrecorded
decision.

## 3. Match the repo's convention

Check first, don't assume a template:

```
git ls-files | grep -iE '(^|/)(adr|adrs|decisions?)(/|$)'
```

If ADRs exist, open the two or three most recent and copy their exact
structure — headers, field labels, section order — rather than
reconstructing one from memory. Format drift between ADRs in the same repo
is a paper cut for every future reader. Nothing found? Ask where they
should live, or use the default template below.

## 4. Default template (no existing convention)

`docs/decisions/`, one file per decision, 4-digit sequential numbers from
`0001`:

```
# NNNN — Title

**Status:** Accepted
**Date:** YYYY-MM-DD

## Context
## Decision
## Rationale
## Consequences
## References        (optional — only when there are real sources to cite)
```

- **Status** defaults to `Accepted` unless the user wants a `Proposed`
  review stage — ask, don't assume.
- **Date** is when the decision was made, not when the file was written.
- **Context** gives enough for a reader with no memory of the discussion to
  understand what was decided and why it mattered.
- **Decision** states the outcome plainly, in one or two sentences.
- **Rationale** cites real numbers, prices, or docs where they exist — not
  vibes.
- **Consequences** names the real trade-offs, including inconvenient ones —
  a decision with no listed downside wasn't examined closely enough.

Don't add sections beyond this list (no Implementation Plan, no ID-coded
bullets) unless the user asks, or an existing convention already uses them.

## 5. Several real alternatives?

Three or more live options each worth a pros/cons comparison: use MADR's
table layout inside the existing Rationale section, not as a replacement
top-level template. Examples: https://adr.github.io/madr/examples.html

## 6. Supersession

Write the new ADR with the next number; in its Context or Rationale, name
what it supersedes and why. Then go back and update the old ADR's
`**Status:**` line to point forward (`Superseded by ADR 0012`) — leave the
rest of it untouched, it's a historical record, not something to rewrite.
If the repo keeps a summary table of decisions (`AGENTS.md`, `CLAUDE.md`,
README), update its row to point at the new ADR. Never edit an old ADR's
Decision or Rationale to match a later choice — that erases the history the
record exists to preserve.
