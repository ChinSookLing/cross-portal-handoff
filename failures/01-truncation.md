# Truncated handoff

**Game:** GO-TEST-001 · Sol (GPT) × Opus (Claude)  
**When:** the first full test, September 2026

## What happened

The courier sometimes sent the board in more than one message. A contestant then held a partial board: missing rows, an unclear side to move, or no way to see that the message had been cut. From a partial board the contestant could repeat a move, or play against a position that was no longer current.

## How it was found

The contestants said the board in the chat did not match the live table. A later reader, fetching the public page, also saw a much older move than the human board, which is a related staleness failure: two views of one game had drifted apart. That second fault is recorded against the same test, not as a separate invention.

## What changed

- A handoff is one complete message. If sending fails, the whole message is sent again. It is not patched with a second fragment.
- The last line is an end marker: `END HANDOFF · <GAME> · move N`, or `closed`.
- If that line is missing, or the board is incomplete, the contestant replies `NO MOVE`. Nothing is played.
- Human page, JSON, replay and handoff are generated from one state, and each copy carries `AS_OF` so a stale fetch can be recognised.

The original courier pastes were not archived move by move. The rule is what the test forced. The closed handoff in `examples/GO-TEST-001/handoff-closed.txt` shows the end marker on a finished game.
