# Halt discipline

When in a halt state, "halt" means literally nothing happens. No reads. No file inventories. No "preparatory absorption." No "useful prep while waiting." Each gate is its own surface-and-ack cycle.

This document covers the halt cases in cairn methodology and how to surface when halting.

## The principle (CLAUDE.md §2.5)

The temptation to do useful prep during a halt IS the signal to surface to operator and ask whether the halt scope should be relaxed — not to act on the temptation and call it within spirit.

If a halt feels productively-wasteful, surface to operator with: what work would be useful, what risks doing it, what risks not doing it. Operator decides whether to relax halt scope.

## The HALT gate ladder (CLAUDE.md §4.2)

Tickets that use cairn methodology pass through these gates:

### HALT 0 — pre-execution surface

Before WB1 RED begins. Run a Phase 1 diagnose: read the ticket spec, surface arbitration questions (Q-MB-TXX-N), surface risks (R1, R2, ...), document acceptance criteria. Operator acks each Q-* and R-* before WB1 RED.

### Per-N-WB status surfaces

Every 3-4 WBs, surface progress for operator awareness. Not a blocking halt; just visibility.

### HALT 1 — pre-final-verification

Before the findings doc + last verification pass. Operator reviews work-in-progress before final commit.

### HALT 2 — pre-push-to-main

Operator-arbitrated final commit + push. Never chain operator-arbitrated actions behind verification commands.

## Other halt cases

### Mid-WB halt

A workblock surfaces an unexpected obstacle (test failure, scope drift, contract surprise). Halt; surface; do not retry by guessing. Use the `cairn-test-failure-triage` agent for failure triage; use the `cairn-anti-fabrication-verifier` agent for "is this real" checks.

### Plan-mode halt (build prompt §1.5)

Claude Code's plan mode is a halt-by-design: while engaged, the agent is read-only except for the plan file. The agent's turn must end with `ExitPlanMode` (to request approval) or `AskUserQuestion` (to clarify). Production-file writes during plan mode fail Q9 of the self-check.

The cairn additions on top of plan mode:

- Verify plan mode is engaged before reading reference files at every `[PLAN MODE]` phase entry.
- Plan content surfaces with `[KNOWN]/[MODELED]/[SPECULATIVE]` labels.
- After operator's toggle-off + ack, the agent executes in default mode.

### Halt at out-of-scope work

If a request would expand scope beyond the current ticket, halt and surface — do not silently absorb the expansion. New work goes in `MB-F-*` followups, not in current scope.

### Halt at frozen-contract touch

If a proposed change would modify a frozen-contract surface (CLAUDE.md §1), halt and surface for operator arbitration. Frozen-contract changes go through `contract:` commits, not silent edits.

## The "useful prep" anti-pattern

When halted, the agent feels the urge to "be productive" — read the next phase's docs, prep file inventories, draft tests for the upcoming WB. This is exactly the failure mode the halt rule prevents.

The reasoning: if you do "useful prep" during a halt, you've shifted the operator's gate-ack cycle into something you decided. The next phase's plan now reflects choices you made before the operator authorized them. The halt was supposed to be a checkpoint; you've smuggled work past the checkpoint.

Surface the urge instead of acting on it. The operator will tell you whether the halt scope can be relaxed.

## Halt-surface format

When halting, the surface to operator includes:

1. **What changed.** What just happened that triggered the halt (commit landed, test failed, gate reached).
2. **Where you are.** Repo state: commit hash, branch, sync status with origin.
3. **What's pending.** What awaits operator action (ack, arbitration, decision between options).
4. **What you will not do during halt.** Explicit statement: "no further reads or prep until ack."
5. **What signal you're listening for.** "When ready, ack to proceed to <next phase>."

A halt-surface that lists "things I'm thinking about during the halt" is a halt-discipline gap. Halt means nothing happens.

## Halt at plan boundaries

A plan file is itself a halt artifact: it surfaces the plan to operator, then the agent halts pending approval (`ExitPlanMode`). Multi-plan workflows that try to span the halt (drafting plan B while plan A awaits approval) violate halt discipline. One plan, one approval, one execution.
