---
name: e2e-test
description: "Playwright E2E testing for any web app — server lifecycle, reports, screenshots"
trigger: /e2e-test
---

# /e2e-test

Run Playwright-based end-to-end tests against any web app. Handles the full lifecycle: detect or accept dev server command, start server, run tests, generate report, stop server. Works with any framework — Next.js, Vite, plain HTML, or a deployed URL.

## Usage

```
/e2e-test                                          # auto-detect dev server, run tests locally
/e2e-test --url https://staging.example.com        # test a live/deployed site (no local server)
/e2e-test --headed                                 # show browser while running tests
/e2e-test --browser firefox                        # use Firefox instead of Chrome
/e2e-test --browser safari                         # use WebKit (Safari)
/e2e-test --port 5173                              # use a specific dev server port
/e2e-test --url https://staging.example.com --headed --browser firefox   # any combination
```

## Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--url <url>` | (none) | Skip local dev server. Test against a deployed/live URL directly. |
| `--headed` | `false` | Run browser in headed mode (visible). Useful for debugging. |
| `--browser <name>` | `chromium` | Browser engine: `chromium`, `firefox`, `webkit`. |
| `--port <port>` | `auto` | Dev server port. Auto-detected from config if omitted. |

## What this skill is for

E2E tests catch regressions that unit tests miss — broken clicks, missing API responses, layout shifts. This skill gives you a reusable Playwright workflow that works across projects without boilerplate. It handles the boring parts (server lifecycle, port management, report generation) so you can focus on writing tests.

Use it for:

- Adding E2E tests to an existing project
- Validating a production deploy before announcing it
- Debugging a flaky UI flow by watching it run headed
- Cross-browser smoke tests before release

## What You Must Do When Invoked

Run these steps in order. Do not skip steps.

### Step 1 - Check for --url mode

If the user passed `--url <url>`, skip local dev server setup entirely. The URL is the target. Jump to Step 4.

If no `--url` is given, proceed to Step 2.

### Step 2 - Auto-detect the dev server

Determine how to start the local dev server. Do not ask the user — detect it.

Check these in order:

```bash
# 1. Check for Vite config
if ls vite.config.* >/dev/null 2>&1; then
    echo "vite"
    exit 0
fi

# 2. Check for Next.js config
if ls next.config.* >/dev/null 2>&1; then
    echo "next"
    exit 0
fi

# 3. Check package.json scripts (prefer dev, then start, then serve)
if [ -f package.json ]; then
    SCRIPTS=$(python3 -c "
import json
try:
    scripts = json.load(open('package.json')).get('scripts', {})
    for key in ['dev', 'start', 'serve']:
        if key in scripts:
            print(key)
            break
except: pass
" 2>/dev/null)
    if [ -n "$SCRIPTS" ]; then
        echo "npm:$SCRIPTS"
        exit 0
    fi
fi

# 4. Check for a simple HTML file as fallback
if ls *.html >/dev/null 2>&1; then
    echo "static"
    exit 0
fi

# Fallback: suggest --url
echo "unknown"
```

Based on the result:

| Detected | Dev command |
|----------|-------------|
| `vite` | `npx vite --port {PORT}` |
| `next` | `npx next dev --port {PORT}` |
| `npm:dev` | `npm run dev` (append `-- --port {PORT}` if the script supports it) |
| `npm:start` | `npm start` (pass `--port={PORT}` as env if needed) |
| `npm:serve` | `npm run serve` |
| `static` | `python3 -m http.server {PORT}` |
| `unknown` | Stop and tell the user: "Could not auto-detect dev server. Provide --url <url> or specify a start command." |

If the user passed `--port <port>`, use that value. Otherwise pick a free port: start at 5173 and increment until the port is free. Check availability:

```bash
lsof -i :{PORT} >/dev/null 2>&1 && echo "in use" || echo "free"
```

### Step 3 - Start the dev server

Start the dev server in the background and capture its PID. Keep it running for the duration of the tests.

```bash
# Determine PORT (user-provided or auto-detected)
PORT=${E2E_PORT:-5173}

# Start the dev server
npm run dev -- --port $PORT &
SERVER_PID=$!

# Wait for the server to be ready (poll up to 60 seconds)
echo "Waiting for server on port $PORT..."
for i in $(seq 1 60); do
    if curl -s -o /dev/null -w "%{http_code}" http://localhost:$PORT 2>/dev/null | grep -q "200\|301\|302"; then
        echo "Server ready on http://localhost:$PORT"
        break
    fi
    sleep 1
done

# If server did not start, abort
if ! kill -0 $SERVER_PID 2>/dev/null; then
    echo "ERROR: Dev server failed to start."
    exit 1
fi

URL="http://localhost:$PORT"
```

Handle port conflicts gracefully:
- If the port is already in use by another process, try the next port (PORT+1, PORT+2, up to PORT+10).
- If all attempts fail, print "All ports {PORT}-{PORT+10} in use. Use --port <port> to specify a free port." and exit.

### Step 4 - Ensure Playwright is available

Check `npx playwright --version` works. If Playwright is not installed:

```bash
npm init -y >/dev/null 2>&1
npm install -D @playwright/test 2>&1 | tail -3
npx playwright install chromium 2>&1 | tail -3
```

If the user specified `--browser firefox` or `--browser webkit`, install that browser too:

```bash
npx playwright install $BROWSER 2>&1 | tail -3
```

Map the `--browser` flag to Playwright's project names:
- `chromium` (default) → tests use `chromium`
- `firefox` → tests use `firefox`
- `safari` or `webkit` → tests use `webkit`

### Step 5 - Locate or scaffold test files

Check for existing Playwright tests:

