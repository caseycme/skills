---
name: validate-frontend
description: >-
  Validates frontend UI changes with Playwright CLI against the local app.
  Use when the user asks to validate, verify, smoke-test, or browser-check UI
  changes; after implementing UI features; or when confirming a page/tab still
  works without manual clicking.
---

# Validate Frontend (Playwright CLI)

Use **Playwright CLI** (`playwright-cli` / `npx playwright-cli`) to validate UI
changes in this repo in **Google Chrome** — same browser target as moz-ui. Prefer
CLI over writing a full `@playwright/test` suite for ad-hoc agent checks.

Official docs: https://playwright.dev/agent-cli/introduction

## Browser target

Validation runs in **installed Google Chrome**, not Playwright's bundled Chromium.

| Setting                 | Value        | Purpose                        |
| ----------------------- | ------------ | ------------------------------ |
| `browserName`           | `"chromium"` | Playwright driver type         |
| `launchOptions.channel` | `"chrome"`   | Launch installed Google Chrome |

Configured in `.playwright/cli.config.json` (aligned with moz-ui). One-time setup:
`npm run playwright:install-browser` installs the Playwright CLI driver; **Google
Chrome must also be installed** on the machine.

## Prerequisites

1. Local app running: `npm run dev` (Traefik + Next on port 3007).
2. Entrypoint: `https://local.moz.com/pro`
3. **Google Chrome** installed on the machine.
4. Playwright CLI driver installed: `npm run playwright:install-browser` (once per
   machine).
5. Repo config auto-loads from `.playwright/cli.config.json` (`channel: "chrome"`,
   ignore HTTPS errors, `storageState` for auth).
6. Prefer `npx playwright-cli` so the project dependency is used even if a global
   install is missing from PATH.
7. For CLI command details beyond this workflow, see
   https://playwright.dev/agent-cli/introduction

## Auth (one-time per machine)

Most routes require login. Auth is restored automatically on `open` via
`storageState` in `.playwright/cli.config.json` (points at
`.playwright/auth.json`).

You usually only need:

```bash
npx playwright-cli open https://local.moz.com/pro --headed
```

Do **not** run `state-load` before `open` — that command requires an open
browser session and will fail with `Browser 'default' is not open`.

### First-time login (or re-save if session expired)

```bash
npx playwright-cli open https://local.moz.com/pro --headed
# Log in manually (may redirect to moz-ui), then:
npx playwright-cli state-save .playwright/auth.json
npx playwright-cli close
```

### Optional: refresh cookies in an open session

Only if the browser is already open and you need to re-apply a saved file:

```bash
npx playwright-cli state-load .playwright/auth.json
npx playwright-cli reload
```

`.playwright/auth.json` is gitignored. Never commit it.

Avoid `--persistent` / shared `userDataDir` unless necessary — leftover Chrome
processes can lock the profile (`Browser is already in use for …/user-data`).

## Standard validation workflow

For each UI change the user wants validated:

1. Confirm `npm run dev` is running (check terminals). If not, start it and wait
   until the app responds.
2. Open a session (reuse if already open):

   ```bash
   npx playwright-cli open <full-url>
   ```

3. Capture page structure and interact via refs (not brittle CSS):

   ```bash
   npx playwright-cli snapshot
   npx playwright-cli click <ref>
   npx playwright-cli fill <ref> "<text>"
   ```

4. Assert the change:

   - Expected copy / headings / controls appear in the snapshot
   - No unexpected error alt-states ("Unable to load…", blank crash)
   - Take evidence: `npx playwright-cli screenshot`

5. Check console errors when relevant:

   ```bash
   npx playwright-cli console
   ```

6. Close when finished:

   ```bash
   npx playwright-cli close
   ```

## URL helpers for this app

| Area                     | Example path (append to `https://local.moz.com/pro`)       |
| ------------------------ | ---------------------------------------------------------- |
| AI Visibility dashboards | `/ai-research/ai-visibility/dashboards`                    |
| Dashboard by id          | `/ai-research/ai-visibility/dashboards/<id>`               |
| Sentiment tab            | `/ai-research/ai-visibility/dashboards/<id>?tab=sentiment` |
| Settings                 | `/ai-research/ai-visibility/dashboards/<id>/settings`      |

Always use absolute Traefik URLs (`https://local.moz.com/pro/...`), not
`http://localhost:3007`, unless the user says otherwise.

## Pass / fail report

Return a short report to the user:

- **URL checked**
- **Result**: PASS / FAIL
- **What was verified** (2–5 bullets)
- **Evidence**: screenshot path under `.playwright/output/` if taken
- **Blockers**: auth missing, app not running, feature flag off, etc.

Do not claim PASS if the page shows a login wall, error alt-state, or the
changed UI is missing from the snapshot.

## Do / don't

**Do**

- Use `snapshot` + element refs for clicks
- Prefer headed mode only when debugging auth: `--headed`
- Rely on `storageState` (`.playwright/auth.json`) for auth across sessions
- Scope checks to the feature the user changed

**Don't**

- Invent a large Playwright test suite for a one-off validation
- Commit `.playwright/user-data`, `.playwright/auth.json`, or screenshots
- Treat Jest unit tests as a substitute for this browser check
- Skip validation when the user explicitly asked to verify the UI
