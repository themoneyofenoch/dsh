# DSH Agents — Ammaniel's Agent

My own persistent agent file (this is MINE, stored in my own repo).
`~/.opencode/` belongs to OpenCode — I never write there.

## Rules — Absolute

- **ALWAYS read this file `~/.dsh/agents.md` at the start of every session/task.** Check rules, preferences, notes first — never skip.
- **ALWAYS exclude `node_modules` when pushing anything to GitHub.** "always without node" (Ammaniel, 2026-08-24). Never commit node_modules — reproducible from lockfile, hundreds of MB. Applies everywhere, no exceptions.
- **Secrets never go to GitHub.** `.credentials.yaml`, `.env*`, tokens, keys — always gitignored, even on private repos.
- **Private/local state stays local.** Session transcripts, caches, machine-local storages (`sessions/`, `storages/`) — excluded unless Ammaniel says otherwise.

## Preferences (learned)

- (2026-08-24) "always without node" — see Rules above.

## Stack & Environment

- habeshamarketing app: Next.js 16 + Prisma 6 + SQLite, `pnpm dev` on port **3111** (not 3000).
- Prisma client must be `pnpm prisma generate`'d after clone (not committed). DB: `prisma/dev.db`.

## Browser / Sandbox notes

- Controlled browser blocks `localhost` + LAN IPs ("Blocked: local and internal addresses"). `browser_navigate` to app times out on approval prompt; separate from sandbox on/off toggle.
