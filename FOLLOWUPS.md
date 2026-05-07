# foxworks-tooling — Followups

`MB-F-*` followup entries surfaced during plugin validation and future maintenance work. Mirrors the foxworks-dispatch `docs/FOLLOWUPS.md` 3-column table convention.

## Validation log

P7 validation run on 2026-05-07 by a fresh CC session pointed at foxworks-dispatch. Per `validation/MANUAL_TESTS.md`.

| Test | Result | Notes |
|------|--------|-------|
| §2.1 cairn-anti-fabrication-verifier | PASS | Correctly refuted fabricated claim; surfaced adjacent candidate files. |
| §2.2 cairn-phase-1-diagnose | PASS | All 7 sections present (Source + 6 spec sections); cited `main.ts:462-499`, `splitter-state.ts:1-35`, `spawn-handler.ts:190-199`; self-classified as "second-pass diagnose" given MB-T12 ship status. |
| §2.3 cairn-followup-drafter | PASS | 3 entries in correct table-row format with explicit `[MODELED]` tier labels; honored read-only discipline. |
| §2.4 cairn-test-failure-triage | PASS (cosmetic deviation) | Functional behavior fully correct: stash push/pop clean, attribution table accurate, no forbidden commands (verified via reflog + stash list). One spec deviation: stash ref was second block, not first line — see `MB-F-A4-STASH-REF-NOT-FIRST-LINE` below. |
| §2.5 cairn-cross-package-impact | PASS | Correctly identified frozen-contract status; agent halted on its own anti-fabrication discipline when the proposed field was discovered to already exist at `v3/schema.ts:787` — exemplary cairn behavior. Search trace fully reproducible. |
| §3.1 cairn-methodology skill | PASS | All 5 verbs + subject format + housekeeping prefixes + CLAUDE.md §2.3 citation. |
| §3.2 foxworks-conductor-codebase skill | PASS | Pnpm command, dist-vs-src reasoning, followup ticket ID, CLAUDE.md §3.4 citation all present. |

Tally: 7 / 7 functional pass; 1 cosmetic deviation filed below.

## Followups

| ID | Scope | Origin |
|----|-------|--------|
| `MB-F-A4-STASH-REF-NOT-FIRST-LINE` | The `cairn-test-failure-triage` agent prepended a drift-warning paragraph before the stash ref during validation, violating the `validation/MANUAL_TESTS.md §2.4` spec that requires the stash ref as "the FIRST LINE of output." Defensible because the drift was operator-actionable (cross-session file mutations from a parallel terminal), but spec-deviant. Closure path: revise `agents/cairn-test-failure-triage.md` system prompt to require the stash ref as the literal first line of any output, with drift warnings or other context as a separate trailing block. Tier 3 — non-blocking; functional behavior fully correct. Discoverability: this row + `agents/cairn-test-failure-triage.md` Output Format section + `validation/MANUAL_TESTS.md §2.4` expected-behavior line. | P7 validation 2026-05-07 |

## Format

This file follows the [foxworks-dispatch FOLLOWUPS.md convention](https://github.com/JoshTSeppich/foxworks-dispatch/blob/main/docs/FOLLOWUPS.md):

- **ID**: `MB-F-<TICKET-or-DOMAIN>-<DESCRIPTOR>` in kebab-case.
- **Scope**: prose with problem statement + closure path + explicit `Tier N` label + at least one discoverability anchor (file path or contract section).
- **Origin**: `<TICKET_ID> <status>` or `<session-name> <action>`. For this repo's first FOLLOWUPS.md, expect `P7 validation <DATE>` initially.

Tier criteria (per `skills/cairn-methodology/SKILL.md`):

- **Tier 1**: ship-gate / vision-gate blocker.
- **Tier 2**: non-blocking, materially impacts ship quality, land before next major version.
- **Tier 3**: deferred, nice-to-have, no version commitment.
