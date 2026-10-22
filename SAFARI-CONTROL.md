# Safari control (~/.dsh/bin/safari)

Drives the user's REAL Safari (his logged-in sessions). AppleScript + `do JavaScript`.
No Chrome, no separate profile, no passwords.

## Commands
    safari tabs                            list w.ind / url / title for every tab
    safari open URL                        focus an existing tab for that URL, else open a new one
    safari text  [--match SUB]             page innerText (<=6000 chars)
    safari map   [--match SUB]             JSON: url, title, text head, every clickable control
    safari click --index N                 click control N from the last map
    safari click --text "Sign in"          click the first control whose text/aria-label matches
    safari click --aria "Toggle on or off" same, matched against aria-label
    safari type  --index N "hello"         set a field value (fires input+change, React-safe)
    safari focus [--match SUB]             focus the tab and wait until it really loads
    safari fill  --file R.json             fill a whole form from a recipe (+ optional submit)
    safari js 'location.href'              run arbitrary JS (or: safari js --file x.js)
    safari shot  /tmp/x.png                screenshot just the Safari window
    safari wait  3

--match SUB targets the tab whose URL contains SUB (case-insensitive).
Omit it and the first tab is used — always prefer --match so the wrong tab is never touched.
--match now works in ANY position (`safari js --match X 'code'` is fine).

## Form filling (added 2026-10-04)

    safari fill --file ~/.dsh/forms/appstoreconnect.json --match appstoreconnect
    safari fill --file R.json --dry-run      # check the fields resolve, type nothing

Recipe format and rules: `~/.dsh/forms/README.md`. Fields are found by `selector`,
`label`, `aria`, `placeholder`, `text` or `name`; values go through the native value
setter and fire `input`+`change` (React/Vue/Angular-safe); the form is submitted only
if the recipe has a `"submit"` key. This replaces screenshot-and-click for forms —
no mouse permission needed.

## Suspended tabs (fixed 2026-10-04)

Safari suspends tabs you have not looked at; `do JavaScript` on them sees an empty
`about:blank` document, which used to make `text`/`map` return nothing. Every read and
act command now detects that, focuses the tab, and waits (up to 15s) for a real
document. Side effect: the tab is brought to the front.

## Requirements (already true on this machine, 2026-09-14)
- Safari Add/Develop: "Allow JavaScript from Apple Events" ON
- Terminal/DSH app: Screen Recording ON (for `shot`), Automation -> Safari ON
- The DSH session must run WITHOUT the file sandbox: Apple Events are denied
  (-10004 privilege violation) under workspace-write. Use danger-full-access.
- LinkedIn/`do JavaScript` runs even when the target tab is not frontmost.

## Known limits
- Full-page screenshots are not possible; `shot` captures the window only (and focuses it).
- `map` indices shift when the page re-renders: re-run map before clicking by index.
- Sites with heavy virtual lists (LinkedIn Campaign Manager) only expose rendered rows.

## Do NOT
- Do not use Safari WebDriver (safaridriver) for this: it starts a separate, logged-out profile.
- Do not click anything in a live ad account without explicit approval.
