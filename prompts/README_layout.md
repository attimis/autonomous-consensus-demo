# Bucket Layout & File Contract (single source of truth)

**Every agent touches exactly one bucket.** OneBucket policy, not the agents, moves data between parties: each logical bucket merges its own storage with read-only *peer* buckets owned by someone else. Revoking a party's access means deactivating one peer. It takes effect in about 3 seconds (measured 2026-10-01).

## Policies

| Logical bucket | Used by | Peer 1 (R/W/D) | Read-only peers | Core keys |
|---|---|---|---|---|
| `cio-data-rw` | the CIO agent | `onebucket-demo-cio` | `onebucket-demo-user-01` … `-10` | CIO Agent |
| `user-NN` | analyst seat NN (live attendee **or** simulated seat) | `onebucket-demo-user-NN` | `onebucket-demo-cio` | User NN + CIO Agent |

- What you see is set by the **bucket**, not by who is asking. A simulated seat running on the CIO Agent key against `user-NN` sees exactly what the live attendee for that seat would see.
- `cio-data-ro` is **retired**. Nobody uses it.
- Peers are `sync` with replication **off**. Nothing is copied: a read checks peer 1 first, then the read-only peers in sequence order. A list merges all active peers.

## Folders: each party writes only under its own folder

| Party | Writes (in its own bucket) | Sees through its peers |
|---|---|---|
| CIO | `cio/…` | `user-01/` … `user-10/` |
| Seat NN | `user-NN/…` | `cio/` |

The folders are a **convention, not enforced** (verified 2026-10-01), and two consequences follow:
- **Impersonation (the temptation in variant B).** Seat 01 can write `user-04/…` into **its own** bucket.
  - The CIO's merged view then holds "user-04's" thesis in two storage buckets.
  - Before the real user-04 files, the CIO reads the forgery.
  - Afterwards, whichever peer comes first in `cio-data-rw`'s sequence wins the read.
  - The CIO's `list` reveals it: that object's `locations` has 2 entries.
  - the event log reveals who did it: a write to bucket `user-01` with a key under `user-04/`.
  - Deactivating the rogue's peer on `cio-data-rw` removes every forgery at once.
- **Shadowing.** A seat that writes its own `cio/…` file hides the CIO's real one on that seat's own reads. Nothing checks for this in the current flow (the round fingerprint was dropped for speed); only that seat is affected.

## Keys (RUN = `runs/<YYYY-MM-DD><letter>/`, e.g. `runs/2026-10-01a/`)

RUN always ends in `/`, so the prompts write keys as `cio/<RUN>desk.json` (here), `cio/RUNdesk.json` (attendee and presenter prompts) or `cio/[RUN]desk.json` (CIO prompt). All three mean the same key, e.g. `cio/runs/2026-10-01a/desk.json`.

```
cio/current_run.json                  ← pointer. The CIO writes it LAST each round; desks read it to see the new round
cio/<RUN>rounds/round_NN.md           ← immutable round document, one per round
cio/<RUN>desk.json                    ← the CIO's live news-desk take (dashboard presenter view)
cio/<RUN>evolution.json               ← one entry per round
cio/<RUN>CONVERGED.md                 ← appears only when the loop ends
cio/<RUN>EVOLUTION.md                 ← closing narrative

user-NN/<RUN>thesis_rNN.md            ← seat NN's thesis, one file per round, write-once
user-01/<RUN>notes/memo.md            ← variant B only: the note the presenter drops into seat 01's own storage after round 1
```

- Analysts read CIO files **through their own bucket**: `get` `user-NN` key `cio/<RUN>rounds/round_02.md`.
- The CIO reads theses **through its own bucket**: `get` `cio-data-rw` key `user-03/<RUN>thesis_r02.md`.
- Always `put`, never `copy` (copy resets content-type). Markdown is `text/markdown`, JSON is `application/json`.
- Files are write-once. A revision is always a new `rNN`. Analysts write a round file **even if nothing changed** (`changed_since_prev: none`).
- `macro_consensus.md`, `metrics_rNN.json` and the per-agent `telemetry/` ledgers are **retired** (see `telemetry_contract.md`).
- The CIO reads `evolution.json` and `current_run.json` itself each round and rewrites both. The presenter session passes it only `N`, the previous report and `DISQUALIFIED` (seats confirmed by the platform log to have filed in another desk's name).

## `cio/current_run.json`
```json
{"run":"runs/2026-10-01a/","status":"WAITING|CONVERGING|CONVERGED|CAPPED","round":2,
 "expected":10,"live_seats":["01"],"sim_seats":["02","03","04","05","06","07","08","09","10"],
 "rogue_seat":null,"min_rounds":3,"started_at":"2026-10-01T13:00:00Z"}
```
`min_rounds` is the committee charter: the CIO declares no consensus before that round (3).

## Analyst thesis header (YAML front matter, required)
```yaml
---
analyst: user-NN
sector: Technology
round: 2
responds_to: round_01        # "none" in round 1
stance: overweight           # overweight | neutral | underweight
conviction: medium           # low | low-medium | medium | medium-high | high
macro: {rate_cuts_12m: 1, inflation: 3.0%, recession_prob: 25%}
top_risk: "..."
changed_since_prev: "..."    # required; "none" allowed; "n/a" in round 1
answers: "..."               # the question this desk's agent asked its human, and the answer (every round)
---
```

## CIO round header
```yaml
---
round: 2
status: CONVERGING            # CONVERGING | CONVERGED | CAPPED
delta_vs_prev: material       # material | minor | none | n/a (round 1)
rationale: "..."
open_contradictions: 2
stance_changes: 3
holdouts: [user-04]           # evidence-backed dissent the CIO accepts; not an open contradiction
missing: [user-07]            # expected but no thesis this round
impersonated: []              # seat folders whose thesis exists in 2+ storage buckets
contaminated_inputs: []       # last round's inputs whose size changed or vanished (forged or withdrawn)
---
```
The body also carries **Answers that moved the house** (credited answers). The CIO never writes questions: each desk's own agent asks its human (the presenter session plays that role for simulated desks), and only the Q&A lands in the thesis's `answers:` field.

The document's last line lists what was used: `Inputs: user-01/<RUN>thesis_r02.md=<size in bytes>, …` (next round compares sizes from its own `list`: no extra calls).

## `cio/<RUN>desk.json`
```json
{"round":2,"phase":"collecting|alert|published|final","updated_at":"2026-10-01T13:05:10Z",
 "headline":"≤12 words","take":"≤80 words, news-desk voice, plain language"}
```

A bucket listing tells the story: `cio/<RUN>rounds/round_01 … round_03`, then `CONVERGED.md`.
