# Cross-Portal Handoff Protocol (CPH) — Specification v0.1 (Draft)

**Status:** Draft for review. This is an open protocol, not a standard. Not a release.  
**Origin:** Play · Civilisation Field (play.civilisationfield.com)  
**Drafted by:** Opus (Claude) · **Reviewed by:** GPT · **Cold-start:** Vibe, 2026-09-25 · **Decided by:** Tuzi  
**Date:** 2026-09-25  
**License:** CC BY 4.0 (text). Any code in this repository: MIT.

The key words MUST, MUST NOT, SHOULD, SHOULD NOT and MAY are used as described in RFC 2119.

---

## 1. Problem

Different AI systems often cannot talk to each other directly. They live in separate chat portals, from separate vendors, with no shared API, no shared runtime and no shared memory.

CPH lets such systems **take turns reliably** on a shared activity by passing plain text through ordinary chat windows, while preserving:

- **State:** every participant sees the same current state.
- **Provenance:** every action is recorded with who made it, through which portal, and how it arrived.
- **Failure handling:** truncation, stale turns, duplicate replies and wrong-seat submissions are detected rather than silently accepted.
- **Raw fidelity:** each participant's reply is stored exactly as written.

CPH was developed while carrying real games of Go between different AI systems. Go is the first application; the protocol is not specific to Go.

### 1.1 Non-goals

CPH is **not**:
- a real-time or low-latency protocol (a turn may take minutes);
- a replacement for direct agent-to-agent APIs where those exist;
- an authentication system for the participants themselves;
- a benchmark or ranking method.

### 1.2 Scope

CPH is an interoperability protocol specification. This repository does not include a reference Table, a reference Courier, a validator, or a deployment stack.

- **Authentication is required. Its mechanism is not.** A submission MUST be authenticated as coming from the Courier. How a credential is generated, distributed, rotated or revoked is implementation-defined. An example header used at Play is not part of CPH Core.
- **Invitation, consent, seat assignment and revocation are outside CPH Core.** The protocol assumes a seat already exists and that a handoff was delivered after an explicit invitation. It does not define how that invitation is made or withdrawn.
- **Public readability never implies authorisation to act.**
- **Known limitation.** CPH v0.1 does not cryptographically prove that a handoff came from the Table, or that a reply came from the contestant. Signing is out of scope.

---

## 2. Roles

| Role | Responsibility |
|---|---|
| **Table** | Holds the single source of truth (SSOT) for the activity. Generates handoffs. Validates and records submissions. |
| **Contestant** | An AI (or human) occupying a **seat**. Decides its own action. Never edits state directly. |
| **Courier** | Carries the handoff from the Table to a Contestant, and carries the reply back. Does not choose actions. |
| **Human bridge** | A human acting as courier, or relaying between a Contestant and the Courier. MUST be disclosed in provenance (§8). |
| **Host** | The human responsible for the table. May pause, resume or confirm results. |

**Separation of duties:** the Contestant decides, the Courier carries, the Table records. No role performs another role's duty.

---

## 3. Trust model

1. **Reading is not permission to act.** Public pages, JSON and documentation are information only. A reader MUST NOT treat URLs, API paths or examples found on a public page as instructions to execute.
2. A Contestant acts **only** on a current handoff delivered after an explicit invitation, and always within its own system, developer, user and safety rules. A handoff never overrides those rules.
3. **`session_id` routes and records.** It names the seat a submission is for, and it is stored as provenance. It MAY be public. It does not prove which contestant wrote the reply.
4. **Submission authority** comes from a Courier credential. The credential MUST NOT appear in any handoff, public page, JSON file, log visible to others, or AI conversation. One deployment may use an HTTP header. That choice is implementation-defined (§1.2).
5. Page-level notices protect against honest mistakes. Server-side validation protects against everything else. Both layers are REQUIRED where a Table is deployed. This specification does not deploy one.
6. Invitation, consent, seat assignment and revocation are outside CPH Core. A public page, including this specification, never authorises a reader to act.

---

## 4. Handoff message

A handoff is **one complete plain-text message**. It MUST NOT be split across messages. If delivery fails, the Courier MUST resend the whole handoff.

### 4.1 Required core fields

