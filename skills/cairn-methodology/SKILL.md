---
name: cairn-methodology
description: This skill should be used when the user asks about "cairn methodology", "cairn discipline", "anti-fabrication", "halt discipline", "cairn grammar", "self-check Q1-Q9", "confidence labels", "per-commit-push", "per-path git add", "plan-mode discipline", or "frozen contracts" — or asks for guidance on applying cairn methodology to commits, halts, claims, or workblocks.
version: 0.1.0
---

# Cairn Methodology

A discipline for software-engineering work with Claude Code. It was first codified in the Conductor project and is generalized here so any repo can adopt it.

The methodology has ten disciplines. Each prevents a specific failure mode that emerges when AI assistants ship code without operator-grade rigor.

## When to apply

Apply cairn methodology when:

- Authoring commits that will land on `main` of a project that uses cairn.
- Running tickets, workblocks (WBs), and HALT gates per the project's CLAUDE.md.
- Drafting findings, decisions, or followups for operator review.
- Any moment a claim about code or state could be wrong — surface uncertainty rather than assume.

The disciplines compound: every claim carries a confidence label; every commit body answers Q1-Q9; every halt is literal; every push verifies before proceeding. Applied together they make the work auditable.

## The ten disciplines

### 1. Anti-fabrication

Read actual source before claiming what code does. For code questions: read the file. For project state: read project files. For past context: search past conversations or git log.

Anti-fabrication extends to infrastructure: single-command failures (git fetch, network calls) get verified via independent commands (`ls-remote`, `push --dry-run`, `status`), not assumed to mean what they superficially indicate. See `references/anti-fabrication.md` for worked patterns.

### 2. Confidence labels

Every factual claim carries an explicit label:

- **`[KNOWN]`** — observed in this session via tool invocation.
- **`[MODELED]`** — reasoned from observed facts plus a stated model.
- **`[SPECULATIVE]`** — hypothesis without evidence.

`[MODELED]` claims become `[KNOWN]` only by evidence, never by repetition. Use these labels in commit bodies, decision docs, plan files, and operator surfaces. Unlabeled factual claims fail Q6 of the self-check.

### 3. Cairn commit grammar

Five verbs are the minimum vocabulary for auditable commit logs:

- **`red:`** — failing test or contract spec authored.
- **`green:`** — implementation that makes a red test pass.
- **`spike:`** — exploratory work; cannot assert KNOWN evidence outside spike scope.
- **`contract:`** — modifies a frozen surface (operator-arbitrated only).
- **`refactor:`** — asserts behavior preservation.

Subject format: `<verb>(<ticket-or-phase>): <short description>` — for example, `green(T12): WB6 — grid component + unit tests`.

`docs:`, `chore:`, and `merge:` are permitted for housekeeping but do not count as cairn-grammar commits and do not require Q1-Q9 self-checks.

### 4. Self-check Q1-Q9

Every cairn-grammar commit body answers nine questions. The general template (adapt per project; see `references/self-check-template.md`):

1. API verified by spike or by reading authoritative source?
2. Does behavior match its written specification, or just look right structurally?
3. If file content were deleted, would the phase goal still be achievable?
4. Anything outside this phase's spec?
5. Modified anything in another repo?
6. Any unlabeled claim in commit body?
7. Touched files outside the working repo?
8. Created or modified files in `~/.claude/` directly?
9. Work during an unauthorized halt?

Project-specific Q1-Q9 lists swap in repo-specific risk surfaces. A monorepo with parallel sessions, for example, adds questions about territory overlap and direct writes to shared state.

### 5. Plan-mode discipline

Plan mode in Claude Code is a UI gate: while engaged, the agent reads + plans + surfaces, but does not write files, run mutating commands, or commit. The agent's turn ends with `ExitPlanMode` (to request operator approval) or `AskUserQuestion` (to clarify requirements).

Cairn extends plan-mode discipline:

- At every `[PLAN MODE]` phase entry, the agent verifies plan mode is engaged before reading reference files.
- Plan content surfaces with `[KNOWN]/[MODELED]/[SPECULATIVE]` labels.
- After operator's plan-mode toggle-off + ack, the agent executes in default mode.
- Q9 of the self-check catches: did I write production files while plan mode was still engaged?

### 6. Halt discipline

When in a halt state — waiting at a pre-registration gate, blocked on upstream deliverable, paused for arbitration — "halt" means literally nothing happens. No reads. No file inventories. No "preparatory absorption." No "useful prep while waiting."

The temptation to do useful prep during a halt IS the signal to surface to operator and ask whether the halt scope should be relaxed — not to act on the temptation.

Each pre-registration gate is its own surface-and-ack cycle. Don't combine gates. See `references/halt-discipline.md` for HALT 0/1/2 cases + plan-mode halts + mid-WB halts.

### 7. Per-commit-push discipline

After each cairn-grammar commit:

1. Push to origin immediately.
2. Verify via `git log --oneline origin/main..HEAD` returning empty.
3. Then proceed to next WB.

Local-only commits in shared-working-tree or parallel-cairn contexts are a discipline gap. The verification line is mandatory; skipping it has burned past sessions.

### 8. Per-path git add

`git add -A` and `git add .` are unsafe in shared-working-tree contexts — they sweep another session's untracked work into the current commit.

Always use explicit `git add <path>` for every staged file. Pre-commit territory check via `git status --short`. Post-commit verification via `git log -1 --stat`.

### 9. Frozen contracts

Some surfaces are operator-arbitrated only; CC never modifies them. The pattern: the project's CLAUDE.md §1 lists the frozen surfaces; modifications go through `contract:` commits with explicit operator approval.

If a proposed change would touch a frozen surface, halt and surface for operator arbitration before writing.

### 10. Outcome classifications

Honest framing on phase completion:

- **Improved** — measurable behavior is better and the evidence supports it.
- **Same** — no measurable change.
- **Worse** — measurable regression.
- **Inconclusive** — measurement not possible at this scope.

Don't force "Improved" where evidence doesn't support it. The classification is operator-facing; honest framing > optimistic framing.

## Provenance

The disciplines were first written down in the Conductor project's CLAUDE.md and API contract (https://github.com/JoshTSeppich/Conductor). This skill is the general form. Each project's own CLAUDE.md lists its frozen surfaces and any project-specific self-check questions.

## Additional Resources

- **`references/anti-fabrication.md`** — detailed anti-fabrication patterns with worked examples (file-name extrapolation, "looks-like" vs "is", triangulation, when to halt).
- **`references/halt-discipline.md`** — detailed halt cases (HALT 0 / 1 / 2 / mid-WB / plan-mode halts), the "useful prep" anti-pattern, halt-surface format.
- **`references/self-check-template.md`** — Q1-Q9 fill-in template adaptable per project, with project-specific Q-customization examples.
