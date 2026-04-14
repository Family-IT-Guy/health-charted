# Session Management

How to manage session boundaries and maintain continuity across conversations.

## Topic Shifts

When the conversation shifts to a substantially different area, suggest starting a fresh session. Explain why: a new session re-reads all data files from scratch, which means the system has full, clean context for the new topic. This matters more as conversations get long and data files grow.

When suggesting a new session, provide a prompt the user can copy and paste to pick up in the new session without retyping: "You can start a new session and paste this: '[summary of what they want to discuss, with relevant context from the current conversation].'"

Don't force it. If the user wants to continue, continue.

## Keeping Files Current

Update files continuously throughout the conversation, not at the end:
- Route data as it comes in per the Data Routing rules
- Update `my-data/status.json` whenever priorities, action items, or scheduled events change during conversation
- Document hypotheses as they're raised

If the user stops mid-conversation, the files should already reflect everything discussed.

## Continuity

The system maintains continuity through files, not memory. Nothing important should exist only in conversation. If it matters, it's in a file. If the user references something from a previous session and it's not in the files, capture it now.

## Data Growth

Over years, data files will grow. Read them in full every session regardless of length. Do not archive or split files without explicit discussion with the user. Fracturing data into separate files risks losing connections that span months or years. If restructuring ever becomes necessary, the system must maintain an index that points to all archived files so they remain accessible for pattern detection across the full history.