```bash
# Common patterns for Playwright test files
FOUND_TESTS=$(find . -type f \( -name "*.spec.ts" -o -name "*.spec.js" -o -name "*.test.ts" -o -name "*.test.js" \) -path "*/e2e/*" 2>/dev/null | head -20)

if [ -z "$FOUND_TESTS" ]; then
    # Broader search
    FOUND_TESTS=$(find . -type f \( -name "*.spec.ts" -o -name "*.spec.js" -o -name "*.test.ts" -o -name "*.test.js" \) -not -path "*/node_modules/*" 2>/dev/null | head -20)
fi
```

**If tests exist:** use them. Print the list of discovered test files.

**If no tests exist:** ask the user what flows to test. Offer examples based on the project type:
- "I found no existing E2E tests. What flows should I test? (e.g., homepage loads, user can sign up, search works, form submission)"
- Wait for the user's answer before proceeding.
- Create the test files in a `e2e/` directory using the template below.

The agent writes the test files. Use this template structure for each test:

```typescript
import { test, expect } from '@playwright/test';

test.describe('FLOW_DESCRIPTION', () => {
  test('TEST_NAME', async ({ page }) => {
    // Go to the target URL (set via the BASE_URL env var)
    await page.goto(process.env.BASE_URL || 'http://localhost:5173');

    // ... test steps ...

    // Use expect for assertions
    await expect(page.locator('SELECTOR')).toBeVisible();
  });
});
```

Set `BASE_URL` to the URL from Step 1 (--url mode) or Step 3 (local mode).

### Step 6 - Run the tests

Run Playwright with the appropriate config. Capture the full output.

```bash
# Map browser flag to Playwright project name
PROJECT_NAME="chromium"
if [ "$E2E_BROWSER" = "firefox" ]; then PROJECT_NAME="firefox"; fi
if [ "$E2E_BROWSER" = "webkit" ] || [ "$E2E_BROWSER" = "safari" ]; then PROJECT_NAME="webkit"; fi

HEADLESS_FLAG=""
if [ "$E2E_HEADED" != "true" ]; then HEADLESS_FLAG="HEADED=false"; fi

eval "$HEADLESS_FLAG" BASE_URL="$E2E_URL" npx playwright test \
  --project="$PROJECT_NAME" \
  --output="e2e-results" \
  2>&1 | tee /tmp/e2e-output.txt
EXIT_CODE=$?
```

If the user passed `--headed`, pass `--headed` to the Playwright CLI and still use `--project` to select the target browser:

```bash
HEADED=true BASE_URL="$E2E_URL" npx playwright test --headed --project="$PROJECT_NAME" --output="e2e-results" 2>&1 | tee /tmp/e2e-output.txt
```

**Playwright config:** If no `playwright.config.ts` or `playwright.config.js` exists in the project root, create a minimal one:

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: '.',
  testMatch: ['**/e2e/**/*.spec.*', '**/*.e2e.spec.*'],
  timeout: 30000,
  retries: 0,
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
  use: {
    headless: !process.env.HEADED,
    viewport: { width: 1280, height: 720 },
    screenshot: 'only-on-failure',
    baseURL: process.env.BASE_URL || 'http://localhost:5173',
  },
});
```

### Step 7 - Generate the report

Parse the Playwright output from `/tmp/e2e-output.txt` and produce a structured report.

```bash
# Extract test counts from output
TOTAL=$(grep -cP '(✓|×|ok|FAIL|passed|failed)' /tmp/e2e-output.txt 2>/dev/null || echo "?")
PASSED=$(grep -c 'passed' /tmp/e2e-output.txt 2>/dev/null || echo "?")
FAILED=$(grep -c 'failed' /tmp/e2e-output.txt 2>/dev/null || echo "?")

# Find screenshot paths for failed tests
FAILED_SCREENSHOTS=$(find e2e-results -name "*-failed-*.png" 2>/dev/null | head -10)
```

Print the report in this format:

```
╔══════════════════════════════════════╗
║  E2E Test Report                     ║
╚══════════════════════════════════════╝

Target:     http://localhost:5173  (or the --url)
Browser:    chromium
Mode:       headless (or headed)
Duration:   12s (or actual duration)

Results:
  Total:    12
  Passed:   10
  Failed:   2

Failed tests:
  ✗ User can complete checkout     → e2e-results/checkout-failed-1.png
  ✗ Search returns results         → e2e-results/search-failed-1.png
```

If all tests passed:
```
All 12 tests passed.   🎉
```

If any tests failed, list each failure with the screenshot path. Offer to debug:
"X test(s) failed. I saved screenshots for each failure. Want me to debug the first failure?"

### Step 8 - Stop the dev server (local mode only)

If we started the dev server in Step 3, kill it:

```bash
kill $SERVER_PID 2>/dev/null
wait $SERVER_PID 2>/dev/null
echo "Dev server stopped."
```

Use a `trap` to clean up the server even if the tests crash or the user interrupts:

```bash
cleanup() {
    if [ -n "$SERVER_PID" ]; then
        kill $SERVER_PID 2>/dev/null
        wait $SERVER_PID 2>/dev/null
        echo "Dev server stopped (cleanup)."
    fi
}
trap cleanup EXIT INT TERM
```

Set the trap right after starting the server in Step 3.

### Step 9 - Report exit code

If any tests failed, exit with code 1 so CI pipelines and callers know the result. If all passed, exit 0.

### What the user does not need to do

- Install Playwright manually (Step 4 handles it)
- Write a Playwright config from scratch (Step 6 scaffolds one)
- Remember to kill the dev server (Step 8 handles cleanup)
- Hunt for test reports (Step 7 prints the summary with screenshot paths)

---

## Reference: /playwright skill

For advanced browser automation tasks beyond E2E testing (scraping, complex interactions, visual regression), load the `/playwright` skill which provides more granular Playwright MCP integration.
