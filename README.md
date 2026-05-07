# foxworks-cairn

Cairn methodology + agents + skills for Claude Code, used across Foxworks projects.

## Overview

`foxworks-cairn` is a Claude Code plugin that codifies the **cairn methodology** — a discipline for software-engineering work with Claude Code that prevents fabrication, drift, and unaccountable claims. The methodology was born in the [foxworks-dispatch](https://github.com/JoshTSeppich/foxworks-dispatch) project (private) and is generalized here so any Foxworks (or Foxworks-adjacent) repo can adopt it cleanly.

The plugin ships:

- **5 cairn-discipline agents** that main Claude Code dispatches for fabrication-verification, Phase 1 diagnose, WB14 followup drafting, test-failure triage, and cross-package impact analysis.
- **2 cairn methodology skills** that auto-load on methodology + foxworks-dispatch-codebase questions.

The discipline is operator-grade: every claim carries a confidence label, every cairn-grammar commit answers a self-check Q1-Q9, every halt is literal, every push verifies before proceeding.

## Installation

### Path A — Local development install (v0.1.0 default)

```bash
cc --plugin-dir /Users/joshuatseppich/Desktop/Automata/foxworks-tooling
```

This loads the plugin from the local clone without going through a marketplace. If `cc` is not on PATH, use the full path to your Claude Code binary. After install, restart CC if it was already running.

### Path B — Marketplace install (deferred to v0.2)

Standard marketplace install (`/plugin install <name>@<marketplace>`) requires the source repo to be structured as a *marketplace* (multiple plugins under `plugins/<name>/` subdirectories). `foxworks-tooling` is structured as a single plugin and is not a marketplace. Restructuring is deferred to v0.2 — see "Roadmap" below.

### Verifying install

After install, verify by running a sample prompt from the agent or skill inventories below. If the plugin loaded, CC will dispatch the named agent or cite the named skill content. Full verification harness at [`validation/MANUAL_TESTS.md`](validation/MANUAL_TESTS.md).

## Agents (5)

| Name | Color | Tools | Purpose |
|------|-------|-------|---------|
| [`cairn-anti-fabrication-verifier`](agents/cairn-anti-fabrication-verifier.md) | cyan | Read, Grep, Glob | Verify factual claims about a codebase before main CC asserts them as KNOWN. |
| [`cairn-phase-1-diagnose`](agents/cairn-phase-1-diagnose.md) | blue | Read, Grep, Glob | Run a Phase 1 diagnose for a ticket spec — surface inventory + arbitration questions + risks. |
| [`cairn-followup-drafter`](agents/cairn-followup-drafter.md) | green | Read, Grep, Bash | Draft `MB-F-*` followup entries at WB14 for `FOLLOWUPS.md`. |
| [`cairn-test-failure-triage`](agents/cairn-test-failure-triage.md) | yellow | Read, Bash, Grep | Stash + isolated re-run to determine if test failures are pre-existing or caused by current work. |
| [`cairn-cross-package-impact`](agents/cairn-cross-package-impact.md) | magenta | Read, Grep, Glob | Grep the import graph for shared schema/contract changes; classify risk + frozen-contract status. |

### `cairn-anti-fabrication-verifier`

**Trigger phrases:** "verify whether X exists", "confirm the signature of Y", "check if Z is pre-existing", "audit this paragraph of claims about file F"

Read-only verifier. Returns a structured verdict block with `[KNOWN]/[MODELED]/[SPECULATIVE]` confidence labels and `file_path:line` citations. Multi-claim dispatches return one block per claim.

### `cairn-phase-1-diagnose`

**Trigger phrases:** "run Phase 1 diagnose for ticket X", "diagnose MB-TXX surfaces", "Phase 1 surface inventory", "what arbitrations does ticket Y need"

Reads a ticket spec, surveys the relevant codebase surfaces, and returns a structured markdown diagnose document — surface inventory, arbitration questions, risks, acceptance criteria, HALT 0 gate. Repo-agnostic; works against whatever repo the dispatching CC is in.

### `cairn-followup-drafter`

**Trigger phrases:** "draft followups for MB-TXX", "WB14 followup list", "compose followup entries", "what should land in FOLLOWUPS.md from this ticket"

Reads recent commit bodies + WB findings via read-only `git log` / `git show`, then drafts table-row entries in the `FOLLOWUPS.md` three-column format (`| ID | Scope | Origin |`) with `[MODELED]` Tier 1/2/3 proposals for operator review. Does not commit.

### `cairn-test-failure-triage`

**Trigger phrases:** "triage these test failures", "are these failures pre-existing", "isolate failure cause", "did my change break this test"

Stashes uncommitted work via `git stash push -u`, runs tests against the resulting clean state, restores the stash via `git stash pop`, then returns a failure-attribution table classifying each failure as pre-existing, caused-by-dispatcher, or unverifiable. Stash ref is always surfaced as the first line of output for manual recovery if anything goes wrong. Bash safe-list strictly enforced in the agent's prompt.

### `cairn-cross-package-impact`

**Trigger phrases:** "impact analysis for changing X", "who imports from Y", "cross-package effect of changing Z", "is this change touching a frozen contract"

Globs for package manifests, greps the import graph across all packages, classifies impact (type-only / behavior-changing / runtime-critical / frozen-contract-violation), checks for parallel-cairn coordination overlap, and returns a structured impact report with reproducible search trace.

## Skills (2)

| Name | Purpose |
|------|---------|
| [`cairn-methodology`](skills/cairn-methodology/SKILL.md) | Documents the ten cairn disciplines (anti-fabrication, confidence labels, cairn grammar, self-check Q1-Q9, plan-mode discipline, halt discipline, per-commit-push, per-path git add, frozen contracts, outcome classifications). Repo-agnostic. |
| [`foxworks-conductor-codebase`](skills/foxworks-conductor-codebase/SKILL.md) | Orientation for the foxworks-dispatch monorepo (5 packages, flat directory convention, sentinel zones, persistence pattern, dispatch-core build discipline, 4 frozen contracts). Repo-specific. |

### `cairn-methodology`

**Trigger phrases:** "cairn methodology", "cairn discipline", "anti-fabrication", "halt discipline", "cairn grammar", "self-check Q1-Q9", "confidence labels", "per-commit-push", "per-path git add", "plan-mode discipline", "frozen contracts"

Lean SKILL.md (~1,500 words) covering the ten disciplines with citations to `foxworks-dispatch/CLAUDE.md §2` + `CONDUCTOR_API_CONTRACT.md §10`. References:

- `references/anti-fabrication.md` — patterns + worked examples.
- `references/halt-discipline.md` — HALT 0/1/2 + mid-WB + plan-mode halts.
- `references/self-check-template.md` — Q1-Q9 fill-in template adaptable per project.

### `foxworks-conductor-codebase`

**Trigger phrases:** "foxworks-dispatch", "Conductor v3", "dispatch-core", "dispatch-workstation", "dispatch-daemon", "dispatch-cli", "dispatch-web", "tile-grid", "splitter-state", "schema spine", "sentinel zones", "post-pull rebuild", "frozen contracts"

11 codebase facts: repo + version model, package layout (5 packages), flat directory convention, sentinel zones in main.ts, persistence pattern, dispatch-core dist build discipline, runtime smoke gate, 4 frozen contracts, schema spine authority, coordination doc convention, tile-grid + splitter-state UI subsystems. References:

- `references/sentinel-zones-inventory.md` — current sentinel zones + refresh procedure.
- `references/frozen-contracts.md` — per-surface detail for the 4 operator-arbitrated surfaces.

## Cairn methodology in one paragraph

Cairn is a discipline that treats every factual claim as auditable: claims carry `[KNOWN]/[MODELED]/[SPECULATIVE]` confidence labels; commits use a five-verb grammar (`red:`, `green:`, `spike:`, `contract:`, `refactor:`) with mandatory Q1-Q9 self-check bodies; halts mean "literally nothing happens" until operator ack; pushes verify against `origin/main` before proceeding; frozen contracts are operator-arbitrated only. The discipline emerged from foxworks-dispatch as a defense against AI-assistant fabrication and scope drift. For full detail, query the `cairn-methodology` skill or read [`skills/cairn-methodology/SKILL.md`](skills/cairn-methodology/SKILL.md).

## Validation

The plugin ships with a manual test plan at [`validation/MANUAL_TESTS.md`](validation/MANUAL_TESTS.md) — runbook for verifying agent dispatch + skill load post-install. Each agent + skill has a sample prompt, expected behavior, and Yes/No verification questions. Failed tests get filed as `MB-F-*` followups in `FOLLOWUPS.md`.

## Roadmap

- **v0.1.0** (this release): 5 agents + 2 skills + validation harness. Local-install only via `--plugin-dir`.
- **v0.2** (planned): marketplace restructure + `foxworks-cairn` listing on a Foxworks plugin marketplace; install verification command improvements; cairn-methodology skill formalization of plan-mode discipline + tier criteria.

## Repository

https://github.com/JoshTSeppich/foxworks-tooling (private)

## Author

Joshua Seppich — josh@aetherx.io
Foxworks Technologies, Ogden UT

## Version

0.1.0 — Initial release with five cairn-discipline agents and two cairn methodology skills.

## License

MIT License — see [LICENSE](LICENSE) for full text.
