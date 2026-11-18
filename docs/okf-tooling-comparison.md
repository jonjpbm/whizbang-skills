# OKF tooling comparison

Analysis date: 2026-11-10. Written: 2026-11-18. Source: a Kiro CLI session (id `sess_4eb07fc4-0de5-4ecd-9049-817af12d47d1`, workspace `/Users/jduarte`).

## Verdict

Use `scaccogatto/okf-skills` for authoring OKF bundles with a coding agent. It is the only one of the four requested repos that is current, tested, maintained, and passes Google's own example bundles. You already have its four skills installed in `~/.kiro/skills/` (`okf`, `validate`, `visualize`, `backfill`), byte-identical to plugin version 0.12.0.

Do not adopt `mattjoyce/okf-skill` or `rakibtg/okf-skill`. Both teach the superseded v0.1 spec. rakibtg's validator rejects every official example bundle.

`xSAVIKx/okf-skills` is a different kind of tool. It generates bundles from databases and APIs. Use it for first drafts only. Do not re-run it over bundles that people have verified.

## The question

Compare four GitHub repos that teach AI coding agents the Open Knowledge Format:

- https://github.com/scaccogatto/okf-skills
- https://github.com/mattjoyce/okf-skill
- https://github.com/xSAVIKx/okf-skills
- https://github.com/rakibtg/okf-skill

Also find other candidates. Confirm every theory with evidence. Do not assume.

## What OKF is

OKF is the Open Knowledge Format. Google Cloud published it in June 2026. It stores knowledge as markdown files with a YAML frontmatter header. People and AI agents read the same files. A folder of such files is a bundle. One file is a concept.

The canonical repo is `GoogleCloudPlatform/open-knowledge-format` (1,426 stars). `SPEC.md` sits at its root. The older path `GoogleCloudPlatform/knowledge-catalog/okf` is a frozen snapshot.

Conformance (SPEC §11) has three rules:

1. Every concept file has parseable YAML frontmatter.
2. Every frontmatter has a non-empty `type`.
3. `index.md` and `log.md` follow §8 and §9 when present.

Everything else is optional: `okf_version`, `title`, `description`, `index.md`, unbroken links. A validator that fails a bundle for one of these is stricter than the spec.

## Spec timeline

Confirmed from git history of `GoogleCloudPlatform/knowledge-catalog` and `open-knowledge-format`.

| Date | Event |
|---|---|
| 2026-06-12 | v0.1 published. Fields: `timestamp`, body heading `# Citations`. |
| 2026-07-24 | v0.2 published. `timestamp` became `generated: {by, at}`. `# Citations` became frontmatter `sources`. Added `verified`, `status`, `stale_after`, `Attested Computation`. |
| 2026-08-14 | Dedicated repo `open-knowledge-format` created. |
| 2026-08-21 | Every timestamp must be an ISO 8601 datetime with an explicit offset. |

Two of the four repos stopped before 2026-07-24. That is the main fault line.

## The four repos

| | scaccogatto/okf-skills | xSAVIKx/okf-skills | mattjoyce/okf-skill | rakibtg/okf-skill |
|---|---|---|---|---|
| What it is | Authoring toolchain: plugin, 4 skills, GitHub Action, MCP server | Go connectors that produce OKF from DBs, APIs, filesystem, git | One authoring skill + validator | One authoring skill + 5 scripts |
| Stars / forks | 448 / 42 | 38 / 2 | 3 / 0 | 6 / 1 |
| Commits / authors | 104 / 8 | 134 / 3 | 3 / 1 | 20 / 1 |
| First / last push | 2026-06-14 / 2026-11-04 | 2026-06-13 / 2026-10-03 | 2026-06-18 / 2026-06-18 | 2026-07-05 / 2026-08-16 |
| Spec version taught | v0.2 | v0.2 (vendored copy predates 2026-08-21) | v0.1 | v0.1 (vendors v0.1 spec) |
| License | MIT | Apache-2.0 | Apache-2.0 | Apache-2.0 |
| Language | Python (`uv` + `pyyaml`) | Go 1.24 | Python stdlib (`pyyaml` optional) | Python stdlib |
| Install | Claude plugin, `npx skills add`, Action `@v1`, MCP | `npx skills add`, `go install`, MCP host | Claude marketplace | `npx skills add`, copy folder |
| Tests (run locally) | 152 pass, 18 fail (hook, macOS only) | all 14 Go modules pass | none | none |
| CI | yes (ubuntu + windows) | yes | no | no |
| Open issues | 6 | 0 | 0 | 0 |
| Clone size | 13 MB | 145 MB (47 MB are committed Linux binaries) | 228 KB | 276 KB |

