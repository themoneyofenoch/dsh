# DSH knowledge structure — where every thing lives

Locked 2026-10-20, checked against the running app (DSH 0.2.1-rc.2), not guessed.
Every number below was measured on this Mac today.

## 1. The nine layers

| # | Layer | Lives in | Who reads it | When it enters | Size cap |
|---|---|---|---|---|---|
| 1 | **Law** | `/Users/nakfaai/.dsh/AGENTS.md` (39,556 B) | every Bot, every chat | first request of every session, injected by `@deepseek-ai/dsh-agent-instructions` | whole baseline capped at **65,536 B**; one file up to 1 MB |
| 2 | **Shared agents file** | `/Users/nakfaai/.agents/AGENTS.md` (absent today) | every Bot | right after the law, same baseline | shared with the 64 KB cap |
| 3 | **Room notice** | `rooms[*].notice` in `/Users/nakfaai/.dsh/bot/state.json` | the members of that room only | every round in that room, as "Group notice (shared instructions for every member)" (`src/host/groups.js:231`) | no fixed cap |
| 4 | **Bot instructions** | Bot settings (`bots[*].instructions`) | that one Bot | its system prompt, every turn (`src/host/prompt.js:50`) | keep 1–2 KB |
| 5 | **Team memory** | `/Users/nakfaai/.dsh/bot/memory/team/MEMORY.md` (3,235 B) + topics | every Bot | system prompt, frozen when a session starts (`src/host/memory.js:712`) | MEMORY.md **4,000 B**, topic **16,000 B**, 40 topics |
| 6 | **Bot memory** | `/Users/nakfaai/.dsh/bot/memory/bots/<session-id>/MEMORY.md` | that one Bot | same as team memory | 4,000 B, one entry 300 B |
| 7 | **Project law** | `<project>/AGENTS.md` — 28 files today, from the `.git` root down to the working folder | Bots working in that folder | first request there; reloads when a read/write/edit reaches a deeper folder | counts inside the same 64 KB |
| 8 | **Procedures** | `/Users/nakfaai/.dsh/skills/<name>/SKILL.md` | a Bot that loads it, or any Bot a slash command reaches | on demand only | model reads it whole when loaded |
| 9 | **Reference docs** | `~/.dsh/persona.md`, `~/.dsh/DESIGN.md`, `~/.dsh/SAFARI-CONTROL.md`, `~/.dsh/docs/*.md` | whoever the task names | on demand | none |

Two facts that shape everything:

- The law and the project chain are **not watched**. A change shows at the next session, or at the next read/write/edit that reaches a deeper folder, or on resume.
- Both `MEMORY.md` files sit in the **system prompt** and are frozen at session start, so the prompt cache holds. A new entry shows from the next part or checkpoint, and `recall` finds it at once.

## 2. Decision rule — one question at a time

1. Must **every Bot** obey it in **every task**? → `AGENTS.md` (law). Nothing else goes there.
2. Is it a **repeatable procedure** with steps? → a skill in `~/.dsh/skills`, with one pointer line in the law.
3. Is it true only inside **one room**? → that room's **notice**.
4. Is it true only inside **one app or project**? → that project's **AGENTS.md**.
5. Is it **one short fact** several Bots need? → **team memory** (≤300 characters an entry).
6. Is it **one Bot's domain detail**? → that Bot's **own memory**, topic for length.
7. Is it **what happened**? → it stays in chat; `read_own_chat` opens it again.

Never: a rule that lives only in chat · a build recipe in memory · a long document in the law with no pointer.

## 3. What is wrong today (measured)

| Problem | Measure | Effect |
|---|---|---|
| The law carries domain detail | 39,556 B of 65,536 used; about 10,000 tokens injected into every session of 33 Bots | the most expensive file in the system, and 7.5 KB of it matters only to some tasks |
| Three rooms have no notice | GEEZEASY, BOOKS 📚📚📚, Testers | their shared rules live only in chat |
| Team memory nearly full | 3,235 / 4,000 B | new team facts push older ones out |
| One Bot memory over the cap | 4,045 / 4,000 B | the memory review trims it every turn |

## 4. The split (proposed — nothing moved yet)

Move these blocks out of the law, leave one pointer line each:

| Block | Bytes | New home |
|---|---|---|
| ☁️ XCODE CLOUD — how every app gets connected | 4,513 | `~/.dsh/docs/xcode-cloud.md` |
| Browser / Sandbox notes | 3,148 | `~/.dsh/docs/safari-and-browser.md` |
| 🎯 PURPOSE PASS (framing) | 2,434 | `~/Developer/projects/Books/AGENTS.md` |
| 🔁 AUTOMATION HANDOFF + Session rollover | 1,893 | `~/.dsh/docs/automation-handoff.md` |
| 🔒 TestFlight public link standard | 803 | `~/.dsh/docs/testflight-public-link.md` |
| Current Focus (2026-09-09) | 704 | `~/.dsh/docs/current-focus.md` |

Result: **39,556 B → about 26,700 B** — every session of every Bot about one third lighter.
Optional step two: move the long reasoning inside **Rules — Absolute** (13,294 B) into `~/.dsh/docs/rules-why.md`, keeping every rule line, toward 18,000 B.

## 5. Room notices to write

- **Testers** — the two skills (`/e2e-test`, `/device-test`), the order (web first, then phone on the packed build), suites in `/.dsh/bot/workspace/e2e/<app>/`, reports in `/Users/nakfaai/output/<app>-e2e/`.
- **BOOKS 📚📚📚** — book list, `publish/KDP` and `publish/APPLE` folders, never spend an ISBN.
- **GEEZEASY** — app ids, `~/.dsh/bin/storeship`, Submit and production remain his click.

## 6. Skills registry (procedures, loaded on demand)

`e2e-test` · `device-test` · `gpp` · `bs` · `app-store-ship` · `plugin-test` · `plugin-workflow` · `plugin-release` · `plugin-write` · `plugin-upgrade` · `plugin-fleet-sweep` · `plugin-runtime-debug` · `plugin-heavy-dep` · `generic-migration` · `dsh-upgrade-audit` · `dsh-benchmark-case` · `office` · `office-docx` · `office-xlsx` · `office-pptx` · `office-render` · `office-export-pdf` · `office-recalculate`
