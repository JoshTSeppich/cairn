# Frozen contracts — foxworks-dispatch

Four surfaces are operator-arbitrated only. CC never modifies them via `green:` or `refactor:` commits. Modifications go through `contract:` commits with explicit operator approval and pass through Q1-Q9 self-check including a custom Q5 ("Did I modify this contract without operator approval?").

Source citation: `foxworks-dispatch/CLAUDE.md §1` (L9-13).

## 1. REGISTRY.md §2 — Registry binary contracts

**File**: `REGISTRY.md` at repo root.

**Section**: §2 — registry binary contract surfaces.

**What's frozen**: contract field schemas, command exit codes, registry-binary IPC payloads.

**Why frozen**: Registry binaries (Sherpa, Lantern, Dispatch CLI) communicate via versioned binary contracts. Drift across binaries breaks cross-binary handshakes.

**How to amend**: operator-authored `contract:` commit modifying §2; downstream binaries adopt the new schema in their own follow-on commits. Never CC-modified.

## 2. CONDUCTOR_API_CONTRACT.md — Conductor v2/v3 API contract

**File**: `docs/build-docs/CONDUCTOR_API_CONTRACT.md`.

**Frozen at**: commit `3ddca60`.

**What's frozen**: HTTP API surface (route tables, request/response schemas), enforcement rules (§10), self-check Q1-Q9 (§10.5).

**Why frozen**: dispatch-cli, dispatch-web, dispatch-workstation all consume the daemon's HTTP API. Drift in route shapes silently breaks consumer behavior.

**How to amend**: operator-authored `contract:` commit. Downstream consumers update their integration tests in subsequent `green:` commits. CC may propose changes via session findings, but the actual edit is operator-territory.

## 3. dispatch-core/src/v3/schema.ts §1-§13 — Zod schema spine

**File**: `packages/dispatch-core/src/v3/schema.ts`.

**Sections**: §1-§13 (the schema-authoritative regions; the file itself documents which sections are frozen at L8-10).

**What's frozen**: all v3 runtime types — Zod schemas defining cross-package contracts.

**Why frozen**: every cross-package contract goes through this file. Duplicating schema definitions across packages was an early failure mode; the spine consolidates them. CLAUDE.md L120 enforces this: "Never duplicate schema definitions across packages."

**Schema-wins rule**: if a markdown spec disagrees with the schema, the schema wins; the markdown is amended to match. Surfacing the drift is operator-arbitrated.

**How to amend**: operator-authored `contract:` commit modifying schema.ts §1-§13. Downstream packages run `pnpm --filter dispatch-core build` to regenerate the dist/ files (per §3.4 post-pull rebuild rule), then update consumers in `green:` commits.

## 4. WORKSTATION_CONTRACT.md §6 — IPC + endpoints

**File**: `docs/build-docs/WORKSTATION_CONTRACT.md`.

**Section**: §6 — IPC channels and endpoint surfaces.

**What's frozen**: IPC channel names, payload shapes between Electron main and renderer, and the workstation's HTTP-fallback endpoint surface.

**Why frozen**: Renderers (tile-grid, console-panel, coarchitect, etc.) all consume IPC; drift in channel names breaks renderer mounts silently. Some channels are also exposed via HTTP fallback for headless test scenarios — drift breaks both.

**How to amend**: operator-authored `contract:` commit modifying §6. Renderer code updates in subsequent `green:` commits.

## General modification procedure

For any frozen surface:

1. **CC surfaces** the proposed change to operator with rationale (typically as a Q-* in a Phase 1 diagnose, or as a finding in a session-findings doc).
2. **Operator decides** whether the change is in scope and what the canonical wording should be.
3. **Operator authors** the `contract:` commit. CC does NOT author this commit.
4. **CC implements** downstream consumer updates via `green:` commits, citing the contract commit hash in commit body Q1.
5. **Self-check Q5** ("Modified anything in another repo?" or project-specific "Did I modify this contract without operator approval?") catches accidental CC-modifications. If Q5 is "yes", revert and re-do under the proper procedure.

## When in doubt

If a proposed change might touch a frozen surface, halt and surface to operator before writing. The cost of asking is low; the cost of an undocumented contract change is operator time spent reverting + re-running affected tests.
