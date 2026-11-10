# whizbang-skills

Personal collection of agent skills (`SKILL.md`), for reuse across projects and
across coding agents that support the skills convention (Claude Code, and
others via `.agents/skills/`-style vendoring).

## Skills

- [`adr-writer`](skills/adr-writer/SKILL.md) — propose and write Architecture
  Decision Records (ADRs) at the moment a decision is made, matching whatever
  ADR convention a repo already has (or a sane default if it has none).
- [`memory-dream`](skills/memory-dream/SKILL.md) — a manual "dream" pass for
  [ai-memory](https://github.com/akitaonrails/ai-memory): merge overlapping
  memory pages with your own model, after you approve the plan, when ai-memory
  has no server-side LLM provider.
