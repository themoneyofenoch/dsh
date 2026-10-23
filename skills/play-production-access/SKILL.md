---
name: play-production-access
description: Read and finish Google Play's production gate — the closed test with 12 opted-in testers for 14 days, then Ammaniel's Apply click. Use before saying any app is "live on Google Play", when a production track returns 400 FAILED_PRECONDITION / 500 INTERNAL, or when an app is stuck on internal or closed testing.
---

# Google Play production access — the 12-tester gate

Checked against Netbrew on 2026-10-22. Every number below was read from the console or the Play API that day.

## The gate

A Play developer account marked **Personal account** needs, per app, in this order:

1. a **published closed testing release**;
2. **12 testers opted in** to that closed test;
3. those 12 kept opted in **14 days in a row**;
4. then **"Apply for access to production"** in the Play Console — a human click. No Play API endpoint applies for production access.

An account that already has a production app (GeezEasy: production 1.6.10 / versionCode 34) passes the gate app by app: the next app still needs its own closed test.

## How to read the state (the API alone cannot answer)

- Opening an edit and reading the `production` track shows **no releases**. Writing a draft release there returns `400 FAILED_PRECONDITION` (or `500 INTERNAL` on PATCH) and names no reason. Do not read that as a build or metadata problem.
- The reason is on the console page
  `play.google.com/console/u/<account-index>/developers/<devId>/app/<appId>/app-dashboard`
  panel **"Apply for access to production"**: tick list (closed testing release ✓), live line **"N tester(s) currently opted-in"**, the 14-day line, and the **Apply for production** button.
- The app-list row shows the app's state: `Draft`, `Internal testing`, `Closed testing`, `Production`.
- In the Play API, track **`alpha`** is the closed testing track; `beta` is open testing; custom closed tracks appear by their own id.

## Finding the ids

- `devId` is in any console URL: `play.google.com/console/u/1/developers/<devId>/app-list`.
- `appId` is per app. On the app-list page the rows are `div.particle-table-row` and are **not** anchors: run one line of JS in the Safari tool over `div.particle-table-row`, pick the row whose text holds the package name, and read the `View app` anchor's `href` — it carries `/app/<appId>/app-dashboard`.
- `u/0` and `u/1` are different Google logins. The Play account here is **u/1** (`ammaniel.hintza@googlemail.com`); `u/0` is a different login and lands on an empty console.

## Adding testers

- The Play API accepts **Google Groups only** in a tester list (`testers_update {googleGroups: ["…"]}`). Single email addresses are a console-only field.
- A tester counts only after accepting the invite and opting in. Invited is not opted in.
- The 14 days need the same tester set staying opted in; dropping below 12 restarts the clock's qualifying run.

## Current Netbrew numbers (read 2026-10-22, America/Chicago evening)

- Package `com.silki.app`; devId `6647209531247794023` (account **Nakfaai**, Personal); appId `4972141634758098933`.
- Closed testing release: published ✓. Testers opted in: **1 of 12**. Production: **Inactive**.
- Android 1.0.1 / versionCode 5 sits on **internal testing** (released, completed).
- The Apple side (build 7 on App Store Connect) does not wait on this gate.

## What stays human

- **Apply for access to production** — Ammaniel's click.
- Any production release, rollout and Play review decision.
- Creating a brand-new app (the API cannot create apps).

## Rule of reporting

Never call an app "live on Google Play", "ready on Google" or "in production" from an internal-testing or closed-testing track, or from an uploaded bundle. It is live only when a `production` track carries a release with status `completed` and the store page resolves.
