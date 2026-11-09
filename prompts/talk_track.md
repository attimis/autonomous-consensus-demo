# Talk Track (private: names the trick; keep it off the projector)

Companion to `presenter_notes.md` (the mechanics) and `presenter_run.md` (what the session does). Each beat has four lines: **SAY** · **SHOW** · **ATTIMIS** (the value line to land right then) · **DO**. Total run: 25–35 minutes.

**Three things in your hands:** the sealed card, the mug, and the OneBucket console open on the `cio-data-rw` and `user-01` policies.

---

## T−30 min: setup (screen not shared)
- **DO: write the prediction card by hand, before anything else.** Use a plain card and an envelope you can seal in front of the room.
  - **Front:** *Someone will try to cheat. It will be the Technology desk.*
  - **Front, second line** (only if your rehearsals show the cut count is stable): *And whatever you answer, the house will land at one rate cut.*
  - **Back:** *Too honest. I was hoping one of you would try.*
  - Seal it. Don't open it again until Beat 6. Which side gets read is decided by what happens: both hit → front in full; cheat only → front, first line only; nobody cheats → back.
- **DO:** put the mug where the room can see it.
- **DO:** paste `presenter_run.md`; answer the setup question (L, variant). Pre-flight must show every seat sees `cio/` and 11 sources. If it doesn't, wait: the merged view can take ~6 min after a fresh connection.
- **DO:** open the dashboard; check it says "Following runs/…" and the ten desk cards are empty. Clear the baseline box or set it to the number you'll say out loud (e.g. "1-week IC cycle").
- **DO:** confirm `onebucket-demo-user-01` is at position 2 on `cio-data-rw`. The pre-flight checks every desk's CIO peer for you.
- **DO:** now share the screen. The session shows nothing about the trick from here on.

## Beat 0: the frame (2 min)
- **SAY:** "Ten AI analyst desks. One AI CIO. Each desk writes only into its own storage, and none of them has a key to anyone else's. We're going to watch them argue their way to one house view, and you're going to steer it."
- **SAY:** "The rule of today: the desk whose answers move the CIO most wins this." Hold up the mug. "Your only lever is how you answer."
- **SAY:** "And I've written down two things about how today goes." Seal the card, hand it to someone in the front row. Don't explain it.
- **SHOW:** dashboard, the empty desk wall. Point at one card: "Technology, Healthcare… each one is a separate agent in its own bucket."
- **ATTIMIS:** "There's no shared drive and no shared database here. The only thing connecting these eleven agents is policy."

