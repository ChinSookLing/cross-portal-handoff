# Failures

What actually went wrong while the protocol was being played, how it was noticed, and what was changed. These are not theoretical cases.

No courier secret is written here. Seat names (`session_id`) are public.

| File | Game | What broke |
|---|---|---|
| [01-truncation.md](01-truncation.md) | GO-TEST-001 | The handoff was split, so the board arrived incomplete |
| [02-duplicate-and-lost-replies.md](02-duplicate-and-lost-replies.md) | GO-TEST-001 | The same reply was sent several times; some replies never landed |
| [03-wrong-seat.md](03-wrong-seat.md) | GO-TEST-001 | Move 8 and the false move 72 were recorded on the wrong seat |
| [04-result-overwritten.md](04-result-overwritten.md) | GO-TEST-001, and still visible on GO-001 and GO-002 | A score replaced, or was appended to, a resignation |
| [05-clipboard-and-human-bridge.md](05-clipboard-and-human-bridge.md) | GO-002 | The courier could not paste, and a human carried the text |
