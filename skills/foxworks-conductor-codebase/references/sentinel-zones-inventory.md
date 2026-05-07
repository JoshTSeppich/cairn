# Sentinel zones inventory — main.ts

This document is a snapshot of sentinel-marked regions in `packages/dispatch-workstation/src/main/main.ts` as of skill authoring. Regions evolve; refresh by re-grepping when the snapshot is stale.

## Snapshot metadata

- **Snapshot date**: 2026-05-07
- **Source**: `packages/dispatch-workstation/src/main/main.ts`
- **Method**: `grep -n "^// ===" packages/dispatch-workstation/src/main/main.ts`
- **Foxworks-dispatch HEAD at snapshot**: see `git log -1 --format=%H` in foxworks-dispatch

## Why sentinel zones exist

Per CLAUDE.md §3.3, sentinel zones bracket tracked regions of code so:

- New logic goes in NEW sentinel blocks (not inside existing ones).
- Operator can grep the file by zone name to navigate the change history.
- Some zones carry "do not modify outside this block" annotations marking operator-arbitrated regions.

## Zone style

```
// === BEGIN: <Name> <description> ===
... region content ...
// === END: <Name> ===
```

Variants observed: `// === <Label> ===` (single-line markers), `// === BEGIN: ... ===` paired with `// === END: ... ===`.

## Current zones (as of snapshot)

| Line | Zone name | Purpose |
|------|-----------|---------|
| L31-37 | Onboarding mount imports | Onboarding renderer mount imports |
| L38-47 | MB-T12 tile-grid mount imports | WB12 DetachTileIpcController + window factory |
| L48-50 | Fix-A api-key bootstrap | API key bootstrap; "do not modify outside this block" |
| L51-59 | Fix-C console trigger imports | Cairn finding #82 |
| L60-62 | Fix-92 webview token bootstrap | Cairn finding #92 |
| L63-73 | Probe-92 obs-infra | Probe-92 / fix-verification observation infrastructure |
| L87-98 | Probe-92 obs-infra — userData isolation | Test isolation via env override |
| L99-101 | Onboarding mount path | Onboarding mount file_path resolution |
| L105-113 | Probe-92 obs-infra — kanban webview handle | Webview handle for kanban surface |

(Plus the corresponding `END:` markers for each `BEGIN:` zone — 18 sentinel-comment lines total.)

## Refresh procedure

1. Run `cd packages/dispatch-workstation && grep -n "^// ===" src/main/main.ts | head -40`
2. Compare against the table above; update entries that have moved or changed.
3. New zones: add a row with the new line range, name, and one-line purpose.
4. Removed zones: delete the row.
5. If a zone was renamed, that's [KNOWN-suspicious] — sentinel renames usually indicate scope drift; surface to operator.

## Edit discipline

- **Inside an existing zone**: edit only if your scope matches the zone's declared purpose. The zone name encodes the scope; if your work doesn't fit, you're in the wrong zone.
- **New work**: open a new zone with a `BEGIN:`/`END:` pair below all existing zones (or at the appropriate logical position).
- **"do not modify outside this block" zones**: operator-arbitrated. Touch only via `contract:` commits with explicit operator approval.
- **Removing a zone**: requires `refactor:` or `contract:` commit with self-check Q1-Q9 explaining the removal rationale.

## Cross-reference

Sentinel discipline is encoded in foxworks-dispatch CLAUDE.md §3.3 (L134-142). When in doubt about whether to touch a zone, surface the scope question to operator before editing.
