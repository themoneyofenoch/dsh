# SESSION — current
- Updated: 2026-10-20 19:32 (America/Chicago) · Bot: BOSS 2
- Status: rules and testing are in place — the knowledge split is done, both testers are briefed, one access step waits on Ammaniel.
- Done:
  - Knowledge split: AGENTS.md 40,378 → 28,475 chars (29,021 bytes); six blocks moved into skills `xcode-cloud`, `browser-sandbox`, `purpose-pass`, `automation-handoff`, `testflight-public-link` plus `/Users/nakfaai/.dsh/FOCUS.md`
  - Testers room notice written; Tester and Tester 2 instructions rewritten to read it every round
  - Skills `e2e-test` (Playwright, web) and `device-test` (iPhone + Android) under `/Users/nakfaai/.dsh/skills/`
  - Kalkidan white screen fixed at `/Users/nakfaai/Developer/projects/kalkidan/vite.config.ts:49`, packed build renders, 370 tests green, phone verified
  - Netbrew web suite 3 tests green at `/Users/nakfaai/.dsh/bot/workspace/e2e/netbrew/`
  - Settings pushed: commit `bbc54fe`, then `settings-push` extended to back up `FOCUS.md` and `SESSION.md`
- In flight: Tester 2 has no full access, so its commands stall on an approval the Testers room cannot show; Tester holds the Kalkidan phone results.
- Next: Ammaniel sets Tester 2 to Full access, then Tester 2 gets its first app.
- Blocked: Tester 2 access is set only in the app, by Ammaniel.

# SESSION — netbrew (Bot: netbrew)
- Updated: 2026-10-20 19:33 (America/Chicago)
- Status: groups (event rooms) run end to end and are on the iPhone; the two-phone proof waits on the second phone.
- Done:
  - Groups follow his spec: list at the place, each group's own page, code or organizer QR to join, ORGANIZER/member skip the page, OFFICIAL at 5 inside.
  - Closing a group page no longer leaves the group (it ended every new group at once) — proven on the simulator; iPhone seq 6776, Android https://netbrew.app/netbrew-android-v1.apk 107,196,673 bytes.
  - 8-hour rule live: a group ends at its end time and clears itself away 8 hours later; the scheduled sweep was proven on real rooms.
  - "Join with a code" removed from the Groups sheet; create failures show in plain words under Create.
- In flight: none.
- Next: two-phone test — phone A opens a group, phone B inside joins with the code, phone B outside is refused.
- Blocked: second iPhone 00008130-00164834087A8D3A reads unavailable; item 1 sign-in flows need Ammaniel's thumb; Play testers wait on the Play app-signing SHA-1.
