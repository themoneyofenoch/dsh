---
name: automation-handoff
description: How every scheduled run starts and ends — read the slug state file, one job only, rewrite the state file in the fixed shape, append one log line, and the session-rollover step.
---

## 🔁 AUTOMATION HANDOFF — every scheduled run (locked 2026-09-28)

Scheduled automations start a FRESH session each time and never remember yesterday. Keep every
run small: use a handoff file instead of a long conversation.

1. **Read first** — `/Users/nakfaai/Developer/projects/ideas/state/<slug>.md` if it exists.
   Slugs: `kalkidan`, `money-focus`, `daily-brief`, `deepseek-midday`, `deepseek-evening`,
   `flight-prep`, `booking-check`, `salsacubanadallas`.
2. **One job only** — do exactly what the task prompt says. Nothing extra.
3. **Write last** — rewrite that same state file, max 20 lines, exactly this shape:
   ```
   # <Task name>
   - Last run: YYYY-MM-DD HH:MM (America/Chicago)
   - Result: <one line>
   - In flight: <half-done work, or "nothing">
   - Next: <the single next action>
   - Blocked: <what Ammaniel must do, or "nothing">
   ```
4. **Append one line** to `/Users/nakfaai/Developer/projects/ideas/state/LOG.md`:
   `YYYY-MM-DD HH:MM | <task> | <result>` — keep only the newest 200 lines.
5. The state file is ALWAYS an allowed write, even when the task prompt says "read-only" or
   "the only file you may write is X". No other permission changes.
6. If the run truly cannot write, skip silently — never fail the task over the handoff.
7. Never paste transcripts, logs, or long text into the state file. It is a handoff, not a diary.

### Session rollover (interactive sessions)

When a conversation passes ~40 turns, changes topic, or the context feels heavy: stop, rewrite
`/Users/nakfaai/.dsh/SESSION.md` in the fixed shape below, and tell Ammaniel to continue in a NEW
session from it. One file for every Bot — the newest session writes it, and nothing else in it.

```
# SESSION — current
- Updated: YYYY-MM-DD HH:MM (America/Chicago) · Bot: <name>
- Status: <one line — where the work stands>
- Done: <finished items, newest first, one line each>
- In flight: <half-done work and who holds it>
- Next: <the single next action>
- Blocked: <what Ammaniel must do, or "nothing">
```

Never push one session until it costs more than it produces.