## Validators against Google's four official bundles

Each validator ran on `bundles/{acme_retail,ga4,stackoverflow,crypto_bitcoin}` from the official repo.

| Validator | acme_retail | ga4 | stackoverflow | crypto_bitcoin | Catches no frontmatter / no `type` |
|---|---|---|---|---|---|
| scaccogatto `okf_validate.py` | conformant (5 warnings) | conformant | conformant (11 warnings) | conformant | yes / yes |
| mattjoyce `validate_okf.py` | conformant | conformant | conformant | conformant | yes / yes |
| rakibtg `validate.py` | NOT CONFORMANT | NOT CONFORMANT | NOT CONFORMANT | NOT CONFORMANT | yes / yes |
| xSAVIKx `okf-lint` | FAIL | FAIL | FAIL | FAIL | yes / yes |

Root causes:

- rakibtg: a hand-written YAML parser (`scripts/_okf_common.py:53-108`) crashes on nested mappings. Every official bundle uses `generated:\n  by: ...`. PyYAML parses the same text without error. The validator also fails `acme_retail/log.md` for having frontmatter. The spec never forbids that.
- xSAVIKx: `okf-go/lint.go:224` requires `okf_version` in the root `index.md`. The spec says MAY. The default `--max-broken-links` is 0. The spec says consumers MUST tolerate broken links. The `stackoverflow` bundle writes `tags: a, b` as a string. `okf-go` types `tags` as `[]string`, so it cannot parse that file.

## Findings per repo

### scaccogatto/okf-skills

Verified strengths:

- The validator reads §11 literally. Errors and warnings are separate. It passes all four official bundles.
- `--migrate` rewrites v0.1 `timestamp` to `generated` and `# Citations` to `sources`. Confirmed on mattjoyce's example bundle.
- The vendored `SPEC.md` is verbatim. Only a 10-line attribution header differs.
- MCP server: `read_concept` and `get_neighbors` resolve ids with `resolve()` plus `is_relative_to(root)` (`servers/okf_mcp.py:115-125`). Path traversal and absolute paths are blocked.
- Supply chain is tested: SRI hashes on CDN scripts, upper-bounded PEP 723 dependencies, SHA-pinned `setup-uv`.
- Three benchmarks are pre-registered. The trust benchmark declares its own primary result invalid.
- The repo documents itself in OKF under `.okf/`, with decision records.

Verified defects:

