# Gate Protocol

The gate is the only mechanism preventing Phase 3 code from being written against unapproved assumptions. Treat it as a state machine, not a politeness ritual.

## Valid approval tokens

Count as approval only when they are the direct reply to a gate card:

- `approve`, `approved`, `lgtm`, `looks good`, `go`, `proceed`, `continue`, `yes`, `ok`
- Any explicit statement of approval in the user's own language, e.g. `setuju`, `lanjut`, `gas`

Everything else is **not** approval:

- Silence, a new unrelated question, an emoji, a compliment
- "Looks reasonable, but..." → this is a revision request
- Approval of *part* of the document ("the ERD is fine") → still a revision; confirm what changes
- A request that presupposes the next phase ("ok now write the migration") → treat as **bypass request**, see below

## Revision flow

1. Reopen the *same* phase. Do not advance.
2. Apply the change to the phase document. Keep the previously approved content intact unless the revision supersedes it.
3. Update `STATE.md`: gate back to `awaiting approval`, increment the revision counter.
4. Print the gate card again with a one-line `Changed:` note listing what moved. Do not re-print the whole document unless the user asks.

Multiple revisions in a row are normal and cheap. Advancing on an unapproved design is not.

## Bypass flow

Bypass requires a literal token: `bypass 1`, `bypass 2`, `bypass 3`, `bypass 4`.

1. Confirm once, in one line: what is being skipped and the risk in concrete terms ("bypass 2 skips the auth rules per endpoint — I'll have to invent them during implementation").
2. On confirmation, write `STATE.md` entry: phase, reason, timestamp.
3. Generate a stub document at the expected path with `status: bypassed` and a list of what was not decided.
4. Continue from the requested phase.
5. Carry the skipped decisions forward as explicit assumptions in the next phase document.

Never infer a bypass from impatience. "Just code it" is not `bypass 2`.

## State transitions in `STATE.md`

| Event | Update |
|---|---|
| Phase starts | `phase: <n>`, gate row `in progress` |
| Document written | gate row `awaiting approval`, link the file |
| Project docs written or updated | add rows to the `Project docs` table with today's date |
| Approval received | gate row `approved`, `approved_on: <date>`, next gate `pending` |
| Revision received | gate row `awaiting approval`, revision counter +1 |
| Bypass | gate row `bypassed` + reason |
| Phase 4 closed | `status: complete` |

`STATE.md` must be accurate enough that a fresh session can resume by reading it alone. Update it before printing the gate card, not after.

## Gate card rules

- Print it as a fenced block, last thing in the turn.
- `Produced:` lists every file written or updated in the phase — the phase document plus any project docs (`docs/architecture.md`, `docs/deployment.md`, `docs/README.md`). Omit files you did not touch.
- Never print a gate card and then continue working — no tool calls, no extra paragraphs.
- Never print two gate cards in one turn.
- Never announce the next phase's content in the card. The card asks for approval; it does not preview the work.
- If a phase produced no document, do not print the card. Finish the document first.

## Approval that arrives out of order

If the user approves gate 2 while gate 1 is still open, do not skip ahead. Reply in one sentence: gate 1 is still open, ask whether they want to approve it too. If they confirm, record both approvals with their timestamps.

## Escalation

If the user repeatedly pushes for code during Phase 1 or 2 and explicitly refuses to ask for a bypass, state once that you will proceed with the current assumptions recorded as `assumed` in the document, then continue with the workflow's Phase 1/2 output — never with implementation code. Do not argue twice.
