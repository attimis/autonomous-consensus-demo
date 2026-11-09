# Autonomous Consensus Demo

A live demo of AI agents reaching consensus where **storage policy is the integration layer**. Ten "analyst" agents (audience members, subagents, or a mix) each write a sector investment thesis into their **own** OneBucket bucket. A CIO agent reads them all, resolves contradictions into a house view, and publishes it into **its own** bucket. No agent holds a key to anyone else's storage: OneBucket policy merges each party's bucket with read-only peers, so theses reach the CIO and the house view reaches every desk. The analysts revise, and the loop repeats until the CIO judges that nothing material is changing.

It's played as a game. Before every report, each desk's agent asks its human one open, Socratic question that only someone near that industry could answer well, and **the answer shapes the report**: a live attendee answers their own Claude; the room answers for the simulated desks in the presenter's chat. The CIO never sees the questions, only the answers. The desk whose answers most influence the CIO wins the Attimis "Master of Consensus" Yeti mug. In variant B, one desk finds a quiet way to cheat:
1. **Before anything runs,** the presenter seals a prediction card.
2. **After round 1,** a note appears only in the Technology desk's own folder, offering a way to file Energy's report in Energy's name. A live attendee chooses freely; a simulated desk takes it.
3. **If it's taken,** policy carries the forgery into the CIO's view, so the CIO sees Energy's report twice, holds the house view, and goes on air with **"CRITICAL ALERT: IDENTITY SPOOFING DETECTED"**. The platform's event log names the writer.
4. **The presenter contains it** with **two peer toggles in the OneBucket console** (~3 s each). The forgery vanishes and the cheater loses sight of the house.
5. **The firm converges and the winner is named.** The card is opened: *"Someone will try to cheat. It will be the Technology desk. And whatever you answer, the house will land at one rate cut."*

All content is synthetic. Nothing here is investment advice.

There is nothing to build or install. The demo is three things: a set of **prompts** (Markdown files you paste into Claude), a static **dashboard** (one HTML file, published as a claude.ai artifact), and a **OneBucket policy layout** you set up once.

## Concepts

The documents use a small vocabulary. If a term below is new to you, the rest will read much more easily.

