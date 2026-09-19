# foxworks-tooling — Followups

`MB-F-*` followup entries surfaced during plugin validation and future maintenance work. Three-column table: ID, Scope, Origin.

## Validation log

P7 validation run on 2026-05-07 by a fresh Claude Code session in a monorepo that uses cairn. Per `validation/MANUAL_TESTS.md`.

| Test | Result | Notes |
|------|--------|-------|
| §2.1 cairn-anti-fabrication-verifier | PASS | Correctly refuted a fabricated claim; surfaced adjacent candidate files. |
| §2.2 cairn-phase-1-diagnose | PASS | All sections present with `file:line` citations; self-classified as a second-pass diagnose. |
| §2.3 cairn-followup-drafter | PASS | 3 entries in correct table-row format with explicit `[MODELED]` tier labels; honored read-only discipline. |
| §2.4 cairn-test-failure-triage | PASS (cosmetic deviation) | Stash push/pop clean, attribution table accurate, no forbidden commands (verified via reflog and stash list). Stash ref was not the first line of output; filed below. |
| §2.5 cairn-cross-package-impact | PASS | Correctly identified frozen-contract status; halted on its own anti-fabrication discipline when the proposed field turned out to already exist. |
| §3.1 cairn-methodology skill | PASS | All 5 verbs, subject format, and housekeeping prefixes present. |

Tally: 6 / 6 functional pass; 1 cosmetic deviation filed below. The run also validated a repo-specific codebase skill that has since been removed from the plugin.

## Followups

| ID | Scope | Origin |
|----|-------|--------|
| `MB-F-A4-STASH-REF-NOT-FIRST-LINE` | The `cairn-test-failure-triage` agent prepended a drift-warning paragraph before the stash ref during validation, violating the `validation/MANUAL_TESTS.md §2.4` requirement that the stash ref be the first line of output. Closure path: tighten the agent's Output Format instruction so the stash ref line is unconditional and first. Tier 3. | P7 validation 2026-05-07 |

## Format

- **ID**: `MB-F-<TICKET-or-DOMAIN>-<DESCRIPTOR>` in kebab-case.
- **Scope**: prose with problem statement + closure path + explicit `Tier N` label + at least one discoverability anchor (file path or contract section).
- **Origin**: `<TICKET_ID> <status>` or `<session-name> <action>`.

Tier criteria (per `skills/cairn-methodology/SKILL.md`):

- **Tier 1**: ship-gate / vision-gate blocker.
- **Tier 2**: non-blocking, materially impacts ship quality, land before next major version.
- **Tier 3**: deferred, nice-to-have, no version commitment.