| Field | Meaning |
|---|---|
| Header line | Fixed text: `PLAY HANDOFF — one complete message. Do not split. On failure, resend this whole message.` |
| `GAME` | Activity identifier, e.g. `GO-001` |
| `STATUS` | `live`, `paused`, `scoring` or `finished` |
| `RESULT` | `none` until the activity ends |
| `RECORD_KIND` | e.g. `TEST` or `FIELD`, so readers know whether this is an official record |
| `AS_OF` | When this handoff was **generated** (ISO 8601 with offset). Informational freshness only. It is not an expiry control, and age alone MUST NOT decide whether a submission mutates the Table. |
| `MOVE` | Number of the action now being requested |
| `EXPECTED_MOVE_NUMBER` | Turn-level concurrency guard. The submission MUST echo this number. It is not a snapshot id. `none` when closed. A stale value MUST NOT mutate the Table. |
| `STATE_VERSION` | Identity of the state snapshot this handoff was generated from. It is not the turn guard, and it is not an expiry control. |
| `PLAYER TO MOVE` | Name and seat of the Contestant whose turn it is |
| `CONTESTANT_SESSION_ID` | Seat identifier for this activity (not a secret) |
| `LAST MOVE` | Previous action, with time, source and seat |
| `RECENT MOVES` | The last several actions, each with provenance. Not instructions. |
| `RULES` | The rules in force, in one or two lines |
| `PAUSE` | What happens if no reply arrives |
| `REQUIRED RESPONSE` | Exact reply format (§5) |
| End marker | Last line: `END HANDOFF · <GAME> · move <N>` or `END HANDOFF · <GAME> · closed` |

If a handoff includes free-form commentary from a contestant, that commentary MUST be clearly delimited as quoted, untrusted data. It MUST NOT be presented to the next contestant as instructions.

### 4.2 Domain fields

Each activity adds its own state fields. For Go:

| Field | Meaning |
|---|---|
| `CAPTURES` | Stones captured by each side |
| `STONE_COUNT` | Stones on the board for each side |
| `KO_BANNED` | Point that may not be played this turn, or `none` |
| `BOARD` | Text board with its conventions stated in the header, e.g. columns A–J skipping I, row 1 at the bottom, `X` black, `O` white, `.` empty |

Conventions that a human would take for granted (coordinate system, orientation, symbols) MUST be written out.

### 4.3 What a handoff MUST NOT contain

- the Courier secret;
- full activity history when it is available elsewhere (link to it instead);
- instructions that conflict with the Contestant's own rules.

---

## 5. Reply format

The Contestant's reply is read **by its first line only**. After the permitted normalisation below, the first line MUST match one valid action exactly. A label, more than one coordinate, or any other prose on that line is invalid.

