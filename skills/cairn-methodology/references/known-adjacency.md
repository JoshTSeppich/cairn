# Known adjacency

Status: proposed. Not part of 0.1.0. The worked example is bankbot at `53bee49`, and the design exists there only as an unmerged branch, `feat/known-adjacency`, with no code on it.

## What this adds

Cairn has a way up and no way down. Discipline 2 says "`[MODELED]` claims become `[KNOWN]` only by evidence, never by repetition." Nothing says when a `[KNOWN]` stops being one.

Inside one session that gap does not matter, because discipline 2 defines `[KNOWN]` as "observed in this session via tool invocation", and the session ends. It matters once evidence is committed. An evidence directory in a repo is read as known by everyone who opens it later, in sessions that observed nothing, for as long as nobody complains. The code under it keeps moving.

Known adjacency is the way down. It is a new primitive, not a refinement of an existing one. The field reports use "ratchet" only for carrying methodology lessons forward, and no discipline tracks what a claim stood on.

### What it governs

It governs claims that a later reader takes as true now: a README table of evidence runs, a ledger of claims, a findings doc cited as current. It does not govern the `[KNOWN]` and `[MODELED]` labels in commit bodies.

A commit body is a statement about its own commit. Git already records its basis: the tree it sits in, fixed by the SHA and never re-read as HEAD. "[KNOWN] at `8f31b28`" stays true after the code moves, because it never claimed anything about later code.

The line is crossed when a later document cites a commit-body `[KNOWN]` as true of the current tree. That citation is a new claim, and it needs its own ledger entry and its own evidence.

Some commit bodies label "operator-reported" results `[KNOWN]`. That is a separate problem. Discipline 2 defines `[KNOWN]` as observed via tool invocation in this session, and a report from a person is not that. It is a labelling error against discipline 2's own definition, and known adjacency does not fix it.

## Three invariants

Every rule below follows from one of these.

1. **Nothing is known until it has been shown.** A claim starts `[MODELED]` and becomes `[KNOWN]` only when evidence is recorded for it. Absence of complaint is never evidence.
2. **A known standing on a modeled is a contradiction and must not be able to exist.** You cannot know something while standing on a maybe.
3. **Fail noisy and conservative, never silent and optimistic.** A false dependency costs some extra re-proving, which is visible and cheap. A missing dependency leaves something counterfeit-known forever, which is silent and corrupting. Where the design has a choice, it takes the extra dependency.

## The mechanism

### A claim and what it stands on

A claim is a sentence, the evidence that shows it, and its adjacency: everything the evidence was gathered against. Adjacency has two parts.

- **Basis.** Things in the world, each named with the digest it had when the evidence was gathered: a tree, a file, a contract version.
- **Relies on.** Other claims, by id.

When the basis changes or a relied-on claim falls, the claim falls. The fall cascades.

### Only the forward edges are stored

Each claim records what it stands on. The reverse, which claims stand on a given thing, is computed by inverting those records every time the ledger is read. It is never written down.

A stored reverse list is a second copy of the same facts. A second copy can drift from the first, and a reverse list that drifts has lost a dependency without saying so. Invariant three rules it out.

This also settles the lifecycle. Registering a claim on everything it stands on, then keeping those registrations in step, then remembering to remove them: none of that exists. There is nothing to unregister, so unregistering cannot be forgotten.

### Known is computed, never stored

The ledger never holds the word `known`. A claim is `[KNOWN]` at the moment it is checked only if all three hold:

- its evidence passes the project's own evidence verifier;
- every basis digest it recorded equals the digest now;
- every claim it relies on is `[KNOWN]` by the same test.

Anything else is `[MODELED]`, with the reason. Invariant two cannot be broken, because nothing can write a known onto a modeled. Demotion is not an event someone has to trigger. It is what the next check finds.

### The run records its own basis

Digests are taken by the run that gathers the evidence, at the moment it starts, and written into the evidence itself. They are never taken when the claim is recorded.

