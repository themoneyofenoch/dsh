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
- Updated: 2026-10-26 11:10 (America/Chicago)
- Status: app is on TestFlight build 8 and Play internal versionCode 6; the café screen can no longer print raw server text, and only the phone install is owed.
- Done:
  - No screen prints a raw server line: new `plainError.ts`, the 8 raw paths on the café screen, safety actions and account delete go through it, and a test fails if a raw `.message` comes back. main `c2de35d`, CHANGELOG line written.
  - Suite 81 pass / 0 fail (was 72); type-check clean outside the 4 known `@types/node` test-file errors.
  - Build 8 on TestFlight with beta review APPROVED (link https://testflight.apple.com/join/8RF2eSWc); Android versionCode 6 / versionName 1.0.1 on Play internal, completed — both from his own numbers yes 8 and 6.
  - "Here now" is measured: 150 m check-in enforced, his face sits only in rooms he is really in, six cafés seeded with 8 faces each.
  - Walking out removes presence at once; a 10-minute heartbeat keeps it alive; café counters count only people still there.
- In flight: Release archive `/tmp/nb-plain.xcarchive` is built; install to his iPhone `00008120-001A5DC90210C01E` refused with `CoreDeviceError 4016` (device locked).
- Next: install that archive on the iPhone when it unlocks, open the café screen, then report to App Manager in one block.
- Blocked: iPhone must be unlocked; Tester 2 re-runs create-group on the next build; Play production needs 12 opted-in testers for 14 days then his Apply click; App Review needs a way in for reviewers outside Dallas; Submit stays his click; new build numbers need his yes before the newer UI work ships.
