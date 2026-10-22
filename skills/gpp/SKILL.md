---
name: gpp
description: Ammaniel's /gpp shortcut — sets one goal in this chat. Plain /gpp = publish the app he is working on (App Store + Google Play). /gpp followed by his own words = that exact goal, carrying his words through. Triggered only by him typing /gpp.
disable-model-invocation: true
---

# gpp — set the goal, in his words

Ammaniel typed `/gpp`. Set the goal now, in this chat, in this same turn. Answer with one short confirmation.

## Two shapes

**Plain `/gpp`** — publish goal for the app this chat works on.

1. Name the app: the chat's working folder, a `ship.json` there (`appName`, `slug`, `bundleId`), or the open project's `package.json` name. Two matches → pick the app this session's files touch. Ask nothing.
2. `create_goal` objective, one line:

   `Publish <app> to the App Store and Play: build, upload, TestFlight or Play track, store listings and screenshots; then stop at Submit and at Play production review.`

**`/gpp <his words>`** — his wording is the goal.

1. Take every word he typed after `/gpp` and use it as the objective, as close to verbatim as a clear sentence allows. Keep his meaning, his limits and his order of steps. Fill in the app name only where he left it as "this".
2. Never swap his goal for the publish default. If his words ask for one step only (for example a TestFlight build), the objective is that one step.
3. `create_goal` with that objective. Say back the objective in one line so he can see what was set.

If a goal already exists and is unfinished, leave it alone and say which goal is already running.

## Rules that stay

- App Review **Submit** and Play production review stay his click; `--dry` on every Play write.
- Version and build numbers move only after he names the exact number.
- Apple plus Play work starts with `~/.dsh/bin/storeship state`.
- Apple builds: Xcode Cloud. iPhone install: direct to his phone.
- Reply shape: max 6 lines, a status dot per line, the last line is his one next step.
