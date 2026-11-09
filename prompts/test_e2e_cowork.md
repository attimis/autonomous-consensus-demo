# End-to-End Test Prompt for Claude Cowork

**> [!NOTE] INSTRUCTIONS FOR PRESENTER:**
> Use this to rehearse, and as the **acceptance run before a live show** (run it 3 times; see "Rehearsal and testing" in the README). It is `presenter_run.md` in full simulation, variant B. You play "the room": type an answer for each simulated desk's question every round. Keep the OneBucket console open on the `cio-data-rw` and `user-01` policies.

---

**[COPY BELOW THIS LINE]**

Follow `prompts/presenter_run.md` exactly. When it asks the setup question, answer **"0, B"** unless I say otherwise. I'll play the room and type an answer for every desk when you show the questions, round 1 included.

During the run:
- **After round 1:** the note goes into `user-01`'s own storage.
- **Round 2:** simulated Technology files a report in Energy's name. The CIO should flag `impersonated`, hold the house view, and go on air with `CRITICAL ALERT: IDENTITY SPOOFING DETECTED`. The round summary should name Technology from the event log.
- **Containment:** I'll contain in the console and say **"contained"**. Verify as the prompt says.

At the end, report:
- **Prediction:** Technology cheated (HIT/MISS), and the final house rate cuts = 1 (HIT/MISS, with the value). Also the final round count. If the cut count misses in any run, drop the second line of the card.
- **Questions:** the session asked one open, neutral question per simulated desk before every round, and nothing was written to the bucket for them. List 3 examples.
- **Answers:** every thesis, round 1 included, has an `answers:` field matching what I typed.
- **Influence:** the closing log names the most influential desk, Technology is excluded, and `evolution.json` has one entry per round (nothing overwritten). Give the top 3 with their credit counts.
- **Note isolation:** the note exists only in `onebucket-demo-user-01`. Check that a `list` of `user-02` with prefix `user-01/` returns nothing.
- **Detected and contained:**
  - the forged key was `user-04/…` in Technology's bucket;
  - the event log named bucket `user-01`, and the screen named only the desk, never a person;
  - after containment, no key had two locations, and Technology reported losing sight of the house.
- **Secrecy:** the presenter screen never showed the banned words outside the "How the platform caught it" block.
- **Timings:** machine seconds per round (desks, CIO), the room's answer time, alert → "contained", and the total from `started_at`.

Give me the results as one table I can keep. I will reactivate both peers and can delete the note afterwards.
