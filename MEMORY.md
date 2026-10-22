# Memory

## App Store Connect — delete an old / duplicate app record (2026-08)

**Why:** used to clear a Guideline 4.3(a) (Design: Spam) rejection caused by a near-identical duplicate test record on the same account (GeezEasy's `old to delete old`, Apple ID 6760924194).

**The 3 steps (in order — "Remove App" silently bounces if 1 & 2 are skipped):**
1. **Pricing and Availability → make the app unavailable in ALL territories.** Open the section → **Manage Availability** → choose **None** to clear every country. Even a never-released app starts at "175 Available".
2. **In-App Purchases → remove any associated IAPs from sale.** Check each app via the API first — some old apps carry zero IAPs.
3. **App Information → scroll to the bottom → Additional Information → Remove App → Remove.**

**Gotchas:**
- The **version / build can never be deleted** — API DELETE returns 409 with permanent "last version of an app cannot be deleted", "a version cannot be deleted if any build has been uploaded", and "a new version must be created on another platform" reasons. An *expired* TestFlight build still counts (expire ≠ delete). Stop trying — it's a dead end.
- App removal also wipes the old app's TestFlight builds (API returns 0 builds for that app id afterwards).
- New App Store Connect UI at a narrow window has **no visible sidebar** — the section links (Pricing and Availability, In-App Purchases, App Information) live in a dropdown that opens when you click the **section's heading button** at the top of the page.
- `/apps/{id}/distribution/ios/pricing` redirects; the real pricing breadcrumb is `/apps/{id}/distribution/pricing`.
- The API **cannot** clear availability for an app that never had it set: `GET/PATCH /v1/apps/{id}/appAvailability` → 404 "relationship does not exist". The territories step is UI-only.

## Owner preferences — ABSOLUTE (2026-09-13, Ammaniel)

**SAFARI ONLY.** Ammaniel uses **Safari and nothing else**. (Stated twice: 2026-09-09 "I don't use Chrome, I use Safari", and 2026-09-13 "use also only safari".)
- NEVER launch, drive, or assume Chrome / Edge / any other browser.
- NEVER `open -a "Google Chrome"`.
- His real logins and browsing live in **Safari** — `open <url>` opens Safari, which is the correct way to show him a page.
- The browser-skill / `bsk` stack is Chrome-only, so it does NOT represent his real browser. Ask before using it.

**ALWAYS ASK BEFORE ACTING.** No build / create / edit / deploy / delete without explicit approval. Propose, then wait. Never touch a live site or production unprompted.

**Timezone = America/Chicago (Dallas).** Always schedule automations in America/Chicago, never Asia/Shanghai.

**DeepSeek off-peak = 11 PM – 5 AM (America/Chicago).** Prefer heavy agentic work in that window.

**Communication style = short, numbers first, bullets/tables, no fluff.**

**Two daily tracks** (both every day): (1) Google Ads → corporate headshots at the already-live ammanielhintza.com funnel — goal 20k in 2 months, measured in BOOKINGS; never rebuild that funnel, never put headshots on pixwee.com. (2) The two books, alternating — `~/Developer/projects/Books/Ethiopia's Stolen Name` / `PRE`. Daily 8:30am brief goes to Telegram (@Nakfaaidshbot).

## DSH in Safari — model / reasoning-effort picker silently does nothing (2026-09-22)

**Symptom:** in **Safari**, clicking a model or effort option in the composer's model menu closes the menu and changes nothing. Works in Chrome.

**Cause:** Safari does not focus a `<button>` on click, so the picker's `onBlur` fires with `relatedTarget === null` → `close()` runs on mousedown → the menu unmounts before the click lands. Chrome focuses the button, so the early `return` fires and the click works.

**Fix applied (local patch — LOST on any DSH upgrade, re-apply after):** in `/opt/homebrew/lib/node_modules/@deepseek-ai/dsh/node_modules/@deepseek-ai/dsh-client-ui-model-selection/lib/client.js`, add `if (event.relatedTarget === null) return;` as the first line of `onBlur`. Backup: same path + `.bak-safari`. Then restart dsh.

**Surrounding facts learned the hard way (do not re-derive):**
- The effort ladder only renders when the session's current model id matches a **catalog** entry. Chats holding retired ids (`deepseek-v4-flash-vision-exp`, `deepseek-v4.1-flash-expires-on-0910`) show an EMPTY effort list — this is not a provider problem.
- Server RPC is callable with the `?token=` cookie: `POST /api/session/modelCatalog` body `{"args":{}}`; `POST /api/session/selectModel` body `{"args":{"request":{"sessionId","provider","model","reasoningEffort"}}}`. Method name uses `/`, envelope `{"type":"client-request","rpcId":…,"method":…,"payload":…}`.
- Model selections are **not** session-log events; `lastUsed` advances only on a real `request/header`. Editing `.jsonl.zstd` session logs achieves nothing — don't.
- DSH auto-updates via launchd `ai.deepseek.dsh-update-check`; it moved 0.1.5 → **0.1.7-alpha.1**, after which 0.1.5-built plugins throw: `dsh-better-sidebar` 0.19.1 → `conversation.chat.turnTail requires options.id`; `sidebar.footer.action` → React #130 (owner: `dsh-context` or `@linxin666/dsh-remote-web-ui`).
- Native DeepSeek per api-docs: base_url OpenAI `https://api.deepseek.com`, Anthropic `https://api.deepseek.com/anthropic`; models `deepseek-flash` (DeepSeek-V4.1-Flash) + `deepseek-v4-pro` (V4-Pro-0813); context 1M, max output 384K; thinking on by default with effort `high` (OpenAI `reasoning_effort`, Anthropic `reasoning.effort`). The built-in provider speaks the Anthropic wire; the OpenAI URL needs `llm-pi-ai`.

## Dead / empty grey sidebar panes — after the 0.1.7 harness replacement (2026-09-23)

The harness at `/opt/homebrew/lib/node_modules/@deepseek-ai/dsh` was **replaced 2026-09-22 09:57**, so every bundled package is now `0.1.7-alpha.1` while user plugins were built for 0.1.5. Several plugin panels render as a dead grey pane:

- **`dsh-element-source`** → a "Local preview" pane stuck on *"Open a session first"* (its tab never binds a session). **Fix: remove `dsh-element-source` from `dsh.profile.bundles`** in `~/.dsh/profiles/web/package.json` (keep it in `dependencies`; backup `package.json.bak-elementsource-*`), then restart. ✅ done, pane gone.
- **`dsh-better-sidebar`** → no longer in `bundles`; its per-session layouts linger in `localStorage` under `dsh-sidebar:v1:<sessionId>` (shape `{panelOpen,width,activePane,splits:{kind:"leaf",id,tabs:[],active},bottomSplits,floats}`). A stale `browser` tab (e.g. google.com) makes the panel show *"Nothing here can view this kind of content yet."* Fix: back up those keys, then set `splits.tabs=[]`, `splits.active=null`, `panelOpen=false`, reload.

**How to inspect the live GUI (reliable method):** DSH runs in **Safari** — use `~/.dsh/bin/safari --match 3080 js '<expr>'`; `do JavaScript` works even when the tab is not frontmost. `safari shot` / `screencapture` capture whatever window is frontmost (OpenCode kept stealing focus), so a screenshot can show a different app entirely — trust the DOM, not the crop. Focus a tab with AppleScript (`set current tab of tw to tt`, `set index of tw to 1`, `activate`).


## Books — King Ezana (Ammaniel, 2026-09-23)
- Author's framing for **The King Who Gave Us the Cross**: the Solomonic dynasty did not want Ezana to be bigger than them, and the church today does not want him well known. Handle as **displacement, not censorship** — never assert a suppression; show it by contrast (what is celebrated vs what is not). Tone model: "It is not hidden. It is not censored. It is simply not in the room."
- Book 2 has **chapter 10 — Ethiopia Means Burnt Face** (new): thesis "No Ezana, no Ethiopia"; the name is Greek, means burnt face, was the Greeks' name for Black Africa (Nubia most concretely), and passed to the highlands through Ezana's title after the Kush campaign.
- **CORRECTION (same day):** that dynasty/church framing is Ammaniel's **private motive — never to be written in the book**. No accusation, no hint. Ezana is made big by giving him his due; no one else is diminished.
- **RULE (2026-09-23):** never add without taking — no new chapter, section or plate without removing an equivalent one. The book does not grow.
- **CORE RULE (2026-09-23):** in the Ezana book the author must give **Ezana more credit than anyone else** — every chapter closes on him; others get their exact due and no more.

## HEART OF THE EZANA BOOK (Ammaniel, 2026-09-23)
- Ask any Ethiopian: what country? Ethiopia. What faith? Orthodox Christian. **Both answers came from one king — Ezana gave the name and the religion.** That is the book.
- Private (never in the text): the Solomonic dynasty and the church do not want this known; the author wants it known.
- Implementation: pair the NAME (ch.10) with the CROSS (ch.3–7); open the Introduction on the two questions; make it the back-cover and closing line.

## DSH Desktop — the official app EXISTS and is installable (2026-09-24)

**Key fact:** `deepseek-ai/deepseek-harness` GitHub **Releases carry 0 assets** (checked all 22 releases — no binaries, no CI artifacts except 2 expired npm tarballs). That is NOT evidence DeepSeek ships no desktop build. An official, Apple-notarized macOS DMG does exist and circulate:

```
DeepSeek Harness.app   bundle id com.deepseek.dsh
version 0.1.7-rc.1.20260924.1     (date-stamped = the desktop test-build scheme)
Authority=Developer ID Application: Hangzhou DeepSeek Artificial Intelligence Co., Ltd (NAN929V4UM)
source=Notarized Developer ID      ← spctl accepted
```

Installed at `/Applications/DeepSeek Harness.app`. Verify any candidate DMG with:
`codesign -dv --verbose=2 <file>.dmg` then `spctl -a -vvv -t open --context context:primary-signature <file>.dmg` — a genuine build shows DeepSeek's team **NAN929V4UM** and `source=Notarized Developer ID`.

**Profile/home model (critical):** the desktop app owns `$DSH_HOME/profiles/desktop` (bundles: `dsh-base`, `dsh-web-app`) — a SIBLING of `profiles/web`. Sessions live at home level in `$DSH_HOME/sessions`, so pointing the desktop at `~/.dsh` **shows all existing sessions/workspaces without moving anything**. Web (`:3080`) and Desktop (`:19387`) coexist; scheduled tasks are OFF by default on Desktop. The app self-updates from DeepSeek's feed (offered V0.1.7-rc.2 in-app).

**Building it from source (only needed if you want your own):**
```sh
git clone https://github.com/deepseek-ai/deepseek-harness.git && cd deepseek-harness
pnpm install && pnpm run build:desktop && pnpm run build
pnpm run start:desktop            # dev launcher; needs DSH_HOME (else "DSH_DESKTOP_PRIMARY_RUNTIME_DIR is required for an unpackaged launch")
pnpm run package:desktop          # signed DMG — macOS CANNOT be unsigned
```
Gotchas hit while packaging (all solved, in order):
1. `.env.macos` **must** exist (copy `.env.macos.example`); release settings are read ONLY from that file — ambient `DSH_DESKTOP_*`/`APPLE_*`/`CSC_*` are stripped by design.
2. `--unsigned` is **win-x64 only**; `createElectronBuilderConfig` explicitly throws "unsigned builds require Windows" for macOS.
3. Auto-update config needs `DOWNLOAD_TEST_RELEASE_ID` = exactly **32 lowercase hex**; `DOWNLOAD_TEST_ORIGIN` + `DSH_DESKTOP_MANDATORY_UPDATE_{TEST,PROD}_ORIGIN` must be valid HTTPS origins (template ships them empty).
4. `security import` rejects OpenSSL 3's default p12 → export with **`openssl pkcs12 -export -legacy`**.
5. `DSH_DESKTOP_MACOS_SIGNING_IDENTITY` is the **bare CN** (`Ammaniel Hintza (8MYWX3FXP9)`): the probe prepends `Developer ID Application: `, and the validator rejects a value that already has the prefix.
6. `signMacOSRuntimeCode` (scripts/verify-macos-signature.mjs) passed the bare CN to `codesign --sign`, which is **ambiguous** when another cert shares the suffix (his `iPhone Distribution: Ammaniel Hintza (8MYWX3FXP9)`). Fix: pass the fully-qualified `Developer ID Application: ${signingIdentity}`. (Local patch in the clone.)
7. Developer ID certs CANNOT be created with an Admin-role API key — Apple returns **403 "can only be performed by the Account Holder"**. Portal route works: create CSR (`~/Developer/private_keys/developer-id/devid.csr`), upload at developer.apple.com → Certificates → + → Developer ID Application, download the `.cer`, pair with the key via `openssl pkcs12 -legacy`.

Supporting facts: `.env.macos`/`.env.windows` and the generated `devid*.p12` are secrets — never commit.

## DSH desktop — voice input blocked by DeepSeek's own signature; how to report bugs (2026-09-24)

**Bug (reported as discussion #7762):** the macOS desktop app is signed with the **hardened runtime** but the signature lacks `com.apple.security.device.audio-input` (only `allow-jit`, `allow-unsigned-executable-memory`, `disable-library-validation`). Under hardened runtime, tccd then *disallows the prompt entirely*:
`Prompting policy for hardened runtime; service=kTCCServiceMicrophone requires entitlement com.apple.security.device.audio-input but it is missing ... Policy disallows prompt ... access denied`.
So microphone permission is **ungrantable by the user** — no prompt, `tccutil reset Microphone com.deepseek.dsh` is useless, and System Settings → Microphone → `+` is blocked by the same policy. The client plugin (`@deepseek-ai/dsh-experimental-client-ui-voice-input`) then shows its `permission` string: "Microphone access is disabled. Allow it in browser and system settings."
- Diagnose any TCC/permission denial with: `log show --last 5m --predicate 'subsystem == "com.apple.TCC"' --style compact | grep -i <bundle-id>`
- Check an app's entitlements: `codesign -d --entitlements - <app>` — a *non-sandboxed* app still needs `audio-input` when the hardened runtime flag is set (that nuance cost time here).
- Workaround: use voice input in the **web** UI (Safari) — the browser holds the entitlement. Or re-sign the app locally with a Developer ID cert + an entitlements plist containing `audio-input`.

**Reporting to DeepSeek — Issues are DISABLED on `deepseek-ai/deepseek-harness` (REST returns HTTP 410 "Issues has been disabled in this repository").** CONTRIBUTING.md directs bug reports to **GitHub Discussions**. Discussions can only be created via **GraphQL**, not REST:
```graphql
query { repository(owner:"deepseek-ai", name:"deepseek-harness") { id discussionCategories(first:25){ nodes { id name } } } }
mutation($repo:ID!,$cat:ID!,$title:String!,$body:String!){ createDiscussion(input:{repositoryId:$repo, categoryId:$cat, title:$title, body:$body}){ discussion { url number } } }
```
Repo id `R_kgDOT3T1gw`; categories: Announcements `DIC_kwDOT3T1g84DDSUa`, **General `DIC_kwDOT3T1g84DDSUb`** (no bug category — use General), Q&A `...SUc`, Ideas `...SUd`, Show Your Plugins `...SUe`, Polls `...SUf`. Auth: `GITHUB_TOKEN` from the launchd plist (login `themoneyofenoch`).

**Inspecting a packed Electron app without extra tools:** `app.asar` is readable with a ~15-line Python reader (uint32LE JSON header size at byte 12; data starts at 16+jsonSize; walk `header.files[].files` and seek to `data_start + int(offset)`). Used it to extract `/dsh/node_modules/.../README.md` and to locate `/lib/main.js` permission logic.
