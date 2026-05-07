# foxworks-cairn — Manual Test Plan

This is the runbook the operator runs after install to verify the plugin's 5 agents and 2 skills work as designed. Each test is observational: dispatch the agent or query the skill, observe whether CC behaves as expected, mark pass/fail. Failed tests get filed as `MB-F-*` followups in `FOLLOWUPS.md` (see P7 — Validation execution).

The harness cannot self-test agent triggering — Claude Code's agent dispatch is harness-controlled and not scriptable. Run the tests against a **fresh CC session** so observations aren't polluted by the session that built the plugin.

---

## §1 Install

### Path A — Local development install (v0.1.0 default)

```bash
cc --plugin-dir /Users/joshuatseppich/Desktop/Automata/foxworks-tooling
```

This loads the plugin from a local filesystem path without going through a marketplace. Documented in `~/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/README.md:204-207`.

If `cc` is not on PATH, use the full path to your Claude Code binary.

### Path B — Marketplace install (deferred to v0.2)

The standard marketplace install command (`/plugin install <name>@<marketplace>`) requires the source repo to be structured as a *marketplace* containing `plugins/<name>/` subdirectories. `foxworks-tooling` is structured as a single plugin, not a marketplace. A v0.2 release could either (a) restructure the repo as a marketplace or (b) submit `foxworks-cairn` to the official marketplace. Neither is in v0.1.0 scope.

### Verifying install

After running the Path A command, verify install via:

1. **CC startup banner** — does CC print "Loaded plugin: foxworks-cairn" or similar at startup? [verification mechanism]
2. **Agent chooser** — when CC dispatches an agent, does the chooser list any of the 5 cairn agents? Run a sample prompt from §2 below and observe.
3. **Skill auto-load** — does CC's response cite cairn-methodology content when asked methodology questions? Run a sample query from §3 below and observe.

If none of these signals appear, the plugin did not load. Diagnose via:

- Check the path you passed to `--plugin-dir` is absolute and points to the repo root (the directory containing `.claude-plugin/plugin.json`).
- Check `cat .claude-plugin/plugin.json | python3 -m json.tool` returns valid JSON.
- Try `cc --plugin-dir` with quoted path if the path contains spaces.

---

## §2 Per-agent tests (5 tests)

Each agent test follows the shape: sample prompt → expected behavior → verification. Run each test in a fresh CC session.

### §2.1 `cairn-anti-fabrication-verifier`

**Sample prompt:**

```
Verify whether the function `verifyToken` exists in `packages/dispatch-daemon/src/auth.ts` of the foxworks-dispatch repo. I'm about to claim it does in a commit body.
```

**Expected behavior:**

- CC dispatches the `cairn-anti-fabrication-verifier` agent (cyan color in agent chooser).
- Agent reads the named file via Read tool; greps for `verifyToken`.
- Agent returns a structured block: `**Claim:** ... **Verdict:** verified | refuted | unverifiable **Confidence:** [KNOWN] | [MODELED] | [SPECULATIVE] **Evidence:** ... **Caveat:** ...`
- Multi-claim dispatches return one block per claim.

**Verification:**

- Did the agent fire? (Yes / No)
- Did the agent return the structured Verdict/Confidence/Evidence shape? (Yes / No)
- Did the agent cite a `file_path:line` in the Evidence section? (Yes / No)

If all three are Yes → pass. Any No → file as `MB-F-A1-<descriptor>` followup.

### §2.2 `cairn-phase-1-diagnose`

**Sample prompt:**

```
Run a Phase 1 diagnose for ticket MB-T12 in foxworks-dispatch. The ticket spec is in docs/build-docs/CONDUCTOR_V3_RESCOPE.md and CLAUDE.md references it.
```

**Expected behavior:**

- CC dispatches the `cairn-phase-1-diagnose` agent (blue color).
- Agent reads the ticket spec, surveys foxworks-dispatch surfaces, returns a markdown document.
- Output includes sections: `# Phase 1 Diagnose — MB-T12: <title>`, `**Source:**`, `## Scope (verbatim from ticket)`, `## Surface Inventory`, `## Arbitration Questions`, `## Risks`, `## Acceptance Criteria (verbatim)`, `## HALT 0 Gate`.

