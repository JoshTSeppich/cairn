---
name: foxworks-conductor-codebase
description: This skill should be used when the user references "foxworks-dispatch", "Conductor v3", "dispatch-core", "dispatch-workstation", "dispatch-daemon", "dispatch-cli", "dispatch-web", "tile-grid", "splitter-state", "schema spine", "sentinel zones", "post-pull rebuild", "frozen contracts", or asks for orientation on the foxworks-dispatch monorepo, package layout, or contract surfaces.
version: 0.1.0
---

# Foxworks Conductor Codebase

Orientation for `foxworks-dispatch` — the Foxworks Conductor v3 desktop app + daemon + web surfaces. This skill loads when a fresh CC session lands in foxworks-dispatch and needs to know the codebase shape before touching code.

For methodology (anti-fabrication, halt discipline, cairn grammar), see the `cairn-methodology` skill.

## When to apply

- Fresh CC session opens in foxworks-dispatch and needs orientation.
- User references package names (dispatch-core, dispatch-workstation, etc.) or contract surfaces.
- Operator asks where a feature lives or what frozen contracts a proposed change might touch.
- Pre-WB1 surface check on a ticket whose scope spans multiple packages.

## Codebase facts

### 1. Repo + version model

The active build is **Conductor v3.0**. The v2 schema is frozen at commit `551c469` and lives at `packages/dispatch-core/src/v2/schema.ts`. The v3 schema is at `packages/dispatch-core/src/v3/schema.ts §1-§13` and is the sole source of truth for all v3 runtime types across packages.

If a markdown spec disagrees with the schema, the schema wins; the markdown is amended via a `contract:` commit. CLAUDE.md §3.1 documents this rule.

### 2. Package layout (5 packages, pnpm workspaces)

Five packages under `packages/`, with `pnpm-workspace.yaml` declaring `packages: ['packages/*']`:

| Package | Role |
|---------|------|
| `dispatch-core` | Zod schemas, transport, shared types. Smallest package; contract spine. |
| `dispatch-daemon` | HTTP daemon, registry, route handlers coordinating CC sessions. |
| `dispatch-workstation` | Electron main process + renderer + IPC. Desktop app. |
| `dispatch-cli` | Command-line interface (`fd` CLI). |
| `dispatch-web` | Kanban web UI (React/Vite) consuming daemon HTTP. |

Cross-package imports use `workspace:*` resolution. Workstation imports compiled `dispatch-core/dist/v3/schema.js` (NOT source) for runtime ESM resolution.

### 3. Flat directory convention (workstation)

`packages/dispatch-workstation/src/` uses **flat directories** — each renderer surface is a sibling top-level directory, NOT nested under a category. Per CLAUDE.md §3.2:

```
src/
├── main/                  # Electron main process
├── console-panel/         # xterm.js renderer
├── coarchitect/           # coarchitect chat surface
├── audit-modal/           # audit modal renderer
├── onboarding/            # onboarding renderer
├── error-display/         # error-display renderer
└── tile-grid/             # tile-grid renderer (MB-T12)
```

**Do NOT create `src/renderer/` or other nested category directories.** The flat convention is established; nesting breaks the pattern and breaks the build pipeline (each surface has its own esbuild script).

### 4. Sentinel-marked regions in main.ts

`packages/dispatch-workstation/src/main/main.ts` uses sentinel comments to bracket tracked regions:

```
// === BEGIN: <Name> <description> ===
... region content ...
// === END: <Name> ===
```

As of this skill's snapshot date, main.ts has **18 sentinel zones** (see `references/sentinel-zones-inventory.md` for the current list + refresh procedure). Existing zones are tracked history; new logic goes in NEW sentinel blocks. Never edit inside an existing zone unless your scope matches that zone's declared purpose.

Some zones carry "do not modify outside this block" annotations (e.g. Fix-A api-key bootstrap). These are operator-arbitrated regions — touch only via `contract:` commits.

### 5. Persistence pattern

Workstation persists state via raw `fs.readFileSync` / `writeFileSync` of JSON files in `app.getPath('userData')`. The reference pattern lives at `packages/dispatch-workstation/src/main/splitter-state.ts` (35 lines, env-override via `MB_SPLITTER_STATE_DIR` for test isolation).

Mirror this pattern for new persistence. **Do NOT install `electron-store`** unless explicitly authorized. The flat-fs pattern is intentional — cross-platform debug-friendly, no native dependencies, easy test isolation.