- Stop hook: `hooks/okf-stop-check.sh:90` fails to parse on macOS `/bin/bash` 3.2. Cause: `case ... in "$root"/*) ... esac` inside `$(...)`. Only `recursive:` mode is affected. CI runs on Ubuntu, so CI is green.
- Validator crashes: `status: [draft, stable]` raises `TypeError` (open issue #62, reproduced). An impossible date raises `ValueError` (#63).
- `search_concepts` reads every `*.md` with `rglob` and no containment check. A committed symlink `x.md -> /etc/passwd` becomes a substring oracle.
- Backfill reads `~/.claude/projects/<slug>/*.jsonl` session transcripts. There is no redaction step. `events.jsonl` is not in `.gitignore`.
- Spec deviations: the backfill bootstrap writes `okf_version` with no `---` fences (`skills/backfill/SKILL.md:74`). The root `index.md` gets plugin keys `upkeep` and `recursive`. §12 permits only `okf_version` there.
- Provenance drift: the weaver stamps `generated.by: okf-backfill/0.9.5`. The plugin is 0.12.0.
- Plugin policy reads as spec in places: lifecycle-only `log.md`, mandatory `status`, a closed `type` list in the weaver.
- No lockfile. `uv run` resolves `pyyaml` at every run. `@v1` is a floating tag.

### xSAVIKx/okf-skills

Not an authoring skill. A set of Go connectors for SQLite, MySQL, PostgreSQL, BigQuery, MongoDB, CSV, OpenAPI, GraphQL, filesystem and git. `okf-mcp` exposes any `okf-*` binary on `PATH` as MCP tools. Three instruction-only skills ride along: `okf-reader`, `okf-enrich`, `okf-producer-generator`.

Verified strengths:

- All 14 Go modules pass `go test`. 173 test functions.
- Deterministic extraction. A re-run on an unchanged source is a no-op.
- Release automation with release-please. Each module is tagged on its own.
- Trust tier derivation (`GetTrustTier`) follows §5.3.

Verified defects:

- Six Linux x86-64 ELF binaries are committed under `skills/*/` (about 47 MB).
- `okf-go` drops unknown frontmatter keys on a read/write round-trip. Test: `owner_team: payments` and a nested custom key vanished. SPEC §4.1 says consumers SHOULD preserve unknown keys.
- `IsStale` parses `stale_after` only as `YYYY-MM-DD` (`okf-go/okf.go:136`). With the spec format `2026-09-23T00:00:00Z` it returns `false`. Stale content looks fresh.
- A re-run destroys trust metadata. Test: add `verified: human:jon`, `status: deprecated`, `stale_after`, `sources` and a custom key to a produced concept, then change the source file. `okf-fs produce` rebuilt the concept and deleted all five. Cause: `MergeConcept` hashes only the body and carries over only `description`, `tags` and `enriched_against` (`okf-go/hash.go:56-77`).
- `okf-lint` rejects every official bundle (see table above).
- Connectors write `stale_after` and `usage_window` as bare dates, against the 2026-08-21 rule.
- PostgreSQL `ingest --sync` builds SQL with `fmt.Sprintf("COMMENT ON TABLE %s.%s IS '%s'", ...)` (`main.go:368`). Schema and table names are not quoted. `sslmode=disable` is hard-coded. The password goes into a DSN string from a `--password` flag.
- The G-Core fork `noplannomercy/okf-skills-fork` found the round-trip, staleness and merge defects on its own. Its `boundary_test.go` has 16 tests. Against upstream HEAD, 1 does not compile and 8 of the remaining 15 fail.

### mattjoyce/okf-skill

One skill, one validator, one example bundle. Three commits on one day. No activity since 2026-06-18.

Verified strengths:

- The validator gives correct §11 verdicts on all four official bundles. It never blocks a valid bundle.
- It states that it does not check `index.md` or `log.md` structure.
- The link audit is informational only, as the spec requires.
- The concept-id slug regex matches the reference tooling exactly (`paths.py:20`).

Verified defects:

- Teaches v0.1 only. `timestamp`, `# Citations`, `okf_version: "0.1"`, v0.1 section numbers throughout. No `generated`, `verified`, `sources`, `status` or `stale_after`.
- `SKILL.md:126-131` tells the agent to write `okf_version` as a bare line with no `---` fences. Both spec versions require a frontmatter block.
- It recommends `--strict`. That mode fails every concept without `timestamp`, which is every conformant v0.2 concept.
- The slug regex is reported as a §9 conformance ERROR. The spec has no such rule.
- `references/tooling.md` names `enrichment_agent` and `kcmd`. The current module is `reference_agent`. `kcmd` lives in another repo.
- The fallback parser (no PyYAML) treats `type: ""` as a non-empty string.

### rakibtg/okf-skill

One skill, five stdlib scripts (init, new concept, index, log, validate), templates, a vendored v0.1 spec. Last commit 2026-08-16, three weeks after v0.2 shipped.

Verified strengths:

- The only one of the two small repos that checks §11 rule 3 (`index.md` and `log.md` shape).
- `add_log_entry.py` produces the correct `## YYYY-MM-DD`, newest-first shape.
- Scripts refuse to overwrite without `--force`.
- Good agent hygiene: "never fabricate `resource` URIs or history".
- The Agent Skills frontmatter is standard: `name`, `description`, `license`, `compatibility`, `metadata`.

Verified defects:

- The hand-written YAML parser cannot read nested mappings, lists of mappings, multi-line scalars or comments. Every v0.2 spec example and every official bundle uses at least one of these.
- `gen_index.py` uses the same parser. On a v0.2 bundle it silently omits every concept it cannot parse.
- `validate.py` fails `log.md` for having frontmatter. The spec never forbids it. The official `acme_retail/log.md` has `type: Log`.
- `SKILL.md:204-206` tells the agent to trust `validate.py` over its own reading. It says the script "implements §9 exactly". It does not.
- `init_bundle.py` writes `okf_version: "0.1"`. Templates write `timestamp` and `# Citations`.

## Other repos found

A GitHub search over ten query variants returned 163 unique repos. Eleven more were cloned and inspected.

### Direct alternatives: single-purpose authoring skills

| Repo | Stars | Spec | Validator | Tests / CI | Verdict |
|---|---|---|---|---|---|
| lorsabyan/okf-skill | 6 | v0.2 | Python stdlib, lenient | 152 / yes, plus an upstream-spec drift check | Closest v0.2 like-for-like. Defect: forbids the `/`-rooted links that §6.1 recommends. |
| sniperunder123/okf-knowledge | 66 | v0.2 | Python + PyYAML; `--strict` turns optional items into failures, and SKILL.md always says `--strict` | 78 / no | Popular. Two commits. No CI. |
| win4r/okf-skill | 2 | v0.1 | Python stdlib, hand-rolled parser with a PyYAML equivalence test | 92 functions, about 137 pytest cases / yes | Good negative corpus (E001 to E008). Stuck on v0.1. |
| catancs/okf-skill | 8 | v0.1 | none (prompt only) | 0 / no | Six files. |
| theesfeld/claude-okf | 4 | v0.1 | none; a Sonnet auditor subagent | 0 / yes | A SessionStart hook injects a "PRIME DIRECTIVE" into every session. The skill fetches the live spec and says "the live spec wins". |

### Different category

| Repo | Stars | What it is | Note |
|---|---|---|---|
| okf-memory/okf-agent-memory | 864 | Go agent-memory system: CLI, MCP, hub server | v0.2. Lenient validator. `okf pull` calls `registry.okf-memory.dev`. |
| serradura/okf | 184 | Ruby: four gems, TUI, MCP, Docker, Claude plugin | v0.2. Full authoring skill inside a large toolchain. A PostToolUse hook shells out. 13 open issues. |
| UmairBaig8/okf-generator | 109 | Python tree-sitter code scanner | Labels itself v0.2 but emits `timestamp`, never `generated`. No validator. Optional OpenAI calls. |
| hamakyo/okf-skills | 2 | Five engineering-workflow skills; `update-okf` is one | v0.2. Its validator separates spec faults from profile rules. Worth borrowing. |
| noplannomercy/okf-skills-fork | 0 | G-Core fork of xSAVIKx | Patches the `okf-go` defects. Removes committed binaries. FORK.md is in Korean. |

Seen in search but not cloned: `0dust/OKFy` (74), `openknowledge-sh/openknowledge` (74), `jyjeanne/okf-rs` (108), `okfcli/okf` (31), `W4G1/okf` (31), `zosmaai/pi-llm-wiki` (627, pi agent), `coleam00/cole-medin-knowledge-base` (154, a bundle, not a tool), two `awesome-okf` lists, and `aws-samples/sample-okf-llm-wiki` (cited by scaccogatto for a BIRD text-to-SQL score of 74.0).

## Recommendations

1. Keep `scaccogatto/okf-skills` as the authoring toolchain. It is already installed under `~/.kiro/skills/`.
2. WARNING: Do not enable `upkeep: enforced` with `recursive:` on macOS until the bash 3.2 bug is fixed.
3. WARNING: Do not run `/okf:backfill` on a repo whose Claude session history contains secrets or private text.
4. If you use the GitHub Action, pin `@okf--v0.12.0`, not `@v1`.
5. For a dependency-free skill in Codex or another harness, use `lorsabyan/okf-skill`. Override its "never start a link with `/`" rule.
6. For bundles generated from a database or an OpenAPI file, use `xSAVIKx/okf-skills` for the first draft only. Prefer the G-Core fork's `okf-go` patches, or wait for upstream to adopt them.

## Method

1. Confirm the meaning of "okf" by web search and GitHub API.
2. Find the canonical spec repo. Pull the v0.1 and v0.2 spec texts and their commit dates.
3. Search GitHub with ten query variants. Deduplicate to 163 repos.
4. Clone the four named repos, the spec repo, and eleven candidates into `/Users/jduarte/tmp/okf-compare/`.
5. Collect git and GitHub metadata per repo: commits, authors, dates, license, tests, CI, open issues.
6. Run every validator against the four official bundles and against hand-made broken files.
7. Build `okf-lint` and `okf-fs` from source. Write small Go tests for round-trip, staleness and merge behavior.
8. Run each repo's own test suite. Diagnose failures to root cause.
9. Dispatch read-only subagents for line-by-line SKILL.md audits and the eleven-repo survey. Spot-check their claims against source lines.
10. Diff vendored spec copies against the official text.

## Not verified

- Whether `npx skills add` resolves each repo on skills.sh. No network test was run.
- Whether the `mattjoyce/claude-plugins` marketplace lists the `okf` plugin.
- The scaccogatto benchmark numbers. RESULTS.md was read, not re-run.
- Exact test counts for serradura/okf and okf-generator. Tool output caps gave lower bounds only.
- Any of the 163 repos that were not cloned.

## Where the evidence lives

- Clones, fixtures and the original report: `/Users/jduarte/tmp/okf-compare/` (scratch; may be deleted).
- Kiro session: `kiro-cli chat --resume-id sess_4eb07fc4-0de5-4ecd-9049-817af12d47d1`, run from `/Users/jduarte`.
- Kiro session storage: `~/.kiro/sessions/0d5f9fc9f08c0286/sess_4eb07fc4-0de5-4ecd-9049-817af12d47d1/` (transcript, seven subagent transcripts, a snapshot of the report).
