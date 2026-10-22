# DSH Web Profile — Plugin Inventory

> Live record of the plugins installed in the DeepSeek Harness **web** profile.
> Profile dir: `~/.dsh/profiles/web` · Managed via `dsh plugin --profile web add <package>`.
> After any change, restart the UI: `launchctl kickstart -k gui/$(id -u)/com.deepseekai.dsh`.
> ⚠️ `~/Developer/dev-tools/dsh-restart.sh` is STALE for this setup — it races the launchd KeepAlive (see Notes).

## Core (always present)
| Plugin | Purpose |
|---|---|
| `@deepseek-ai/dsh-base` | Core Harness base layers |
| `@deepseek-ai/dsh-web-app` | Web app shell |

## Marketplace / third-party (installed)
| Plugin | Version | What it does | Install command |
|---|---|---|---|
| `dshmarket` | ^1.44.0 | **Plugin Market** — browse/search/one-click install plugins & themes (Settings → Plugin Market) | `dsh plugin --profile web add dshmarket` |
| `dsh-context` | ^0.44.0 | **Context insight panel** — context dashboard, `/context` command, context browser, evolution/compaction stats | `dsh plugin --profile web add dsh-context` |
| `@linxin666/dsh-remote-web-ui` | ^0.3.16 | **Remote & Mobile** — control the workspace from a phone/PC via QR + token channel, SSE sync | `dsh plugin --profile web add @linxin666/dsh-remote-web-ui` |
| `dsh-better-sidebar` | **0.18.1** (pinned) | Improved sidebar. ⚠️ **PINNED — do not install 0.19.x** while core is on the 0.1.2 line: 0.19.0 needs DSH `0.1.5-rc.1+` and `@deepseek-ai/dsh-client-ui-sidebar-right` (absent here). Installing it silently removes the sidebar (2026-09-10). Review after a core upgrade to ≥0.1.5-rc.1. | `dsh plugin --profile web add dsh-better-sidebar@0.18.1` |
| `@michengai/dsh-automation` | ^0.1.31 | Automation support | `dsh plugin --profile web add @michengai/dsh-automation` |
| `dsh-background-nakfaai` | github | Background skin (per-panel background image) | `dsh plugin --profile web add github:themoneyofenoch/dsh-background-nakfaai` |
| `dsh-oil-sticky-prompt` | github | Sticky prompt bar | `dsh plugin --profile web add github:oil-oil/dsh-oil-sticky-prompt` |
| `dsh-plugin-doctor` | github | Server diagnostics (patch plugin, not a visible panel) | `dsh plugin --profile web add github:lin-cheng-lab/dsh-plugin-doctor` |
| `@tt-a1i/archify-dsh` | 0.1.0 | **archify skill** — architecture/sequence/flow diagrams | `dsh plugin --profile web add @tt-a1i/archify-dsh` |
| `@yuxianglin/dsh-bridge-browser` | link | ⏸️ **DISABLED from bundles (2026-09-08)** — built for core ≤0.1.1 (`apiProxy` service, removed in core 0.1.2 which renamed it to `sessionController`/`workspaceController`). Pending source port in `~/.dsh/dsh-browser`. Re-add to bundles in `~/.dsh/profiles/web/package.json` after port. | `dsh plugin --profile web add @yuxianglin/dsh-bridge-browser` |

## DeepSeek V4.1 Flash Beta route (expires ~Sep 10, 2026)

Model ID: `deepseek-v4.1-flash-expires-on-0910` ("DeepSeek-V4.1-Flash-Beta"). NOT in the pristine
`@deepseek-ai/dsh-llm-deepseek@0.1.2-rc.1` catalog. Two layers keep it available:

1. **Durable (settings.yaml, added 2026-09-08):** `llm-deepseek.models` section in `~/.dsh/settings.yaml`
   lists ALL four models (v4-flash, v4-pro, v4-flash-vision-exp, v4.1 beta). This overrides the adapter
   catalog at runtime, survives core reinstalls/updates, and is editable from the Models page
   ("Customized model catalog"). Validated by a clean boot after edit.
2. **Fallback (node_modules patch, 18:42):** the v4.1 entry was also hand-patched into
   `/opt/homebrew/lib/node_modules/@deepseek-ai/dsh/node_modules/@deepseek-ai/dsh-llm-deepseek/lib/index.js`
   (DEFAULT_MODELS). This gets WIPED by any core reinstall (`dsh-update.sh`, `npm install -g`) — do not
   rely on it. Verified identical-except-patch vs the npm 0.1.2-rc.1 tarball.

