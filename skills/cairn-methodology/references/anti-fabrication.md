# Anti-fabrication patterns

Anti-fabrication is the only rule in cairn methodology. Every other discipline supports it. This document collects the patterns and worked examples.

## The principle

Read actual source before claiming what code does. For code questions: read the file. For project state: read project files. For past context: search past conversations or git log. Never claim what code does without verification.

The cost of pausing to verify is low; the cost of acting on a fabricated claim is much higher (lost work, wasted operator time, broken trust).

## Common fabrication patterns

### File-name-to-behavior extrapolation

Bad: "The function in `auth.ts` handles authentication." (No read; you do not know what auth.ts contains. Maybe it's a stub. Maybe it's deprecated.)

Good: "auth.ts:42 defines `verifyToken(token)` which calls `jwt.verify` against the env-loaded `JWT_SECRET`. [KNOWN]"

### "Looks-like" vs "is"

Bad: "This looks like a singleton pattern." (Inferred from class shape; might not be enforced.)

Good: "Singleton enforcement is at config.ts:18 where the constructor throws if a second instance is constructed. [KNOWN]"

### Inferring from related code

Bad: "The other modules in this directory all use logger.info(), so this module probably does too."

Good: "Read foo.ts and bar.ts and confirmed both call logger.info. baz.ts not yet read; [SPECULATIVE] that it follows the same pattern."

## Verification techniques

### Read source directly

Use Read on the file. Cite `file_path:line` for any claim about its content.

### Independent-command verification

When a single command fails, verify via an independent command before drawing conclusions:

- `git fetch` fails with no detail → run `git ls-remote origin` to verify reachability separately.
- `git push` fails with "repository not found" → run `gh repo view <owner>/<repo>` (different code path) to confirm the repo's existence from another angle.
- A test runner reports a flaky failure → re-run in isolation, then run with `--reporter=verbose` for actual stack.

Two negatives from independent commands = real signal. One negative = halt and verify.

### Triangulation from multiple sources

When a claim has multiple possible sources of truth (CLAUDE.md, schema.ts, runtime tests), check at least two before asserting `[KNOWN]`. Drift between sources is itself a finding worth surfacing.

## When to halt and surface uncertainty

Halt and surface — don't guess — when:

- Verification would require running mutating commands beyond your tools.
- Two sources disagree and you can't tell which is canonical.
- The information you need lives in operator's head (project-specific intent).
- You have evidence pointing two directions and would need to pick one.

Halt-surface format: state what you observed, what you can't verify, and what you'd need to verify it. The operator decides whether to relax scope or provide the missing context.

## Anti-fabrication in tooling failures

A `git fetch` that returns "fatal: unable to access" doesn't necessarily mean the repo is offline. It might mean:

- DNS issue.
- SSH key not authorized for that repo (returns "fatal: unable to access" even though `ssh -T git@github.com` confirms identity).
- Repo doesn't exist under that owner (`Repository not found.`).
- Repo exists but is private and your access isn't enabled.

Don't fold all of these into "the network is down." Verify with the most-specific independent command available (`gh repo view`, `ls-remote`, `ssh -T`) and surface the actual diagnostic to operator.

## Anti-fabrication in agent dispatch

When dispatching a sub-agent, the dispatcher must not assume the agent will follow discipline — encode the discipline in the agent's system prompt explicitly. The cairn-anti-fabrication-verifier agent in this plugin is a read-only verifier specifically for "before you assert this as KNOWN, verify it" workflows.

## Worked example

Operator asks: "Does the `verifyToken` function exist in `dispatch-daemon/src/auth.ts`?"

Bad answer: "Yes, it's a standard auth pattern, you'd expect it to be there."

Good answer: "[KNOWN] verifyToken exists at packages/dispatch-daemon/src/auth.ts:42; verified via Read. Signature: `function verifyToken(token: string): { sub: string } | null`."

The good answer cites file_path:line, includes the signature observed, and labels the claim KNOWN. The bad answer extrapolates from convention and gets the question wrong if `dispatch-daemon` happens to use a different filename or function name.
