---
name: xcode-cloud
description: Connect an app to Xcode Cloud and read its builds — shared scheme, ci_scripts placement, the Node/pnpm script pattern, ASC API calls, failure signatures and current status.
---

## ☁️ XCODE CLOUD — how every app gets connected (locked 2026-10-17)

Apple's build machines watch GitHub and build each app there. **One connection per app** — Apple ties a
product + workflow to the app record / bundle ID. The GitHub grant is shared: `themoneyofenoch` is already
authorized (team 0418454e-4e54-4fc7-a387-b3b2460fd5e8), so later apps reuse it with no Safari re-login.

**The four things each app needs, in this order:**

1. **A git repo Xcode Cloud can read** — a pushed `main`, `node_modules` excluded, secrets gitignored.
2. **A shared scheme** that runs `Archive` (`xcodebuild -list` shows it at top level, and
   `xcshareddata/xcschemes/*.xcscheme` is committed). Autocreation in Xcode makes a scheme that only
   builds locally — share it via Product → Scheme → Manage Schemes → Shared.
3. **`ci_scripts` sitting in the SAME folder as the `.xcodeproj`** — `ios/App/ci_scripts/` for a project at
   `ios/App/App.xcodeproj`, **not** the repo root. That is the single most common breakage: with the folder
   elsewhere Xcode Cloud silently skips it, SPM then fails on `node_modules/... does not exist` in ~1 min.
4. **Build scripts must see what they import.** pnpm on Xcode Cloud uses the isolated layout, so a module
   that exists only as a transitive dependency of vite (`esbuild` was the case) is unresolvable from
   `scripts/*.mjs` → make it a direct devDependency and push `package.json` + `pnpm-lock.yaml` together.

**Script rules:** file names are fixed — `ci_post_clone.sh`, `ci_pre_xcodebuild.sh`, `ci_post_xcodebuild.sh`;
executable, shebang first line; Xcode Cloud sets the cwd to `ci_scripts` itself, so use
`$CI_PRIMARY_REPOSITORY_PATH` for the repo root and always `cd` there before pnpm. Fresh clone every build,
nothing cached between builds.

**The Node/pnpm pattern that works (GeezEasy, `ios/App/ci_scripts/ci_post_clone.sh`):**
`set -euo pipefail` → export `PATH`/`NODE_HOME` for Node 22 → `corepack prepare pnpm@11.15.1 --activate`
→ `corepack pnpm --version` → `cd "$CI_PRIMARY_REPOSITORY_PATH"` → `pnpm install --frozen-lockfile --prod=false`
→ `export VITE_STORE_BUILD=1` → `pnpm run sync:ios:store` (vite build → native:web → `npx cap sync ios`).

**Ammaniel's clicks per app (he does these, 6 clicks):** open the project in Xcode → **⌘9 Report navigator**
→ **Cloud** → **Get Started** → select the product → review the summary → **Grant Access** → GitHub
**Authorize** (Safari) → **Complete** → branch **`main`** → **Start Build**. Everything after that is
automatic: every push to that repo's `main` builds it (workflow default: clean, auto-cancel).

**Watching builds from a Bot (ASC API, key `~/Developer/private_keys/AuthKey_DASF43QDRT.p8`, JWT aud
`appstoreconnect-v1`):** `/v1/ciProducts` → `/v1/ciProducts/{id}/workflows` →
`/v1/ciProducts/{id}/buildRuns` → `/v1/ciBuildRuns/{id}/actions` → `/v1/ciBuildActions/{id}/issues` →
`/v1/ciBuildActions/{id}/artifacts` (fileType `LOG_BUNDLE`, `GET /v1/ciArtifacts/{id}` gives `downloadUrl`;
the zip holds `ci_post_clone.log`). Use the relationship endpoints — `/v1/ciBuildRuns` and
`/v1/ciWorkflows?filter[product]=` return **403** for this key. The API reads; it cannot start a build —
a build starts from a push or from Ammaniel's Start Build.

**Failure signatures → fix:**

| Issue text | Means | Fix |
|---|---|---|
| `Could not resolve package dependencies … node_modules/.pnpm/… doesn't exist` in ~1 min | `ci_post_clone.sh` never ran | move `ci_scripts` beside the `.xcodeproj` |
| `Running ci_post_clone.sh script failed (exited with code 1)` | script ran and failed | download the log bundle, read `ci_post_clone.log` |
| `ERR_MODULE_NOT_FOUND: Cannot find package 'X'` | X only transitive | add X as a direct devDependency |

**Rules that stay:** builds only — versions and build numbers move only on Ammaniel's explicit yes for that
exact number; App Review **Submit** is his click; TestFlight distribution is added to the workflow only when
he names the build number to distribute.

**Status 2026-10-17:** GeezEasy connected — product `App` `ae3c02f8-eda0-4cb1-b250-61448e556c8d`, workflow
`Default` `D1792C59-28B4-4409-957C-9C7C26B33B7D` (containerPath `ios/App/App.xcodeproj`, start on `main`),
builds 1 and 2 FAILED (the two causes above), **build 3 SUCCEEDED on `e0a2e06`**. `main` is 1.6.10 / 93; the
local 1.6.11 / 95 bump is still uncommitted. Remaining apps (orominia, kalkidan, onme, gottasee, fideltype,
facelink, wefam, pixwee, tebaateay1/mobile) are **not** connected yet.