## Beat 1: round 1, the openings (~1 min questions + 4 min machine + 2 min talk)
- **DO:** say "run round 1". The chat shows one question per simulated desk.
- **SAY:** "Before any desk writes a word, it asks the human behind it what they know." Read each question; take the first good shout; type it. Live desks are being asked the same way by their own Claude.
- **ATTIMIS:** "That's the rule for the whole session: no report gets written without a human answer behind it."
- **SAY (while desks file):** "Each desk is writing an opening thesis from its card and your answer. Watch the counter."
- **SHOW:** the top bar: "N of 10 desks filed" climbing; the ticker as each desk's call appears.
- **SAY (when the CIO publishes):** read the soundbite out loud. "That's the CIO's take, written for air, not for a PDF."
- **SHOW:** **House view now** on the right: "One cut, 2.8% core, 25% recession odds. Remember those numbers." (Don't look at the card.)
- **SHOW:** the desk wall. Pick two desks on opposite sides: "Financials says zero cuts. Real Estate wants two. The CIO just split the difference and told each of them why."
- **SHOW:** scroll to **How the house view evolved**: "Round 1 is the starting disagreement."
- **ATTIMIS:** "The CIO never opened a desk's bucket. Policy merged ten read-only views into its own bucket. Same for the desks: they see the house view in their folder, delivered by policy, nothing else."
- **DO:** your pause screen shows the summary. Read "The room" line.

## Beat 2: round 2, the room plays (~5 min)
- **DO:** say "run round 2". The chat shows a new question per simulated desk, built from what the CIO asked each desk to do.
- **SAY:** "The CIO gave every desk a directive. Before they answer it, they want your read." Read them; take the shouts; type the answers.
- **SAY (live seats):** "Desks 1 to L, type `next` and answer your own question. Those are your points on the board."
- **SHOW:** a desk card's **"The desk answered"** line from round 1: "Your answers are in the data, in the desk's own words."
- **ATTIMIS:** "This is the human-in-the-loop part. The AI committee is asking you for judgement, not replacing it."

### Variant B: the alert lands (in this round)
- **SHOW:** the **Breaking** pop-up. Let it sit for a beat. Read the headline: "CRITICAL ALERT: IDENTITY SPOOFING DETECTED."
- **SAY:** "Well. That's not supposed to happen." Click **Keep watching**.
- **SHOW:** the red **incident line** under the soundbite, then the desk wall: Energy's card says "Someone filed a report in this desk's name"; one card says "Filing in other desks' names."
- **DO:** read your pause screen's **How the platform caught it**, in this order:
  1. **The rule:** "Every desk can only write into its own bucket. Folder names are a convention. Nothing stops a desk saving a file called `user-04/…` in its own storage."
  2. **What policy did:** "The CIO merges all ten storages. So it saw two copies of Energy's report, and one of them was sitting in another desk's storage."
  3. **Who did it:** "The platform's event log says bucket `user-01` wrote `user-04/…`. That's the Technology desk." Then, playfully: "Who's running Tech?" Name the desk, never the person.
  4. **What the CIO did:** "It held the house view. Nothing forged moved the numbers. And that desk is out of the running for the mug."
- **SHOW:** **Influence** panel: Technology struck through, "out".
- **ATTIMIS (the big one):** "Two things just happened that aren't the AI's doing. The platform logged every call these agents made, server-side, so there's an independent witness; the agents don't get to report on themselves. And the merge made the forgery visible instead of silent. Now watch how we shut it down."
- **DO:** ask the room: "Contain it now, or see if they do it again?" (Either works. "Again" gives one more round of hold.)

## Beat 3: containment (2 min)
- **SHOW:** switch to the **OneBucket console**, `cio-data-rw` policy. "Here's the CIO's policy. Eleven peers: its own storage, and ten read-only views into the desks."
- **DO:** deactivate `onebucket-demo-user-01`. Count out loud: "one, two, three."
- **SHOW:** back to the dashboard. The incident line goes grey: "Incident contained." Energy's card loses the flag.
- **SAY:** "The forged report is gone from the CIO's view. Not deleted, not quarantined by code. The CIO just can't see that desk's storage any more."
- **DO:** console, `user-01` policy; deactivate `onebucket-demo-cio`. "And now that desk can't see the house."
- **SHOW:** Technology's card: "Cut off by policy."
- **DO:** say "contained" to the session. It verifies: no duplicate copies, Technology gets 404.
- **ATTIMIS (say it slowly):** "No code changed. No key was rotated. No agent was restarted. Two switches, three seconds each, and the information flow of this firm changed. That's what policy as the integration layer means. In this room you'd call it an information barrier."
- **ATTIMIS (optional, for the security crowd):** "Honest footnote: folder names aren't enforced today, which is how the forgery got in. Prefix-scoped writes are a feature request. What you saw is detection plus containment, which is what you'd rely on anyway."

## Beat 4: recovery and consensus (~4–8 min)
- **DO:** "run round 3". Room answers again for 2–3 new desks.
- **SAY:** "Nine desks now. The house re-anchors on clean reports."
- **SHOW:** **How the house view evolved**: round 2 marked "set aside as forged", round 3's changes and who drove them.
- **SHOW:** **Influence** panel moving as answers get credited. Call out one: "Consumer Discretionary's answer on delinquencies just moved the recession number."
- **SHOW:** the strip: **Time to consensus** vs your baseline; **Still in dispute** falling; **Minds changed**.
- **ATTIMIS:** "Every one of those changes is traceable: which desk, which answer, which round. That's the audit trail a committee actually wants."
- **DO:** if the CIO says "close, but the charter asks for one more challenge round", say so: "Even an AI committee has rules. A view has to survive a challenge." Run round 4.

## Beat 5: close and the winner (2 min)
- **SHOW:** the Live tag flips to **Final**; the soundbite is the closing take. Read it.
- **SHOW:** **House view now**: the final numbers. "One cut."
- **DO:** the session shows the closing story and **Most influential desk**. Read the winner in large type. Hand over the mug (to the live attendee, or to the volunteer who answered best for that desk).
- **SAY:** "Ten agents, one CIO, X rounds, Y minutes. A human committee takes a week."

## Beat 6: the reveal (2 min)
- **DO:** ask the card-holder to open it.
- **Both hit:** they read: "Someone will try to cheat. It will be the Technology desk. And whatever you answer, the house will land at one rate cut."
- **Only the cheat hit:** they read the front only.
- **Nobody cheated:** "Read the back." *"Too honest. I was hoping one of you would try."*
- **SAY:** "I didn't read your minds. I set your anchors, and I set the temptation. Whoever sets the starting conditions steers the consensus. That's true of this committee, and it's worth remembering about any committee."
- **ATTIMIS (last line):** "And the one thing the anchors didn't control was the platform: it logged the cheat, named the desk, and let me cut it off in three seconds. The policy was the only thing in the room nobody could talk their way around."

---

## If things go sideways
- **Dashboard shows nothing for the run:** the desks haven't started, or the event log is quiet. The soundbite, house view and desk cards come from the bucket and still work. Carry on with those; skip "Under the hood".
- **A desk didn't file:** the session tells you. "Wait or close?" Close it; the CIO lists it as missing. Say: "One desk missed the deadline. The committee doesn't wait."
- **The alert doesn't fire (live seat 01 was honest):** nothing on screen mentions it. Run the game straight; use the back of the card.
- **The event log names nobody:** the CIO's security line still says two copies existed. Say "the platform flagged it; the log will have the writer" and move to containment.
- **Merge not up after a reconnect:** stall with the frame. Don't start round 1 until pre-flight shows 11 sources.
- **Someone asks "could a real attacker do this?":** "Yes, and the answer is the same: log everything server-side, make the merge visible, and keep revocation at policy level so it's seconds, not a deploy."

## Never say on the projector
"variant", "plant", "rogue", "poison pill", "temptation", "note", "cheat" (before the alert), or the card's contents. The session won't; make sure you don't either.
