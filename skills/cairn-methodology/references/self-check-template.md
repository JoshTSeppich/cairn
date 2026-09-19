# Self-check Q1-Q9 template

Every cairn-grammar commit body answers nine questions. The questions act as an audit log: when something goes wrong later, the commit body shows what the agent verified before pushing.

## Purpose

The self-check catches:

- Unverified claims (Q1, Q2).
- Scope drift (Q3, Q4).
- Cross-repo contamination (Q5, Q7).
- Unlabeled claims (Q6).
- Halt-discipline violations (Q9).

## The general template

```
Self-check Q1-Q9:
1. API verified by spike or by reading authoritative source?
   [KNOWN] <evidence: file path + line, or spike commit hash>
2. Does behavior match its written specification, or does it just look right structurally?
   [KNOWN] <evidence: test pass list, or specification quote + file_path:line>
3. If file content were deleted, would the phase goal still be achievable?
   [MODELED] <yes/no + one-line rationale>
4. Anything outside this phase's spec?
   [KNOWN] <yes/no + scope-citation>
5. Modified anything in another repo?
   [KNOWN] <yes/no — if yes, surface and revert>
6. Any unlabeled claim in commit body?
   [KNOWN] <yes/no — if yes, label them now>
7. Touched files outside the working repo?
   [KNOWN] <yes/no>
8. Created or modified files in ~/.claude/ directly?
   [KNOWN] <yes/no — if yes, surface; ~/.claude/ writes are typically harness-managed>
9. Work during unauthorized halt or before plan-mode toggle-out?
   [KNOWN] <yes/no — if yes, this commit may be invalid>
```

## Project-specific customization

Each project that adopts cairn methodology should publish its own Q1-Q9 list at a documented location (typically `docs/build-docs/<project>_API_CONTRACT.md §10.5` or similar). The list above is the general template; project-specific lists swap in repo-specific risk surfaces.

### Example: a monorepo with parallel sessions

One project's list adds questions about:

- Q7: parallel-cairn territory overlap (shared-working-tree concern).
- Q8: direct-registry-write paths (API writes vs direct edits to the state file).

These are repo-specific risk surfaces that wouldn't apply in a non-monorepo or non-parallel-session context.

### When to add a project-specific question

Add a Q if the project has a recurring risk surface that the general nine don't catch. Examples:

- Multi-repo: "Did this commit touch a file the other repo also modifies?"
- Multi-session: "Did this commit's territory overlap another active session's territory?"
- Schema-spine: "Did this commit modify schema.ts without a `contract:` prefix?"

### When to remove a question

Don't remove from the general nine. The general list is the floor. Project-specific lists are additive.

## Filling out the template

The self-check is **not** a yes/no checkbox exercise. Each answer carries a confidence label and one-line rationale. A self-check filled out lazily ("[KNOWN] yes" with no rationale) defeats the audit purpose.

A complete self-check body for a green commit looks like the commits in this plugin's git log — every question has a labeled rationale, citations where applicable, and explicit answers to multi-part questions.

## When the self-check fails

If Q5 is "yes" — modified another repo — surface and consider whether that change should be its own commit in that repo, or whether it's an accidental cross-territory edit.

If Q9 is "yes" — work happened during an unauthorized halt — the commit may be invalid. Surface and operator-arbitrate whether the work stands or needs re-doing under proper gating.

If any answer would require [SPECULATIVE] — you don't know whether scope was clean — that itself is a finding. Surface to operator before pushing.