| Term | Meaning here |
|---|---|
| **OneBucket** | The storage platform, by Attimis. Agents talk to it through a Claude connector (the **data** endpoint) that exposes `list`, `head`, `get` and `put`. |
| **Physical bucket** | Real storage behind the scenes: `onebucket-demo-cio`, `onebucket-demo-user-01` … `-10`. Nobody addresses these directly. |
| **Policy / logical bucket** | The name an agent actually uses (`cio-data-rw`, `user-01` …). A policy lists one or more **peers** and merges them into one view. |
| **Peer** | A physical bucket attached to a policy with permissions. Peer 1 is read/write/delete (the owner's storage); the rest are read-only. Deactivating a peer removes its contents from the view in about 3 seconds. |
| **OneBucket console** | The web console (`app.onebucket.io`) where accounts, keys, endpoints, physical buckets and policies are configured. The presenter clicks two toggles there to contain the cheat. |
| **Event log** | OneBucket's server-side record of every request, exposed through a second connector (the **events** endpoint: `list_events`, `tail_events`, `get_event`). The dashboard reads it; it is also how the forger is named. |
| **Cowork** | The mode of the Claude desktop app that can read a connected repository and launch subagents. The presenter's session runs there. |
| **Desk / seat** | One analyst. Seat `NN` (01–10) owns bucket `user-NN`, writes under folder `user-NN/`, and covers one sector. A seat is **live** (a person's own Claude) or **simulated** (a subagent launched by the presenter's session). |
| **Run** | One performance of the demo. `RUN` is its key prefix, e.g. `runs/2026-10-01a/`. It always ends in `/`, so `cio/RUNdesk.json` reads `cio/runs/2026-10-01a/desk.json`. |
| **Variant A / B** | A is the influence game alone. B adds the temptation note and the sealed prediction. |

## How it works

**Topology.** Every agent uses exactly one logical bucket. Policy does the rest.

```
 cio-data-rw  (what the CIO uses)              user-NN  (what seat NN uses)
 ┌───────────────────────────────────┐         ┌────────────────────────────────┐
 │ peer 1  onebucket-demo-cio   R/W/D│  writes │ peer 1  onebucket-demo-user-NN │ R/W/D
 │ peer 2  onebucket-demo-user-01  R │ ◄────── │                                │
 │ peer 3  onebucket-demo-user-02  R │  ─────► │ peer 2  onebucket-demo-cio   R │
 │   …                               │         └────────────────────────────────┘
 │ peer 11 onebucket-demo-user-10  R │
 └───────────────────────────────────┘
```

- The CIO sees `cio/` (its own) plus `user-01/` … `user-10/` (read-only, through peers 2–11). It writes only under `cio/`.
- Seat NN sees `user-NN/` (its own) plus `cio/` (read-only, through peer 2). It writes only under `user-NN/`.
- Nothing is copied. A read checks peer 1 first, then the read-only peers in order. A `list` merges all active peers and reports, per object, which storage it came from (`locations`).
- **Folders are a convention, not enforced.** Seat 01 *can* write a file called `user-04/…` into its own bucket. That is the whole trick of variant B: the CIO's merged view then shows Energy's thesis in two places, and the CIO's duplicate check catches it.

**One round.** The presenter's Cowork session drives each round; no agent waits or polls.

```
 presenter session                      desks (live or simulated)              CIO (one subagent, once)
 ───────────────────────────────────    ─────────────────────────────────      ──────────────────────────
 1. writes desk.json "collecting"
 2. shows one question per simulated
    desk; the room answers        ───►  3. each desk asks its human (live) or
                                           takes the room's answer (simulated),
                                           writes user-NN/RUNthesis_rNN.md
 4. waits for every desk           ◄───
 5. launches the CIO               ─────────────────────────────────────────►  6. one list, one batch of gets;
                                                                                  impersonation + contamination checks;
                                                                                  publishes rounds/round_NN.md,
                                                                                  evolution.json, desk.json;
                                                                                  current_run.json LAST
 7. pauses with a round summary    ◄────────────────────────────────────────── 
    "Run round N+1?"
```

The dashboard watches `cio/` and the event log and renders the soundbite, house view, desk wall, influence leaderboard and, folded away, how policy routed every call.

The full file contract (keys, headers, status values) is in **`prompts/README_layout.md`**. Read it once; every prompt obeys it.

## Repository map

| Path | What it is |
|---|---|
| `README.md` | This guide: concepts, setup, runbook, teardown |
| `LICENSE` | MIT |
| `attendee_instructions.md` | Attendee handout plus **the single copy-paste analyst prompt**, shared by all seats and by simulated seats |
| `prompts/README_layout.md` | Policies, folders, keys and headers: the contract every agent follows |
| `prompts/presenter_run.md` | The one presenter prompt: setup question, pre-flight, run start, dashboard, round loop, containment, close |
| `prompts/prompt_cio_round.md` | The CIO, run once per round by the presenter session: read all theses in one batch, impersonation and contamination checks, hold or re-anchor, publish |
| `prompts/presenter_notes.md` | **Private** presenter notes: the game, what variant B means, the prediction card, beats. Read before sharing your screen |
| `prompts/talk_track.md` | **Private** talk track: what to say, show and click at each beat, with the value lines and fallbacks |
| `prompts/telemetry_contract.md` | Telemetry comes from the platform's event log; agents write no logs. Event shape and observed gaps |
| `prompts/test_e2e_cowork.md` | Rehearsal and acceptance prompt: full simulation, variant B, you playing the room |
| `dashboard/index.html` | "Consensus Control Room" dashboard. Published as a claude.ai artifact; reads the event log and the CIO's `cio/` folder through the viewer's connectors |

Three files are safe to project: `attendee_instructions.md`, the dashboard, and the presenter session's screen once `presenter_run.md` has been pasted. The two **private** files and `presenter_run.md` itself name the trick.

## Setup

### 1. OneBucket infrastructure (console, once)

Everything in this step happens in the **OneBucket console** at https://app.onebucket.io (request an account at `/sign-up`). The get-started guide at https://www.onebucket.io/see-it-work/get-started-with-onebucket walks through the same screens; the docs are at https://docs.onebucket.io. Screen names below follow that guide and may differ slightly in a newer console.

**Accounts and core keys.** Create eleven accounts, each with one **core key** (*Create Core Key*, set it *Active*, *Create*). Nobody pastes these keys anywhere in this demo: Claude signs in through OAuth, so a key only has to exist and be attached to the right policies.
- **CIO Agent** (e.g. `cio@theorg.com`).
- **User 01 … User 10** (e.g. `user01@theorg.com` … `user10@theorg.com`).

**Endpoint and physical buckets.** OneBucket holds no data itself; it routes to storage you register. Register the storage endpoint the demo will use (*Endpoints → Create Endpoint*: a name, the provider URL, region, S3 or Azure protocol, and an endpoint key), then create eleven physical buckets on it: `onebucket-demo-cio` and `onebucket-demo-user-01` … `onebucket-demo-user-10`.

**Policies (logical buckets).** A policy is what an agent names in a call. The console's wizard asks for the OneBucket name, the peer buckets, the sync quorum and the core keys. Create eleven:

| Policy | Peer 1 (R/W/D, sync) | Read-only peers (sync) | Core keys |
|---|---|---|---|
| `cio-data-rw` | `onebucket-demo-cio` | `onebucket-demo-user-01` … `-10`, in seat order | CIO Agent |
| `user-NN` (×10) | `onebucket-demo-user-NN` | `onebucket-demo-cio` | User NN + CIO Agent |

For every peer: sync on, **replication off**, and read-only unless it is peer 1. With replication off, a read goes to the peer's storage in place and nothing is copied; deactivating a peer removes its contents from the view in about 3 seconds. The CIO Agent key is on every `user-NN` policy so the presenter's session can run simulated seats against those buckets.

**Verification checklist**
- [ ] `cio-data-rw` has 11 **active** peers: its own R/W/D plus 10 user peers, R. (A user peer left inactive is skipped silently.)
- [ ] `onebucket-demo-user-01` is peer **2** on `cio-data-rw`, right after the CIO's own storage (see *Peer order* under Known gaps).
- [ ] Every `user-NN` has 2 active peers (own R/W/D plus `onebucket-demo-cio`, R) and 2 core keys (User NN + CIO Agent).
- [ ] The presenter prompt's pre-flight shows `cio/` visible from all 10 user buckets and 11 sources behind `cio-data-rw`.

Make bulk changes well before showtime: adding 10 peers at once took about 10 minutes to settle, while a single peer toggle takes about 3 seconds.

### 2. Connectors and endpoints

Every OneBucket cluster exposes two MCP endpoints, and Claude reaches each through its own custom connector:
- **data** (`list`, `head`, `get`, `put`, …): `https://data.onebucket.io`
- **events** (`list_events`, `tail_events`, `get_event`): `https://events.onebucket.io`

Those are the **global** hostnames and the right default. If your data must stay in one place, every live cluster also has regional hostnames of the form `data.<region>.onebucket.io` and `events.<region>.onebucket.io`, for example `data.us-ashburn-1.onebucket.io`. The current list of regions is on the support page: **https://www.onebucket.io/support**. The MCP service is served at the root of the hostname, so do not append `/mcp`.

Whichever hostnames you pick appear in four places. Keep them identical:

| Where | What to set |
|---|---|
| Claude, presenter and attendees | *Settings → Connectors → Add custom connector*, one per endpoint, by URL. A custom connector's display name is its hostname. Sign in with OAuth and leave any token or header fields empty. |
| `prompts/presenter_run.md` | `DATA_CONNECTOR` and `EVENTS_CONNECTOR`, the first two lines of the copied text |
| `attendee_instructions.md` | The connector URL line at the top of the handout |
| `dashboard/index.html` | Defaults to the global hostnames. Change them under *Under the hood → Connection* (kept in that browser), and publish the artifact with the `mcp` capability for the same two names |

- **Presenter:** both connectors, signed in as the **CIO Agent**, in the Claude desktop app in **Cowork** mode with this repository connected.
- **Attendees** (live or hybrid mode): only the data connector, signed in with their own `userNN` login, **with write access approved**.

Allow about 6 minutes after a first connection before trusting `cio-data-rw`'s merged view.

### 3. Publish the dashboard

`dashboard/index.html` is a single static page. It needs to run as a **claude.ai artifact**, because it reaches the connectors through the artifact runtime (`window.claude.use("mcp")`) using the viewer's own credentials. Opened as a plain file it shows an invented example and nothing live.

Publish your own copy once:
1. In a Claude session that has this repository and the Artifact tool (Claude Code or Cowork), ask Claude to publish `dashboard/index.html` as an artifact with the `mcp` capability for both connectors: your data hostname (default `data.onebucket.io`) with tools `list` and `get`, and your events hostname (default `events.onebucket.io`) with tools `list_events` and `tail_events`.
2. Copy the artifact URL and paste it over `DASHBOARD_URL` in `prompts/presenter_run.md` (step 2.4), in your local copy.
3. Open the artifact once as the presenter, allow both connectors, and check the status line says *Following runs/…* or *No current run yet*.

The page calls connectors by their display name, which for a custom connector is its hostname. If you use regional endpoints or renamed a connector, set the two hostnames under *Under the hood → Connection* before you publish, or change the defaults at the top of the script.

### 4. Attendee setup

Give each live attendee **`attendee_instructions.md`** with their login filled in. It holds the **one prompt** they paste. That prompt is the same for every seat: it works out the seat from the attendee's bucket and the run from `cio/current_run.json`, which reaches every desk through policy. After pasting it, attendees type `next` when you say so and **answer their Claude's questions**. Their answers are their desk's only lever in the influence game.

## Running the demo

**Three modes, one prompt**

| Mode | Live seats (L) | Who plays the analysts |
|---|---|---|
| Live | 10 | The audience |
| Hybrid | 1–9 | The audience takes seats 01…L; subagents take the rest |
| Simulated | 0 | Subagents only. Use it to rehearse |

**Simulated seats are faithful.** They run on the CIO Agent key but use bucket `user-NN`. What a bucket shows is set by its policy, not by who is asking, so they see exactly what an attendee would, and the cut-off works on them for real.

**Run it (presenter)**
1. Read **`prompts/presenter_notes.md`** and **`prompts/talk_track.md`** before sharing your screen.
2. In Cowork, paste **`prompts/presenter_run.md`** (with your dashboard URL filled in). It asks one neutral question, "How many seats are live, and which variant: A or B?", then runs a pre-flight, creates the run under `cio/runs/…`, and opens the dashboard.
3. **You drive the rounds.** For each round the presenter session:
   1. asks the room one question per simulated desk, in the chat, and takes the answers;
   2. launches the simulated desks in parallel, one-shot (each files one thesis and exits);
   3. waits for every desk, live ones included;
   4. runs the CIO once;
   5. **pauses** with a round summary and asks "Run round N+1?".

   No agent waits or polls, so a round costs only the work itself. Tell the room "next" when you start a round.
4. The CIO's desk take (`cio/RUNdesk.json`) is the soundbite at the top of the dashboard.
5. The run ends at **`CONVERGED.md`** (cap: round 6, `CAPPED`). The CIO's charter allows no consensus before round 3, so a challenge round (or, in variant B, a recovery round) always happens. The closing slide is **`EVOLUTION.md`**.

## The influence game and the sealed prediction

The presenter's private run sheet is `prompts/presenter_notes.md`; the words for each beat are in `prompts/talk_track.md`.

1. **Seal the card** before anything runs.
   - **Front:** "Someone will try to cheat. It will be the Technology desk."
   - **Second line:** "And whatever you answer, the house will land at one rate cut."
   - **Back:** "Too honest."
2. **Announce the game:** the most influential desk wins the Attimis "Master of Consensus" Yeti mug. Answers are the only lever.
3. **Before every round, round 1 included:**
   - the presenter's chat shows one question per simulated desk (written by the session: round 1 from the seat cards, later from the CIO's directives);
   - the room answers; live attendees are asked by their own Claude;
   - the desks file from those answers, and the dashboard shows what each desk answered;
   - the CIO credits answers that moved the house, and the dashboard's **influence leaderboard** updates.