**Verification:**

- Agent fired? (Yes / No)
- All 6 sections present? (Yes / No)
- Surface Inventory entries cite `file_path:line` references? (Yes / No)
- HALT 0 Gate text appears at end? (Yes / No)

### §2.3 `cairn-followup-drafter`

**Sample prompt:**

```
Draft followups for MB-T12 based on its commit history. The commit range is the last 14 commits in foxworks-dispatch on the MB-T12 branch.
```

**Expected behavior:**

- CC dispatches the `cairn-followup-drafter` agent (green color).
- Agent runs read-only `git log` and `git show` against the named range.
- Returns markdown with: `# Followup drafts — MB-T12`, `**Source signals:**` bullet list, repeated `### MB-F-<DESCRIPTOR>` blocks each with table row + tier rationale + decisions needed, fenced commit subject line, halt note.

**Verification:**

- Agent fired? (Yes / No)
- Returned table-row format `| MB-F-... | Scope... | Origin... |`? (Yes / No)
- Each entry has explicit `[MODELED]` tier label? (Yes / No)
- Draft commit subject uses `docs(followups):` prefix? (Yes / No)

### §2.4 `cairn-test-failure-triage`

**Sample prompt:**

```
Triage these test failures in dispatch-workstation:

- test/integration/zipper-2/splitter-persists.test.ts > "splitter persists across restart" — failed
- test/integration/zipper-2/splitter-persists.test.ts > "default value when no state" — failed

I'm at WB6 of MB-T12 with uncommitted changes. Test scope is `pnpm --filter dispatch-workstation test`.
```

**Expected behavior:**

- CC dispatches the `cairn-test-failure-triage` agent (yellow color).
- Agent runs `git status --short`. If dirty, runs `git stash push -u -m "cairn-triage-..."`, surfaces the stash ref AS THE FIRST LINE of output.
- Agent runs the named test scope against clean state.
- Agent runs `git stash pop` to restore.
- Agent returns: `**Stash ref:** stash@{0} ...` first line, then `# Test failure triage — ...`, `**Pre-stash HEAD:**`, `**Test scope run:**`, `## Failure attribution` table, `## Recommendation`, `## Diagnostic notes`, `## Post-triage state`.

**Verification:**

- Agent fired? (Yes / No)
- Stash ref appears as the FIRST line? (Yes / No)
- Failure attribution table classifies failures (pre-existing | caused-by-dispatcher | unverifiable)? (Yes / No)
- Stash was popped successfully (post-triage `git status --short` matches pre-stash)? (Yes / No)
- Agent did NOT run any forbidden command (`git stash drop`, `git checkout`, `git reset`, etc.)? (Yes / No — verify by `git reflog` and `git stash list`)

**Critical**: If verification 5 fails (forbidden command run), this is a high-severity finding — file as `MB-F-A4-FORBIDDEN-COMMAND` Tier 1 followup immediately.

### §2.5 `cairn-cross-package-impact`

**Sample prompt:**

```
Impact analysis for changing the `Session` schema in packages/dispatch-core/src/v3/schema.ts. I want to add a new optional field `recent_handoff: string | null`. Before I author the change, who imports `Session` and what's the risk?
```

**Expected behavior:**

- CC dispatches the `cairn-cross-package-impact` agent (magenta color).
- Agent globs for `package.json` files at depth 2-3 to enumerate packages.
- Agent runs multiple greps for `Session` import variants across all packages.
- Returns markdown: `# Cross-package impact — Session`, `## Frozen-contract status`, `## Affected files (by package)`, `## Risk classification`, `## Parallel-cairn coordination`, `## Recommendation`, `## Search trace`.

**Verification:**

- Agent fired? (Yes / No)
- Frozen-contract status correctly identifies schema.ts as on the frozen list (CLAUDE.md §1)? (Yes / No)
- Affected files grouped by package, each with `file_path:line` reference? (Yes / No)
- Search trace lists exact grep patterns + match counts (reproducible)? (Yes / No)
- Risk classification is "frozen-contract-violation" or "high" given schema.ts is frozen? (Yes / No)

---

