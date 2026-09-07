# DSH Web Profile — Plugin Inventory

> Live record of the plugins installed in the DeepSeek Harness **web** profile.
> Profile dir: `~/.dsh/profiles/web` · Managed via `dsh plugin --profile web add <package>`.
> After any change, restart the UI: `~/Developer/dev-tools/dsh-restart.sh`.

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
| `dsh-better-sidebar` | 0.18.0 | Improved sidebar | `dsh plugin --profile web add dsh-better-sidebar` |
| `@michengai/dsh-automation` | ^0.1.31 | Automation support | `dsh plugin --profile web add @michengai/dsh-automation` |
| `dsh-background-nakfaai` | github | Background skin (per-panel background image) | `dsh plugin --profile web add github:themoneyofenoch/dsh-background-nakfaai` |
| `dsh-oil-sticky-prompt` | github | Sticky prompt bar | `dsh plugin --profile web add github:oil-oil/dsh-oil-sticky-prompt` |
| `dsh-plugin-doctor` | github | Server diagnostics (patch plugin, not a visible panel) | `dsh plugin --profile web add github:lin-cheng-lab/dsh-plugin-doctor` |
| `@tt-a1i/archify-dsh` | 0.1.0 | **archify skill** — architecture/sequence/flow diagrams | `dsh plugin --profile web add @tt-a1i/archify-dsh` |
| `@yuxianglin/dsh-bridge-browser` | link | Powers the browser bridge (extension connects separately) | `dsh plugin --profile web add @yuxianglin/dsh-bridge-browser` |

## Notes
- Plugins marked *patch plugin* load at the harness/cordis level and have no visible panel — that's by design, not a failure.
- Some plugins install a native build script (e.g. `cloudflared` for remote-web-ui). If pnpm blocks one, add it to `allowBuilds` in `~/.dsh/profiles/web/pnpm-workspace.yaml`, then re-run the add command.
- Third-party plugins run with your own permissions — only install ones you trust.

Last updated: 2026-09-07