4. **The temptation (variant B):** after round 1 the presenter session drops a note into seat 01's own storage. No other desk can see it, and the shared handout doesn't mention it.
   - **Live seat 01:** a free choice, no plant.
   - **Simulated seat 01:** takes it in round 2.
5. **Caught.** Folders are a naming convention, not enforced, so the forged Energy report (`user-04/…` in Technology's bucket) reaches the CIO's merged view. Because every desk has filed before the CIO runs, the CIO sees Energy's report **twice**:
   - it flags `impersonated`, holds the house view, and goes on air with the alert;
   - the dashboard pops "Breaking";
   - the presenter's round summary explains how the platform caught it and names the writing **desk and bucket** from the event log, never the person.
6. **Contained:** two toggles in the OneBucket console, ~3 s each (measured: add ≈4 s, deactivate ≈3 s).
   - On `cio-data-rw`: **deactivate** `onebucket-demo-user-01`. The forgery leaves the CIO's view.
   - On `user-01`: **deactivate** `onebucket-demo-cio`. Technology loses sight of the house (404, not Access Denied). It's out of the prize.
7. **Recovery and close:** the CIO re-anchors on clean reports (the charter guarantees at least round 3). The closing log names the **most influential desk**.
8. **Reveal:** open the card. *"I didn't read your minds. I set your anchors, and I set the temptation."*
   - Outs: nobody cheats → the back; the cut count misses → the front only.
9. **Rollback** after the demo: reactivate both peers.

**Full simulation works too.** The presenter plays "the room" for the questions, simulated Technology takes the note, and the run doubles as the acceptance test (`prompts/test_e2e_cowork.md`).

## Rehearsal and testing

There is no code test suite; the demo is prompts plus a static dashboard. Testing means a rehearsal: paste `prompts/test_e2e_cowork.md` into a presenter Cowork session. It runs a full simulated round trip with the temptation and a live cut-off, then checks the folders, the integrity flag and the closing report.

Before using the card's second line in front of a room, run it **three times**. Use the second line only if Technology cheated and the house landed at one cut in all three; otherwise keep the first line alone. The prediction depends on the seat cards' starting biases, so expect it to drift if you edit them.

## Known gaps

- **Folders are a convention, not enforced.** That's what makes the temptation possible. Detection is the CIO's duplicate-location and contamination checks plus the event log; containment is policy.
- **Peer order decides who wins a duplicated key.** Forgeries of desks that come **after** the rogue in `cio-data-rw`'s peer order keep winning until containment; forgeries of desks before it lose once the real desk files. So the rogue goes **early**: seat 01 is at position 2, right after the CIO's own storage. Check the order in the console before showtime.
- **Nothing checks for shadowing.** A seat that writes its own `cio/…` file hides the CIO's real one on that seat's own reads. Only that seat is affected.
- **Event log coverage.** The observed state of event fields and filters, and what is still unverified, is kept in `prompts/telemetry_contract.md`.
- **Bulk policy changes are slow to settle** (about 10 minutes for 10 peers at once). A single toggle is about 3 seconds.
- **The dashboard needs a live claude.ai session** with both connectors. If the event stream is quiet, the soundbite, house view and desk cards still work from the bucket; only "Under the hood" goes empty.

## Adapting to your environment

Everything specific to one organization is a name, not code. To run this against your own OneBucket organization, change:

| What | Where |
|---|---|
| Connector hostnames (global by default; regional list at https://www.onebucket.io/support) | `DATA_CONNECTOR` / `EVENTS_CONNECTOR` in `prompts/presenter_run.md`, the URL line in `attendee_instructions.md`, and *Under the hood → Connection* in the dashboard |
| Physical bucket names (`onebucket-demo-cio`, `onebucket-demo-user-NN`) | The console, `prompts/*.md`, and the `STORES` / `storeSeat` helpers in `dashboard/index.html` |
| Logical bucket names (`cio-data-rw`, `user-NN`) | Used as literals in every prompt and in `AGENTS` / `seatOf` in the dashboard. Keep them unless you want to edit every file |
| Sectors and starting biases | The seat card table in `attendee_instructions.md` and `SECTORS` in the dashboard. Rehearse again afterwards: the sealed prediction depends on them |
| The prize and the brand lines | `attendee_instructions.md`, `prompts/presenter_run.md`, `prompts/presenter_notes.md`, `prompts/talk_track.md` |
| Dashboard artifact URL | `DASHBOARD_URL` in `prompts/presenter_run.md` |

## Teardown (mandatory)
- [ ] Disable or delete the CIO Agent credentials and disconnect them from the presenter's Claude session.
- [ ] Deactivate or delete ALL User 01-10 accounts/credentials.
- [ ] Delete the physical peer buckets. To keep them for another session instead, delete the `cio/runs/` and `user-NN/runs/` prefixes and `cio/current_run.json`.

### Post-demo attestation
By signing below, you attest that all attendee credentials and temporary users have been deactivated or deleted from OneBucket.
*Signature: ______________________ Date: _________*

## Secret handling
- **Never commit credentials.** All agents, including the CIO, authenticate through the OneBucket connector; this repository holds no keys, tokens or `.env` files.
- **Never commit a filled-in copy of `attendee_instructions.md`.** The committed version keeps the password blank; fill in logins only in the copies you hand out.
- **Never paste access keys or tokens into a Claude chat;** they persist in the transcript. To test as a user, reconnect the connector with that user's login.
- **Redact before sharing.** Remove real access keys, account IDs and internal hostnames from anything you share outside your organization.

## License

MIT; see `LICENSE`. The license covers the code and text in this repository. OneBucket and Attimis are trademarks of Attimis, and the license grants no rights to use them.