Verified working 2026-09-08: API key valid, `/models` + `/files` + streaming chat with dsh's exact wire
format (`thinking: {type: enabled}` + `reasoning_effort` + `max_tokens: 256000`) all succeed. The model
is NOT listed by `GET /models` (unlisted beta) but chat completions accept it.

**⚠️ Bridge-browser (updated 2026-09-09 00:25):** re-added to bundles twice by config restages
(2026-09-08 19:03 commit + 2026-09-09 00:03 plugin adds) — each time it kill-booted dsh with
`1 entry did not activate: pending (waiting for service: apiProxy)` (fail-hard fires AFTER the port
binds, so restart scripts print "up" then the process dies). As of 2026-09-09 it is **fully removed**:
gone from bundles AND from `dependencies` (`pnpm remove` with `--config.minimum-release-age=0`).
Do NOT re-add (including via Plugin Market install) until the source in `~/.dsh/dsh-browser` is ported
from `apiProxy` to `sessionController`/`workspaceController`. The `link:` source stays untouched.

**pnpm note:** profile edits that re-resolve the lockfile hit the `minimumReleaseAge` policy when any
dep was published <24h ago — relax it for that one command:
`cd ~/.dsh/profiles/web && pnpm <cmd> --config.minimum-release-age=0` (or use `dsh-update.sh`).

## Notes
- Plugins marked *patch plugin* load at the harness/cordis level and have no visible panel — that's by design, not a failure.
- Some plugins install a native build script (e.g. `cloudflared` for remote-web-ui). If pnpm blocks one, add it to `allowBuilds` in `~/.dsh/profiles/web/pnpm-workspace.yaml`, then re-run the add command.
- Third-party plugins run with your own permissions — only install ones you trust.
- **Core/plugin version rule (learned 2026-09-08):** when the dsh core version changes, ALL profile plugins must match that core generation or boot dies (fail-hard loader). Update order: core first → profile plugins second → boot. One incompatible plugin = whole app won't start.
- **⚠️ Silent-failure variant of that rule (learned 2026-09-10):** a mismatched plugin does NOT always throw a visible error — it can make a **whole feature quietly vanish** while the app still boots. Real case: `dsh-better-sidebar` 0.19.0 on core 0.1.2-rc.1 removed the entire sidebar with no on-screen error; only `/tmp/dsh-web.log` showed `does not provide an export named 'SessionLogOffset'`. **When a feature disappears, read the server log before assuming it was never there.**
  - Checking a plugin's real compatibility: read its `peerDependencies` floors, not symbols in its source. A plugin importing `SessionLogOffset` is NOT proof of incompatibility — core 0.1.2-rc.1 does export it.
  - Verifying a fix *before* restarting: (1) `node --input-type=module -e "await import('<plugin>/lib/index.js')"` and (2) `dsh --profile web --dump-config | grep -A3 <plugin-id>` — both must come back clean.
- **Launchd service (learned 2026-09-10):** the web server is NOT started by hand — it is `~/Library/LaunchAgents/com.deepseekai.dsh.plist` (Label `com.deepseekai.dsh`, runs `/opt/homebrew/bin/dsh web`, `RunAtLoad true`, **`KeepAlive true`**).
  - **Restart with `launchctl kickstart -k gui/$(id -u)/com.deepseekai.dsh`.** Do NOT use `kill`/`pkill`/`dsh-restart.sh`: KeepAlive instantly respawns, and `dsh-restart.sh` additionally `nohup`s its own second copy → two servers racing for :3080.
  - **The live log is `~/Library/Logs/dsh-web.log`** (repointed from `/tmp/dsh-web.log` on 2026-09-10 so it survives reboots). `~/.dsh/web.log` is STALE and belongs to an old manual-launch path — do not read it for live status.
  - The auth token is stable across respawns, so a restart does not force a re-login. The UI is the Safari web app `~/Applications/DSH.app`.
- **Safe update flow:** run `~/Developer/dev-tools/dsh-update.sh` (core install → plugin reconcile → restart → health check). It handles the pnpm `minimumReleaseAge` policy, the `/opt/homebrew/bin/dsh` symlink, and 401-token health checks. `--check` to preview.
- Core install (manual): `npm install -g /opt/homebrew` prefix — dsh lives in the standard Homebrew location again since 2026-09-08 (the `~/Developer/bun` workaround copy was removed; `sudo chown -R nakfaai:admin ~/.npm /opt/homebrew/lib/node_modules/@deepseek-ai` has been run, npm works normally).
- npm cache: permanently redirected to `~/.npm-cache` (the old `~/.npm/_cacache` has root-owned files). `sudo npm/pnpm/yarn` is blocked by a ~/.zshrc guard.

Last updated: 2026-09-10