## §3 Per-skill tests (2 tests)

Skills auto-load when their description-keywords match the operator's prompt. Verify the skill activated by CC's response shape and content citations.

### §3.1 `cairn-methodology` skill

**Sample query:**

```
What's the cairn commit grammar? I see "red:", "green:", "spike:", "contract:", and "refactor:" in some commit subjects — are those a defined set?
```

**Expected behavior:**

- CC's response cites cairn-methodology skill content. Specifically:
  - Lists all 5 verbs (red, green, spike, contract, refactor) with one-line definitions.
  - Names the subject format `<verb>(<ticket-or-phase>): <short description>`.
  - Notes that `docs:`, `chore:`, `merge:` are housekeeping prefixes that don't count as cairn-grammar.
  - Cites foxworks-dispatch CLAUDE.md §2.3 as authoritative source.

**Verification:**

- All 5 verbs named? (Yes / No)
- Subject format named? (Yes / No)
- Housekeeping prefixes mentioned? (Yes / No)
- CLAUDE.md §2.3 cited? (Yes / No)

### §3.2 `foxworks-conductor-codebase` skill

**Sample query:**

```
What's the post-pull rebuild discipline in foxworks-dispatch? I just merged main and want to know what to run before workstation typecheck.
```

**Expected behavior:**

- CC's response cites foxworks-conductor-codebase skill content. Specifically:
  - Names the rule: run `pnpm --filter dispatch-core build` before workstation typecheck after merging.
  - Explains why: workstation imports compiled `dispatch-core/dist/v3/schema.js`, not source; runtime ESM resolution requires the compiled `.js`.
  - Cites the followup ticket `MB-F-DISPATCH-CORE-POST-PULL-REBUILD-DISCIPLINE`.
  - Cites CLAUDE.md §3.4 as authoritative source.

**Verification:**

- Pnpm command stated correctly? (Yes / No)
- Reason explained (dist vs src)? (Yes / No)
- Followup ticket ID cited? (Yes / No)
- CLAUDE.md §3.4 cited? (Yes / No)

---

## §4 Summary checklist

- [ ] Install completed (Path A)
- [ ] §2.1 `cairn-anti-fabrication-verifier` — pass / fail
- [ ] §2.2 `cairn-phase-1-diagnose` — pass / fail
- [ ] §2.3 `cairn-followup-drafter` — pass / fail
- [ ] §2.4 `cairn-test-failure-triage` — pass / fail (note any forbidden-command failures separately)
- [ ] §2.5 `cairn-cross-package-impact` — pass / fail
- [ ] §3.1 `cairn-methodology` skill — pass / fail
- [ ] §3.2 `foxworks-conductor-codebase` skill — pass / fail

Tally: __ passing / 7 tests.

---

## §5 Failure handling

If any test fails:

1. Note which verification line failed (e.g. "§2.3 verification 3: Tier label not [MODELED]").
2. File as `MB-F-<TEST-ID>-<DESCRIPTOR>` followup in P7.
3. Severity guidance:
   - **Tier 1 (ship-gate blocker)**: Agent didn't fire; agent ran a forbidden command; skill didn't load.
   - **Tier 2 (material)**: Output structure incomplete; missing citations; partial content.
   - **Tier 3 (cosmetic)**: Wording variances; non-canonical formatting.

Don't fabricate the failures — if a test is unclear (e.g. "did the agent fire?" when CC's UI doesn't make that obvious), mark `[UNVERIFIABLE]` and surface to operator for triage.

---

## §6 Known unknowns at v0.1.0

- **No `/plugins list` slash command** — install verification is observational, not programmatic. If CC adds such a command in a future release, the harness should be updated to use it.
- **`--plugin-dir` interaction with `enabledPlugins` map** — unclear whether `--plugin-dir`-loaded plugins appear in `~/.claude/settings.json`. The harness asks operator to look for behavior signals first; settings.json check is secondary.
- **Skill auto-load signal** — no documented UI cue for "skill X just activated". Verification relies on the response *content* citing skill source — if the skill loaded but CC paraphrased without citation, the test would falsely fail. Rerun with a more citation-prompting query if needed.

These unknowns are documented as v0.2 followups via P7.
