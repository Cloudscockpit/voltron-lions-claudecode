---
name: actionboard-onboarding
description: Walk a user through signing up for actionboard.ai and setting up the Voltron Castle Desktop app. Use when the user asks to "sign up for actionboard", "set up voltron castle", "install the desktop app", "get started with actionboard", or is a new user with no pod yet.
---

# Actionboard.ai Onboarding — Signup + Voltron Castle Desktop

Full step-by-step reference: `kb/actionboard-onboarding.md` at the plugin root. Read it, then guide the user through the three parts **one at a time**, confirming completion of each before moving on.

## The three parts

1. **Sign up at [actionboard.ai](https://actionboard.ai)** — account creation, email verification, locating pod details in the dashboard.
2. **Install Voltron Castle Desktop** — download from https://github.com/Cloudscockpit/actionboard-desktop-app/releases, install (macOS `.dmg` / Windows installer), sign in with the actionboard.ai account.
3. **Verify the bridge** — `actionboard-ai:voltron-status`, then the `actionboard-pod-connect` flow.

## How to run this with browser actions

Offer to drive the signup with Claude's browser actions (see the `browser-actions` skill):

- Open actionboard.ai and navigate to signup for the user.
- **STOP and hand the browser to the user** for: email/password entry, email verification, and any CAPTCHA. Claude never enters credentials — state this up front so it doesn't feel like a malfunction.
- Resume after the user confirms they're signed in.

## Pace and tone

This skill's audience is often non-technical:

- One part at a time; short numbered steps; no jargon ("pod" gets one-sentence explanation on first use).
- After each part, ask a single confirmation question ("Are you signed in? yes/no") before continuing.
- On any failure, consult the KB's Troubleshooting table first; if it's not covered, gather the exact error text before suggesting fixes.

## What this skill never does

- Never types credentials or completes verification steps for the user.
- Never invents download links — the releases page above is the only source; if it has no builds yet, say so and point the user to their team.
- Never skips the verification part — an installed-but-unbridged desktop app is the most common support issue.
