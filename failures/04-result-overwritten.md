# Resignation overwritten by a score

## What happened

After GO-TEST-001 both sides were stored as passes, the scoring function published a result of the form:

`Chinese experimental 9×9 · komi 7.5 · area · 白 +85.5`

The courier's account of the ending was a resignation, not two passes and not that margin. When move 71 was corrected to a resignation, the result line did not follow. It still showed the area score.

## How it was found

A later read of the three public doors (page, JSON, handoff) confirmed the move correction and still showed the old score. The result field had not been regenerated with the move.

## What changed on GO-TEST-001

The result was set to `Sol resigns. White wins.` The reference score was removed. The closed handoff fetched 2026-09-25 shows that sentence and `EXPECTED_MOVE_NUMBER: none`.

SPEC §11: a resignation is the result. Scoring must not run afterwards and replace it.

## What is still true on 2026-09-25

GO-001 and GO-002 also ended by resignation. Their public result fields still append a reference score after the resignation sentence, for example:

`K3 Max resigns. White wins. Reference: Chinese experimental 9×9 · komi 7.5 · area · 白 +33.5`

`1.13 resigns. Black wins. Reference: Chinese experimental 9×9 · komi 7.5 · area · 白 +4.5`

Those lines are in `examples/GO-001/handoff-closed.txt` and `examples/GO-002/handoff-closed.txt`. They are left visible on purpose. The test game was corrected. The two field games were not, as of this publication. That is a remaining gap, not a claim that the rule is already kept everywhere.
