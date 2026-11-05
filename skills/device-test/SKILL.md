---
name: device-test
description: "On-device E2E for an app — iPhone and Android: build, install, launch, tap, verify, screenshots, report"
trigger: /device-test
---

# On-device E2E test

The web layer lives in the `e2e-test` skill (Playwright). This skill is the second layer: the app on a real phone.
Run it after the web layer passes, and again on the PACKED build — a suite that runs the dev server can pass while the shipped bundle crashes.

## 1. Read the tool skills first

- iPhone, real or simulator: load `ios-ui-automation` before the first `ios_sim_*` call.
- Android, real or emulator: load `android-ui-automation` before the first `android_*` call.

Never drive the UI from a screenshot's guessed coordinates. Tap by identity (label, resource-id, accessibility id), and confirm with `expect_text` / `expect_gone`.

## 2. Find the device

```bash
xcrun devicectl list devices            # state "connected", not "unavailable"
adb devices                             # state "device", not "unauthorized"
```

- A phone in `unavailable` is unplugged or locked. Say so, do not retry in a loop.
- Note the exact identifier you will use, e.g. `00008120-001A5DC90210C01E`.

### The devices that exist on this Mac (checked 2026-11-02)

- Android emulator, AVD name `kalkidan_test` — Android 14, API 34, serial `emulator-5554`.
  Start it with the `android_boot` tool (`device: "kalkidan_test"`), or
  `$ANDROID_HOME/emulator/emulator -avd kalkidan_test &`. `android_devices` lists whatever is
  live and the AVD names that can be booted; read it first rather than assuming.
- An emulator is a pretend Android phone running on this Mac: the same Android system and the
  same APK, no hardware. Testing there is real testing of the app, not a simulation of it —
  but say plainly that it is an emulator when reporting, never "a phone".
- A real iPhone or Android phone only appears when Ammaniel plugs it in. If none is attached,
  say so and test on the emulator instead of waiting.

## 3. Build the build that ships

- Prefer the project's own pipeline: `~/.dsh/bin/storeship state` first, then `build`.
- iOS by hand (Release, signed, never a version bump):
  ```bash
  xcodebuild -workspace ios/App/App.xcworkspace -scheme App -configuration Release \
    -destination 'generic/platform=iOS' -derivedDataPath ios/App/build-<app> \
    CODE_SIGN_IDENTITY="Apple Development" CODE_SIGN_STYLE=Automatic \
    DEVELOPMENT_TEAM=8MYWX3FXP9
  ```
- Capacitor projects: `npx cap sync ios` (and `android`) after `npm run build`.
- Android: `./gradlew assembleRelease`, APK under `android/app/build/outputs/apk/release/`.

Versions and build numbers move only when Ammaniel names the exact number.

## 4. Install and launch

```bash
xcrun devicectl device install app --device <UDID> path/to/App.app
xcrun devicectl device process launch --device <UDID> com.example.app
adb install -r app-release.apk && adb shell monkey -p com.example.app -c android.intent.category.LAUNCHER 1
```

First launch on iOS can take 3–5 s; wait for real text, not a fixed sleep.

## 5. Walk the screens

Cover, in this order and stop at the first break:

1. Launch — the app paints text, not a white or black screen.
2. First screen the user sees — headline, primary button, one tap.
3. One core flow end to end — tap, type, confirm the result on screen.
4. Sign-in and permission prompts — say which prompt you cannot pass (Face ID, keychain) and hand it to him.
5. Back and tab bar — every tab opens, no crash.

Screenshot every step to `/Users/nakfaai/output/<app>-e2e/shots/` with a numbered name (`p01-launch.png`). Check the file size — a tiny PNG is usually a blank screen.

## 6. Read the failure properly

- Capture the crash/console text before guessing: `ios_sim_logs`, `android_logs buffer:"crash"`, `android_backtrace`.
- A JS bundle error such as `Cannot read properties of undefined (reading 'createContext')` at boot is a bundling fault (chunk order), not a device fault.
- Old passing tests prove nothing about this build: check how they start the app (`playwright.config.ts` `webServer`, `npm run test:e2e`) before trusting them.

## 7. Report

Write `/Users/nakfaai/output/<app>-e2e/report/index.html` (single file, LIGHT theme only — no `prefers-color-scheme: dark`) and `report.md` beside it:

- what was built (version, build number, commit), on which device (model, iOS/Android version);
- each step: pass or fail, with the screen text or the error quoted;
- pictures inline, one per step;
- one "next" line: the single thing to fix or the single click that stays Ammaniel's.

Link the report by full path, never `~`, and keep it outside `.dsh` — the viewer breaks links under `.dsh/bot/workspace`.

## Limits that stay

- Maestro 2.11.0 runs Android phones and simulators only ("Physical iOS devices are not yet supported"); use it for Android flow scripts, run with `JAVA_HOME=/opt/homebrew/opt/openjdk@17`.
- App Review Submit, version bumps, ISBNs and purchases stay Ammaniel's click.
- Playwright and Maestro drive their own browsers/devices — never his Safari tabs.

## Daily self-check — do it without being told

Ammaniel, 2026-11-02: *"Do the full check on your app yourself — do not wait for me to tell you."*

Once a day, on your own app:

1. Open the real app (phone or emulator, and the web) and tap one flow nobody has tapped yet:
   sign-up, sign-in each way, password reset, delete account, filters, purchases,
   notifications, chat, block and report, sign out and back in.
2. Write down in plain words what worked and what broke, with one picture per fault.
3. Check both stores (App Store and Google Play) for what changed and what waits on him.
4. Never guess. If you did not look, say "not checked".
5. Ask him only for: a version number, a versionCode, an upload, a push to main, or a Submit
   press. Everything else: fix it, or hand it to the Bot that codes it, with proof.
6. One question at a time, and nothing else in that message.

Report short: what you checked, what broke, what you fixed, and the one thing he does next.
