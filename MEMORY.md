# DSH Memory — Ammaniel's Agent

My own persistent memory (this is MY memory, stored in my own repo).
`~/.opencode/` belongs to OpenCode — I never write there.

## Rules — Absolute

- **ALWAYS exclude `node_modules` when pushing anything to GitHub.** "always
  without node" (Ammaniel, 2026-08-24). Never commit node_modules — they are
  reproducible from the lockfile and weigh hundreds of MB.
  Applies to: dotfiles, config folders, apps — everything, no exceptions.
- **Secrets never go to GitHub.** `.credentials.yaml`, `.env*`, tokens, keys —
  always gitignored, even on private repos.
- **Private/local state stays local.** Session transcripts, caches, machine-local
  storages (`sessions/`, `storages/`) — excluded unless Ammaniel says otherwise.

## Preferences (learned)

- (2026-08-24) "always without node" — see Rules above.
