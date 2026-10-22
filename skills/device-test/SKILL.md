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