### 6. dispatch-core dist build discipline

Workstation imports `dispatch-core/dist/v3/schema.js` (compiled ESM), NOT `src/v3/schema.ts`. TypeScript path-mapping resolves both at typecheck, but Node ESM at runtime requires the compiled `.js` files in `dist/`.

**Post-pull rebuild rule**: after any merge to main that adds or changes dispatch-core exports, run

```
pnpm --filter dispatch-core build
```

BEFORE running workstation typecheck or smoke. Tracked at `MB-F-DISPATCH-CORE-POST-PULL-REBUILD-DISCIPLINE`. Skipping this rule produces `ERR_MODULE_NOT_FOUND` at runtime, invisible to typecheck.

### 7. Runtime smoke gate for main.ts

Per CLAUDE.md §4.6, any commit touching `main.ts` MUST include a runtime smoke test:

```
pnpm --filter dispatch-workstation exec electron dist/main/main.js
```

Typecheck does not catch `ERR_MODULE_NOT_FOUND` in dynamic imports or the dispatch-core dist gap. The smoke test catches both. Tracked at `MB-F-WORKSTATION-RUNTIME-RELAUNCH-AS-MERGE-GATE`.

### 8. Frozen contract surfaces

Per CLAUDE.md §1 (L9-13), four surfaces are operator-arbitrated only — CC never modifies them:

- `REGISTRY.md §2` — Registry binary contracts.
- `docs/build-docs/CONDUCTOR_API_CONTRACT.md` — Conductor v2/v3 API contract (committed at `3ddca60`).
- `packages/dispatch-core/src/v3/schema.ts §1-§13` — Zod schema spine for all cross-package contracts.
- `docs/build-docs/WORKSTATION_CONTRACT.md §6` — IPC + endpoints.

Modifications to any of these go through `contract:` commits with explicit operator approval. See `references/frozen-contracts.md` for per-surface detail.

### 9. Schema spine authority

`dispatch-core/src/v3/schema.ts §1-§13` is the sole source of truth for v3 runtime types. The schema's authoritative status is encoded in the file itself (L8-10 declares the precedence rule).

When designing changes that touch shared types, start at the schema. Do not duplicate type definitions across packages — every cross-package contract goes through schema.ts.

### 10. Coordination docs convention

Per CLAUDE.md §3.8:

- `docs/coordination/<session-name>-findings-<date>.md` — per-session findings (e.g., `sess-mbt12-findings-2026-05-07.md`).
- `docs/coordination/<ticket>-decisions-<date>.md` — arbitration docs tied to a ticket.
- `docs/cairn-coordination/batch-<N>/` — parallel-batch coordination directories.
- `docs/coordination/parallel-batch-<N>-<date>.md` — cross-session batch coordination.

These are operator-facing logs. New coordination docs follow the dated naming convention so the log stays chronologically navigable.

### 11. Tile-grid + splitter-state (UI subsystems)

Two persistent UI state subsystems worth knowing by name:

- **tile-grid** (MB-T12). Tab-layout chrome (position, collapse). State at `tile-grid-state.ts`. Renderer mounted via WB12 DetachTileIpcController. Build script `scripts/build-tile-grid.mjs`.
- **splitter-state**. Chat/main-content vertical divider position. State at `splitter-state.ts`. Reference persistence pattern (see §5).

Both subsystems use the raw-fs JSON persistence pattern. Both have integration tests in `packages/dispatch-workstation/test/integration/`.

## Authoritative source map

| Topic | Citation |
|-------|----------|
| Frozen contracts | CLAUDE.md §1 (L9-13) |
| Package layout | CLAUDE.md §3.1 (L112-120) |
| Flat directory | CLAUDE.md §3.2 (L122-133) |
| Sentinel zones | CLAUDE.md §3.3 (L134-142) |
| dispatch-core build | CLAUDE.md §3.4 (L154-157) |
| Persistence | CLAUDE.md §3.5 (L159-162) |
| Build pipeline | CLAUDE.md §3.7 (L171-172) |
| Coordination docs | CLAUDE.md §3.8 (L174-175) |
| Runtime smoke gate | CLAUDE.md §4.6 (L225-229) |

## Additional Resources

- **`references/sentinel-zones-inventory.md`** — current sentinel zones list (snapshot date + refresh procedure).
- **`references/frozen-contracts.md`** — detailed coverage of the 4 frozen surfaces.

For methodology (cairn discipline, anti-fabrication, halt rules), see the `cairn-methodology` skill.
