---
name: record-browser-gif
description: Record a short, truthful demo of a web app flow as one optimised GIF — real server from the repo, headless Chromium capture, the bundled deterministic encoder, then delivery to /Users/nakfaai/output. Use when asked to make, record, or generate a GIF of an app screen or flow (a demo for a README, an X post, a bug report, or "show me it working").
---

# Record a browser GIF

One short GIF that shows an app really working: a real server from the repo, a repeatable script that clicks the flow, then the bundled encoder turns it into a small GIF.

Adapted from the DeepSeek Harness skill `.agents/skills/record-browser-gif` (original kept beside this file as `SKILL.original.md`). The encoder script is copied unchanged from that repo; this file is adapted to Ammaniel's Mac and his rules.

## Rules that never bend

- **Never drive his Safari.** He uses Safari only and it holds his real logins. Recording uses headless Playwright Chromium, which is allowed for headless checks. Do not open his browser, do not reuse his profile, do not log in as him.
- **Real server, real flow.** Boot the app from its own repo and click it like a person. No fixture queries, no mocked transport, no injected events, unless he asked for a fixture.
- **One GIF = one run.** Every frame comes from the same server, the same state and the same script. If a capture breaks, throw its frames away and rerun from scratch; never splice two runs.
- **No secrets, no personal data in frame.** Check the recording interval for keys, emails, phone numbers, other people's photos. His own profile screens are fine only if he asked for them.
- **Nothing installed without asking.** `python3`, `ffmpeg` and `ffprobe` are already on this Mac. `gifski` is not, and it is not needed.
- **Deliver outside `.dsh`.** Copy the finished GIF to `/Users/nakfaai/output/<app>-gif/`, `ls -l` it, and put the full absolute path on its own line.

## Where things go

- Working files: a gitignored folder inside the app repo, `.playwright-mcp/<date>-<flow>/` — script, raw video, QA frames.
- Delivered GIF: `/Users/nakfaai/output/<app>-gif/<flow>.gif`.
- Encoder and its test: `${GIF_SKILL_DIR}/scripts/` (`encode_gif.py`, `test_encode_gif.py`).

## Stage the app

1. Confirm the repo is clean enough to name the commit: `git rev-parse HEAD`, and record it next to the GIF so the proof is exact.
2. Start the app the way its own docs say (for the Expo/Next/Vite apps here: `CI=1 npx expo start --port <port>`, `pnpm dev`, or `npm run dev`). Note the origin you recorded.
3. For a shipped-bundle check, serve the built output (`dist/`, `build/`, or `public/`) with a static server instead of the dev server — a dev server hides chunk-order bugs that only the packed build shows.
4. Stop the old server by PID or exact command-line match, never a broad `pkill -f` (it can kill the shell that started it).

## Record the flow

Pick a story with three to six states and keep one viewport for the whole GIF.

```js
import { createRequire } from 'node:module'
import { join } from 'node:path'
const require = createRequire(join(repo, 'package.json'))
const { chromium } = require('playwright')
const size = { width: 1440, height: 900 }
const browser = await chromium.launch()
const context = await browser.newContext({
  viewport: size,
  recordVideo: { dir: join(runDir, 'videos'), size },
})
try {
  const page = await context.newPage()
  const video = page.video()
  // navigate and exercise the real app here, waiting on visible text, not timers
  await context.close()                       // flushes the video
  await video.saveAs(join(runDir, 'demo.webm'))
} finally {
  await browser.close()
}
```

- Set `viewport` and `recordVideo.size` equal, or Playwright scales the video down and the UI text turns unreadable.
- Wait for real text or a locator before each action; a fixed delay may only hold a settled state, never establish readiness.
- Keep animations and scrolling when they are the thing being shown.

## Encode the GIF

Export the skill folder first, on its own line, so the path expands:

```sh
export GIF_SKILL_DIR=/Users/nakfaai/.dsh/skills/record-browser-gif
python3 "$GIF_SKILL_DIR/scripts/encode_gif.py" \
  /absolute/path/demo.webm /absolute/path/demo.gif \
  --fps 10 --max-width 1200 --colors 128
```

From ordered screenshots instead of video:

```sh
python3 "$GIF_SKILL_DIR/scripts/encode_gif.py" \
  /absolute/path/frames /absolute/path/demo.gif \
  --durations 1.5,1.5,1.5,3.5 --fps 10 --max-width 1200 --colors 128
```

- Video flags: `--start`, `--end`, `--speed`, `--final-hold`, `--max-bytes` (default 5 MB), `--force`.
- Directory flags: `--pattern` (default `*.png`), `--durations` (one value for all frames, or one per frame).
- Shrink an oversized GIF in this order: `--max-width`, then `--colors`, then `--fps`.
- Keep the default 5 MB target unless a service needs smaller.

## Verify, then hand it over

1. Read the encoder's JSON summary: path, frame count, dimensions, duration, bytes.
2. Look at the encoded GIF itself, not only the source frames — decode a few frames with `ffmpeg` if the viewer shows only the first one.
3. Confirm no secrets or personal data appear anywhere in the interval.
4. Copy to `/Users/nakfaai/output/<app>-gif/`, `ls -l` the copy, `sidebar_open` it, and give the full absolute path on its own line plus the commit it was recorded from.

## Publish only when asked

Attaching a GIF to a pull request is a separate step, done only when the task asks for it. Never commit the GIF to a feature branch; use `gh pr edit <pr> --body-file <body.md> --attach <path/to/demo.gif>` (needs gh v2.99.0+, a github.com repo, write access, GIF ≤ 10 MB), or an orphan `<series>-assets` branch when attach cannot apply. Never force-push or rewrite an assets branch.

## Encoder maintenance

```sh
python3 -m unittest discover -s "$GIF_SKILL_DIR/scripts" -p 'test_*.py' -v
```

Run it after any edit to `encode_gif.py`; the tests need the same media prerequisites.
