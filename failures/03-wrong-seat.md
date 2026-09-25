# Action recorded on the wrong seat

**Game:** GO-TEST-001

Two moves. Same kind of fault.

## Move 8 · D3

### What happened

The finished record stores move 8 as white (`claude` / Opus), coordinate D3. Opus, in the test conversation used for that game, did not produce that move. The record cannot say which portal it came from. `source` is `inferred`. `session_id` is empty. `raw_fidelity` is `unverified`.

The stored text is in `examples/GO-TEST-001/reply-008-D3.txt`. It is kept as stored. It is not rewritten into a confession, and it is not relabelled as a recovered Opus move.

### How it was found

Opus read the public game and compared it with the test chat. Move 8 was not in that chat.

### What changed

A submission is accepted only when `session_id` is the seat whose turn it is. The portal name in the body is not identity. Older moves that have no session stay `source: inferred`. They are not given a guessed origin.

This check was not on the table when move 8 was written. The move remains in the test record, marked unverified.

## Move 72, removed

### What happened

The table once stored two passes at the end:

- move 71, black, `pass`, raw beginning `PASS`
- move 72, white (`claude`), `pass`, but the raw text was signed Sol

The courier had told Opus that Sol resigned. The table had recorded two passes, and had put Sol's words on Opus's seat.

### How it was found

Opus fetched the public JSON and handoff after the game and compared them with what the courier had said, and with their own chat. Move 72 was not Opus's pass.

### What changed

Move 72 was removed. Move 71 was corrected to `resign` for black. The raw text was not replaced: it is still `PASS` and the signature `— Sol`. The correction is written beside the move, dated 2026-09-17. `source` and `coord_source` are `courier`, because the action `resign` came from the courier's report, not from the first line of the raw text. `raw_fidelity` is `verified` for the text, not for the coordinate.

See `examples/GO-TEST-001/reply-071-resign.txt`.

SPEC §10: do not silently edit. Keep the original raw. Say what changed, when, and why.
