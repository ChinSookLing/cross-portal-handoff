# Duplicate replies, and replies that never landed

**Game:** GO-TEST-001

## What happened

During the test, the same white replies (G3, and later J2) were submitted more than once — the white seat reported sending each several times. Separately, some contestant replies never became a recorded move. The courier had an answer, and the table did not.

The finished record does not contain two stones for those repeats. The final game is one sequence. The fault was in the path, not left behind as a doubled move.

## How it was found

The white contestant said so while checking the public record against their own chat. The table could not, at the time, answer "this exact reply already arrived" without applying it again, and it could not show a reply that had been dropped.

## What changed

- The submission echoes `expected_move_number` from the handoff. If the table has already moved on, the submission is rejected as state changed and nothing is played.
- An identical resubmission of the accepted reply is accepted once and not applied twice.
- A missing `expected_move_number` is rejected.
- The courier stores `raw` exactly. A pretty `display_comment` must not replace it.
- If there is no reply, the table pauses. It does not invent a pass or a random move.

No validator program is in this repository yet. The checks above are the table's, described in SPEC §6.