If they were taken at record time, this sequence would produce a counterfeit known: run on tree A, commit B, record. The ledger would say the evidence was gathered against B. Only the run knows what it ran on.

### The lifecycle is three writes of one entry

| Operation | What it writes |
|---|---|
| Record | The claim's entry, read from evidence that passed verification |
| Re-prove | The claim's whole entry, replaced from new evidence |
| Remove | The claim's entry, deleted |

Each is one atomic write of one entry. Its reverse edges go with it because they are the same data.

There is deliberately no operation that updates a digest. Bumping a digest without re-running is how a counterfeit known would be made by hand.

## Rules that follow from invariant three

| Decision | Choice | Why |
|---|---|---|
| How finely code is tracked | One digest for the whole source tree, plus the lockfile and any policy file | Tracking per module through the import graph misses templates, config and dynamic imports. The `cairn-cross-package-impact` agent already labels dynamic imports `[MODELED]` for that reason. A missed input is a counterfeit known. |
| A run started on a dirty tree | The basis is recorded as `dirty`, and that claim can never be `[KNOWN]` | A state with no name can't be shown against. |
| A digest that can't be read (git error, missing file) | Counts as changed | Never as unchanged. |
| A claim with an empty basis | Rejected when the ledger loads | It would pass without checking anything. |
| Claims that rely on each other in a loop | Rejected when the ledger loads | Neither could be shown first. |
| The evidence's own bytes | Part of the basis | Rewriting a committed log in place is a change to what the claim stands on. |
| A dependency reachable two ways | Record both | A replay records the artifact's digest and also relies on the claim that produced the artifact. Two routes to the same fall cost nothing. |

## When it fires

Demotion is immediate: every check computes it. Re-proving waits until an irreversible operation needs the claim known. That means the gate a project runs before it ships, such as `make review`, CI, a merge or submission, and not every commit.

A pre-commit gate is the wrong place. Under whole-tree digests, one docstring change demotes every claim in the repo. If re-proving costs a paid model run or a person at a browser, a gate that fails on every commit gets bypassed within the day. A bypassed gate is worse than a late one, because it teaches the reader to ignore it. So the gate fails at the point where shipping a counterfeit known would be irreversible, and the check is free to run at any time before that.

### `claims check`

The check prints every claim's state. It lists `[MODELED]` claims heaviest first: by how many claims stand on them, counted through every level, with ties broken by id. Each claim gets the reason it fell and the chain it fell through.

Heaviest first is also a safe order to re-prove in. If A relies on B, everything standing on A also stands on B, and so does A itself. So B always has strictly more claims standing on it than A. Reading the list top to bottom never re-proves a claim before something it stands on.

## Worked example: bankbot

Bankbot is a take-home: a model records a bank workflow once, and replay runs it without the model. Its README has a table of nine evidence runs, each a committed directory plus a sentence saying what it proves. Those nine sentences are the repo's claims. [KNOWN] (read at `53bee49`.)

| Claim | Evidence | Relies on |
|---|---|---|
| 01 | Live discovery compiles to `capability.json` | none |
| 02 | Five replays, one event-sequence hash | 01 |
| 03 | A missing member is an outcome, not a failure | 01 |
| 04 | Session expiry mid-run recovers | 01 |
| 05 | An operator's false claim of completion is rejected | 01 |
| 05b | An operator dismisses a modal and the run succeeds | 01 |
| 06 | Policy blocks a risky action in discovery | none |
| 07 | One artifact on a second tenant's build | 01 |
| 08 | A member the recording never saw | 01 |

Every claim's basis is the `bankbot/` tree, `policy.yaml`, `uv.lock` and its own evidence bytes. The seven replays add the digest of `01-discovery/capability.json`.

### The failure it would have caught: `db6e355`

The timeline, all [KNOWN] from `git log`:

