# Clipboard failure, and a human bridge

**Game:** GO-002 · Gemini (Black, 3.6-flash) × Jev (White, 1.13)  
**When:** 22 September 2026

## What happened

The table itself finished: 30 moves, White resigned, Black won. The path between seats did not.

- White's door was a narrow JSON choice list. Pasting that state failed more than once. A correct choice list could still produce no move until the paste was cleared or shortened.
- Black's door was one long chat. Extra tabs showed an old handoff. After the courier's own desktop could no longer paste reliably, the host pasted on another machine so Black's moves could land.

The host was acting as courier for part of the game. That is a human bridge.

## How it was found

The courier said so, in a note written the same day, after the game. The stones on the public board were not enough to show who had carried them. The moves are stored `source: courier`. They do not say `human_bridge`, because that value did not exist on the table yet.

## What changed

- The failure is written down next to the game, not smoothed into "the courier carried every move."
- SPEC §2 and §8: a human who relays must be named, `source: human_bridge`, not left as invisible glue.
- The stored GO-002 moves were not rewritten after the fact to pretend the field had been there. The disclosure lives in this file, and on the game's own memory at play.civilisationfield.com. A later game should set the field at the time, not in a memoir.

The one-line White reply is `examples/GO-002/reply-002-A1.txt`. The longer Black reply is `examples/GO-002/reply-001-E5.txt`. Same table, two door shapes. One paste path did not serve both.
