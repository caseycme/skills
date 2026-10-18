# Validate Frontend

Validate recent frontend UI changes using Playwright CLI against the local app in
**Google Chrome** (`.playwright/cli.config.json` → `launchOptions.channel: "chrome"`,
same as moz-ui).

## Instructions

1. Read and follow `.cursor/skills/validate-frontend/SKILL.md`.
2. Confirm Google Chrome is installed and `npm run playwright:install-browser` has
   been run once on this machine.
3. Confirm `npm run dev` is running. If not, start it and wait for readiness.
4. Ask which URL/route to check if the user did not specify one. Default to the
   feature touched by the latest code changes when that is clear.
5. Run Playwright CLI (`npx playwright-cli`) to open the page in Chrome, snapshot,
   interact as needed, screenshot, and check console errors.
6. If blocked by login, tell the user to complete the one-time auth steps in the
   skill (`--headed` login + `state-save`), then retry.
7. Report PASS/FAIL with URL, what was verified, evidence paths, and blockers.