- `ef76834` (Sep 19) recorded runs 1 to 7.
- `be94c5b` (Sep 21, 09:09) changed how the compiler names a step. The committed artifact still said `click_dana_whitfield`, which a recompile at HEAD would no longer produce.
- `9047fe1` (Sep 21, 16:14) took the extracted value out of `output_extracted`, which moves every replay's event-sequence hash.
- `db6e355` (Sep 21, 17:51) re-recorded the five replays on a rebuilt artifact. Its body opens: "Two changes since these were last recorded made them stale."

A person noticed. Nothing failed. Between `ef76834` and `db6e355`, 40 commits touched `bankbot/`, `policy.yaml` or `uv.lock`, and 33 of them were not `docs` commits. For every one of those commits, the README said nine things were proven about a tree that no longer existed.

Under known adjacency, the first of those commits demotes all nine. `claims check` would have printed this (illustrative output, not a real run; digests shortened):

```
MODELED  01   7 stand on it   bankbot/ tree 4e1a0c -> 9b77f2
MODELED  02   0 stand on it   bankbot/ tree 4e1a0c -> 9b77f2; relies on 01 (MODELED)
MODELED  03   0 stand on it   bankbot/ tree 4e1a0c -> 9b77f2; relies on 01 (MODELED)
...
MODELED  06   0 stand on it   bankbot/ tree 4e1a0c -> 9b77f2
```

`make review` fails from that commit on. The staleness `db6e355` found by hand is a failing gate instead.

It would not have said why. It reports that the tree's digest moved, not that the compiler renames steps. Diagnosis is still a person's job. Known adjacency only guarantees the question gets asked.

### An in-place rewrite: `e9d8587`

`e9d8587` rewrote `run_started` in nine committed logs, taking the port out of `base_url`. It used the same function the engine had just started calling. Its body shows two fresh runs whose hashes match the rewritten logs. [KNOWN]

Under known adjacency the rewrite changes the evidence's bytes, so the five claims whose logs it touched (02, 03, 04, 05, 07) fall. The two fresh runs in the commit body are the re-proving. Recording them would have put the claims back. Rewriting and asserting would not.

### What it costs at n=9

Every code commit demotes all nine claims. Getting them back takes two paid model runs (01 and 06) and a person at the browser (05 and 05b). That cost is why the gate is `make review` and not pre-commit, and it arrived at nine claims, not ninety.

### Why it is not in bankbot

Bankbot is a finished submission. Adding this would change `run_started`, which moves every event-sequence hash, including the determinism evidence itself. It would also force all nine runs to be re-recorded. Under invariant one, the existing runs start `[MODELED]`: nobody recorded which tree they ran on, and assuming it would be optimistic. The branch stays unmerged, as the design's proof of fit.

## What it does not do

- **It catches drift, not forgery.** A `run_started` edited by hand before recording looks exactly like a real one. Anti-fabrication still covers lying. This covers going stale.
- **It names which digest moved, not what it meant.** Every change is treated as possibly meaningful, including the ones that aren't.
- **Tree-level granularity demotes on a docstring edit.** I took that. The alternative is a finer graph that can miss something.
- **It sees only what is in the repo.** In bankbot the target app lives in the tree, so a change to it is covered. For a real bank's app, a deprecated model, or a remote contract, the basis needs kinds I have not designed. I have not tested this beyond one repo with nine claims.
- **A revert re-promotes.** If the tree returns to digest X, evidence gathered against X is `[KNOWN]` again. I think that is correct: it was shown against X, and the world is X again.

## Open questions

- Whether this becomes discipline 11 in `SKILL.md`. Answered: it stays a reference until a second project adopts it.
- Where the ledger lives in a project with no package. Frozen contracts live in each project's CLAUDE.md. A ledger probably belongs next to the evidence it describes.
- What the self-check asks. A candidate Q10: "Does any claim this commit's body labels `[KNOWN]` rest on a basis this commit changed?"