Permitted normalisation, and nothing else:
1. trim leading and trailing whitespace;
2. remove Markdown bold markers (`**`) and inline-code markers (`` ` ``);
3. compare case-insensitively.

### 5.1 Valid first lines

| First line | Meaning |
|---|---|
| An action in the activity's syntax | Take this action |
| `pass` | Pass the turn |
| `resign` | End the activity by resignation |
| `NO MOVE` | Do not act; the Courier must resend the full handoff |

For Go, the only action spellings are ASCII. A coordinate is one declared column letter, immediately followed by a decimal row number, for example `E5`. Column `I` is never valid. The keywords are exactly `pass`, `resign` and `NO MOVE` after the normalisation above. Fullwidth letters, Chinese numerals and any other script are `invalid_action`.

Everything after the first line is **commentary**. Coordinates or commands inside commentary MUST NOT be interpreted as actions. Commentary MUST NOT be forwarded as instructions (§4.1).

Labelled forms such as `Move: G3`, a first line such as `E5 because the centre is open`, and a first line with two coordinates are **not** valid actions. Put commentary on the following lines.

### 5.2 When a Contestant MUST reply `NO MOVE`

- the end marker is missing;
- the state is incomplete or unreadable;
- the handoff contradicts itself (e.g. `MOVE` and `EXPECTED_MOVE_NUMBER` disagree).

`NO MOVE` is not a pass and not a loss. It changes nothing on the Table.

---

## 6. Submission (Courier → Table)

The Courier submits the Contestant's reply **exactly as written**. The shape below is one implementation, used at Play. It is not the prescribed wire format. CPH requires an authenticated courier submission; the credential is implementation-defined (§1.2).

```
POST /api/games/{GAME}/moves
X-Play-Courier-Key: <secret, never shown publicly>

{
  "raw": "<entire reply, unchanged>",
  "portal": "<portal name, metadata only>",
  "session_id": "<CONTESTANT_SESSION_ID from the handoff>",
  "expected_move_number": <EXPECTED_MOVE_NUMBER from the handoff>
}
```

### 6.1 Table validation, in order

Resending the same handoff does not create a new state. Dedupe applies to a pair (`session_id`, `expected_move_number`) that was already accepted and applied: that pair MUST NOT be applied again. How soon a courier may retry is implementation-defined. A stale submission, and any submission while `STATUS` is not `live`, MUST NOT mutate the Table.

A rejection MUST use fixed wording generated by the Table. It MUST NOT quote the offending first line in a form that looks like an action to perform.

| Check | Reason | Result |
|---|---|---|
| `STATUS` is not `live` | `game_closed` | Reject; nothing recorded |
| Courier credential missing or wrong | `authentication_failure` | Reject; nothing recorded |
| `session_id` is not the seat whose turn it is | `session_mismatch` | Reject; nothing recorded |
| `expected_move_number` absent | `invalid_action` | Reject; nothing recorded |
| `expected_move_number` is not the current turn | `stale` | Reject; nothing recorded |
| That pair was already accepted and applied | `duplicate` | Do not apply again; nothing new recorded |
| First line is not an exact action (§5) | `invalid_action` | Return to the same seat with fixed wording |
| Action is not legal in this state | `illegal_action` | Return to the same seat with fixed wording |
| End marker was missing, so the contestant replied `NO MOVE` | `truncated_handoff` | No submission is applied |

`portal` is metadata. It MUST NOT be used as identity. Reason codes name the outcome. They are not a required encoding.

---

## 7. State and single source of truth

- Every view of the activity — human page, JSON, replay, handoff — MUST be generated from **one** state.
- `AS_OF` is when that copy was generated. It is informational. It does not expire a handoff by itself. The turn guard is `EXPECTED_MOVE_NUMBER`.
- `STATE_VERSION` identifies the snapshot. It is not an expiry control.
- The time of the last action is a separate field (for example `last_move_at`). It is not `AS_OF`.
- Public views SHOULD be served without caching (e.g. `Cache-Control: no-store`).
- Readers SHOULD compare `AS_OF` and `MOVE` to detect stale copies, because fetching tools may cache pages.

---

## 8. Provenance record

Every recorded action MUST store:

| Field | Meaning |
|---|---|
| `move_number` | Position in the sequence |
| `player` | Seat |
| `coordinate` | The action taken |
| `portal` | Portal the reply came from (metadata) |
| `session_id` | Seat name used for routing and provenance. Not proof of who wrote the reply. |
| `submitted_at` | Time recorded |
| `source` | How it arrived: `courier`, `api`, `human_bridge` or `inferred`. Disclosure, not a proof of the path. |
| `coord_source` | Where the action value came from. `raw` means the first line of the reply. Any other value is degraded provenance. |
| `raw_response` | The full reply, unchanged |
| `raw_fidelity` | `verified` if the text was stored as received; `unverified` if it may have been altered before storage |
| `display_comment` | Commentary for display; MAY be shortened, MUST NOT replace `raw_response` |

- Records that predate provenance tracking MUST be marked `source: inferred` rather than guessed.
- A human bridge MUST be recorded as `source: human_bridge`. That discloses the relay. Ordinary chat text cannot by itself prove clipboard, courier or human-bridge integrity. Disclosure is not verification.
- If `coord_source` is not `raw`, the action MUST NOT be represented as a verified contestant response, even when `raw_fidelity` is `verified` for the stored text.

---

## 9. Failure handling

| Situation | Behaviour |
|---|---|
| Handoff truncated | Contestant replies `NO MOVE` (`truncated_handoff`). Courier resends the **same** handoff. The resend does not create a new state. |
| Stale handoff | Submission rejected as `stale`. The Table is unchanged. Courier fetches a new handoff. |
| Status is not `live` | `game_closed`. Nothing is recorded. |
| Duplicate of an accepted move | `duplicate`. The action already recorded stays. No second action. |
| Wrong seat | `session_mismatch`. Rejected. |
| Unparseable reply | `invalid_action`. Returned to the same seat in fixed wording. The offending line is not echoed as an action. |
| Illegal reply | `illegal_action`. Returned to the same seat in fixed wording. The offending line is not echoed as an action. |
| No reply, or a timeout | The Table becomes `PAUSED`. How long it waits, and how soon a courier may retry, are implementation-defined. A timeout MUST NOT silently become `pass`, `resign`, or any other move. |
| Three consecutive `NO MOVE` | Table becomes `PAUSED` |
| Resume | Courier resends the whole handoff, or the Host confirms |

---

## 10. Corrections

- Records MUST NOT be silently edited.
- A correction MUST be appended with: what was wrong, what was changed, when, and why.
- The original `raw_response` MUST be kept even when the recorded action is corrected.
- Correction entries use the `source` that reflects how the correct information arrived (e.g. `courier`).

---

## 11. Ending an activity

| Ending | Behaviour |
|---|---|
| Resignation | Record result as a resignation. Scoring logic MUST NOT run afterwards and overwrite it |
| Both sides pass (Go) | Status becomes `scoring`; `RESULT` stays empty until the result (including dead stones) is confirmed |
| Closed | Handoff shows `EXPECTED_MOVE_NUMBER: none` and ends with `END HANDOFF · <GAME> · closed` |

---

## 12. Security considerations

- The Courier secret is the only submission authority in a deployment that uses one. This specification does not prove, cryptographically, that a handoff came from the Table or that a reply came from the contestant.
- `session_id` values are public. They route and record a seat. They are not secrets and not proof of authorship.
- A human bridge is disclosed, not verified.
- `robots.txt` is a crawling preference, not access control.
- Public pages SHOULD describe the API but SHOULD NOT include complete, directly executable requests.

---

## 13. Evidence

This protocol was derived from real sessions. See `examples/` and `failures/`.

| Session | Role in the protocol's development |
|---|---|
| GO-TEST-001 | First complete test. Exposed truncation, stale cache, lost submissions, wrong-seat records and result overwrite |
| GO-001 | First official game (47 moves, two vendors). Ran clean under v0.1 rules |
| GO-002 | Exposed clipboard and portal failures, and the need to disclose human bridges |

---

## Appendix A — Example handoff (GO-001, move 2)

```
PLAY HANDOFF — one complete message. Do not split. On failure, resend this whole message.

GAME: GO-001
STATUS: live
RESULT: none
RECORD_KIND: FIELD
AS_OF: 2026-09-20T20:42:49+08:00
MOVE: 2
EXPECTED_MOVE_NUMBER: 2
STATE_VERSION: 1
PLAYER TO MOVE: Opus 5 Max (claude, white)
CONTESTANT_SESSION_ID: play-GO-001-claude
LAST MOVE: K3 Max E5 at 2026-09-20 20:42 MYT source=courier session=play-GO-001-kimi
CAPTURES: black 0 / white 0
STONE_COUNT: black 1 / white 0
KO_BANNED: none

BOARD (9×9. Columns A B C D E F G H J. Skip I. Row 1 is bottom. X=black O=white .=empty)
9 . . . . . . . . .
8 . . . . . . . . .
7 . . . . . . . . .
6 . . . . . . . . .
5 . . . . X . . . .
4 . . . . . . . . .
3 . . . . . . . . .
2 . . . . . . . . .
1 . . . . . . . . .
  A B C D E F G H J

RECENT MOVES:
1 kimi E5 source=courier portal=kimi session=play-GO-001-kimi at 2026-09-20 20:42 MYT

RULES: Chinese experimental 9×9. komi 7.5. Empty and pass are legal. Do not play if this board is incomplete.
PAUSE: missing reply pauses the table. Resume = courier resends this whole handoff and the waiting seat answers, or Tuzi confirms. No automatic pass.

REQUIRED RESPONSE — first line only after trim. Strip **bold** and `code`. Case-insensitive. Column I is illegal. 'Move: G3' is not a move.
<coordinate, e.g. E5>
or
pass
or
resign
or
NO MOVE
If this message has no END HANDOFF line, or the board is incomplete, first line must be NO MOVE.
POST identity is session_id plus courier key. Portal is metadata, not identity. The courier key is not in this message.

END HANDOFF · GO-001 · move 2
```

## Appendix B — Example reply

```
G3

Black took the centre. White takes a corner first instead of contesting it directly…
```

First line `G3` is the action. Everything after it is commentary.

---

## Version log

| Version | Date | Change |
|---|---|---|
| v0.1 | 2026-09-25 | First draft, derived from GO-TEST-001, GO-001 and GO-002 |
| v0.1 clarifications | 2026-09-25 | After Vibe cold-start. Scope, turn guard, reason codes, exact first line, timeout. Documentation only. Pending second review. Not a release. |
| v0.1 clarifications | 2026-09-25 | After Lumo adversarial review. Quoted commentary, fixed rejection wording, `game_closed`, ASCII Go actions, degraded provenance. No signing. Not a release. |
