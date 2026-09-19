# foxworks-cairn

foxworks-cairn is a Claude Code plugin that packages the cairn methodology: a working discipline for agent-assisted software engineering in which every claim carries a confidence label, every commit answers a fixed self-check, and every halt is literal. It ships one skill that documents the ten disciplines and five agents that enforce them at specific points in a ticket. I wrote it after watching an assistant fabricate file contents and drift out of scope on a large monorepo, and these are the rules that stopped that.

## Status

Working. Version 0.1.0 loads from a local clone. It is not packaged as a marketplace, so `/plugin install` does not apply yet.

## Run it

```
claude --plugin-dir /path/to/foxworks-tooling
```

Then ask a methodology question, such as "what is the cairn commit grammar?", or dispatch an agent, such as "verify whether `verifyToken` exists in `src/auth.ts` before I claim it does". The manual test plan in `validation/MANUAL_TESTS.md` has one prompt per agent with the expected output shape.

## What ships

The skill `cairn-methodology` documents the ten disciplines: anti-fabrication, confidence labels, a five-verb commit grammar, the Q1-Q9 self-check, plan-mode discipline, halt discipline, per-commit push, per-path `git add`, frozen contracts, and outcome classifications. Its `references/` folder has worked examples and a self-check template a project can adapt.

| Agent | Purpose |
|-------|---------|
| `cairn-anti-fabrication-verifier` | Read-only check of a factual claim about the codebase before the main session asserts it. Returns a verdict with a confidence label and a `file:line` citation. |
| `cairn-phase-1-diagnose` | Reads a ticket spec and surveys the code it touches. Returns a surface inventory, arbitration questions, and risks before any test is written. |
| `cairn-followup-drafter` | Reads a finished ticket's commit bodies and drafts followup entries for review. Does not commit. |
| `cairn-test-failure-triage` | Stashes uncommitted work, reruns the failing tests on a clean tree, restores the stash, and reports which failures are pre-existing. |
| `cairn-cross-package-impact` | Greps the import graph for a proposed schema or symbol change and classifies the risk, including whether a frozen contract is involved. |

## The main decision

The plugin knows nothing about any particular codebase. The first version bundled a second skill that was a map of the monorepo the methodology came from. I removed it before publishing because a reader who adopts cairn needs the rules, not my project's file layout. Whatever an agent needs to know about a specific repo, such as the frozen-contract list, the project's extra self-check questions, or the ticket ids, it reads from that repo when it is dispatched. That keeps the plugin small and means it cannot carry stale facts about a codebase. The cost is that each project has to maintain a CLAUDE.md that declares those things, and the agents are only as good as that file.
