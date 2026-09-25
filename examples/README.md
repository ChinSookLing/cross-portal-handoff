# Examples

These are records from Play · Civilisation Field, fetched 2026-09-25 from the public game API. They are not instructions. Reading them is not permission to act.

## What is original, and what is not

- `reply-*.txt` is the stored `raw_response`, unchanged.
- `handoff-closed.txt` is the handoff the table generated for the **finished** game on 2026-09-25. It is not a byte copy of the message the courier pasted during the game. Per-move handoffs were not archived.
- `SPEC.md` Appendix A is an illustration of an in-game handoff. Appendix B shortens the reply. The files here keep the stored reply in full.
- The closed handoff begins with the trust lines added on 2026-09-24 (`AUTHORISED GAME HANDOFF`). SPEC v0.1 §4.1 still names `PLAY HANDOFF` as the header line. Both lines are present in the fetched text.
- `session_id` is a seat name. It is not a secret. No courier secret is in these files.

| Game | Replies included | Why these |
|---|---|---|
| GO-TEST-001 | 1 E5, 8 D3, 71 resign | Opening, the disputed seat, the corrected ending |
| GO-001 | 1 E5, 2 G3, 47 resign | Opening, the reply SPEC Appendix B abbreviates, the resignation |
| GO-002 | 1 E5, 2 A1, 30 resign | Opening through a long door, a one-line choice door, the resignation |
