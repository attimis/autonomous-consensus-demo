# Telemetry Contract: the platform logs, agents don't

**Agents write no call logs.** The per-agent `telemetry/rNN.json` ledgers and the CIO's `metrics_rNN.json` are retired (2026-10-01). They were slow: every agent had to write its own log, and the CIO's ledger alone was ~14 KB a round. They were also blind: a live attendee's calls were only visible if the attendee wrote them into its own bucket.

## Source of truth: the events endpoint
OneBucket records every request server-side. The **events** connector (`events.onebucket.io`, or `events.<region>.onebucket.io` for a regional cluster; see https://www.onebucket.io/support) exposes them:

| Tool | Use |
|---|---|
| `list_events(since, until, bucket, action, limit, cursor)` | History, most recent first. Load a run's window. |
| `tail_events(cursor, waitSeconds, limit)` | Long-poll for new events, oldest first. Follow a run live. |
| `get_event(eventId)` | One event. |

## Event shape (observed 2026-10-01)
```json
{"eventId":"…","reqId":"…","action":"s3:GetObject","bucket":"user-03","key":"cio/probe.txt",
 "statusCode":206,"bytes":{"in":0,"out":0},"time":"2026-10-01T12:38:15.870Z","durationMs":417}
```
- **Each request produces 1 front-end event plus 1 per backend storage bucket touched**, all with the same `reqId`.
  - The front-end event has the **logical** bucket and no `endpoint`.
  - Backend events have `endpoint` (e.g. `WASABI`) and the **physical** bucket (`onebucket-demo-cio`, `onebucket-demo-user-03`).
  - Count calls from front-end events. Draw policy routing ("read missed own storage, served from the CIO's") from backend events.
- **Peer cut-off is visible:** after a peer is deactivated, that seat's backend events stop including `onebucket-demo-cio`.

## Observed status of the event log (2026-10-01, one regional cluster)
- ✅ `layer: "gateway" | "backend"` on every event.
- ✅ `accessKey` on gateway events. Backend events carry no caller; join them on `reqId`.
- ✅ Real `bytes.in/out`.
- ✅ Filters: `status` (ok/denied/error), `statusCode`, `accessKey`, `user`.
- ⚠️ No `user` field seen on events yet, so the `user` filter returns nothing. The dashboard tells simulated seats from live ones by comparing each desk's key with the CIO's.
- ❓ Refused storage calls (403) are logged: unverified. The cluster showed no events at all after ~20:00Z on 2026-10-01.
- `locate`'s per-peer HEADs are not logged (minor).

The dashboard (`dashboard/index.html`) is built on the events connector (`list_events` from `started_at`, then `tail_events`). It groups events by `reqId`, counts gateway events, draws routing from backend events, and leaves its own reads out of the counts.
