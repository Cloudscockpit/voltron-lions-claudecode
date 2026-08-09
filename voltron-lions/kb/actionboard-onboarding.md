# KB — Actionboard.ai Signup & Voltron Castle Desktop Setup

End-to-end onboarding: create an Actionboard AI account, install the Voltron Castle Desktop app, and get connected. Written for non-technical users; developer notes are marked.

## Part 1 — Sign up for Actionboard.ai

1. Go to **[actionboard.ai](https://actionboard.ai)** in your browser.
2. Choose **Sign up** and create your account (work email recommended — your team's boards attach to it).
3. Complete any email verification the site asks for.
4. After first login, your **AI pod** is where actionboards, actionlists, and flows live. Note where the dashboard shows your **pod details/credentials** — the Claude-side connection (via the Actionboard AI plugin) will ask for them later.

> Claude can help with this signup using its browser actions — it will open the page and guide you, but it will hand the browser to you for the password and any verification step. Claude never types credentials.

## Part 2 — Install Voltron Castle Desktop

Voltron Castle Desktop is the desktop app for your pod: boards, the Action Panel, the Flow tab, and the Voltron Desktop agent bridge.

1. Download the latest release for your platform (macOS/Windows) from the releases page:
   **https://github.com/Cloudscockpit/actionboard-desktop-app/releases**
2. Open the installer:
   - **macOS:** open the `.dmg`, drag the app to Applications. First launch: right-click → **Open** if macOS warns about an unidentified developer.
   - **Windows:** run the installer `.exe` and follow the prompts.
3. Launch the app and **sign in with your actionboard.ai account** (Part 1).
4. In the app, confirm your pod loads: you should see your actionboards and the Action Panel.

**Developer alternative:** clone `Cloudscockpit/actionboard-desktop-app` and run it from source (Electron; see that repo's README for `npm install` / run scripts).

## Part 3 — Verify the bridge from Claude

Once the desktop app is running and signed in:

1. In Claude Code / Cowork, ask: **"check voltron status"** — this uses `actionboard-ai:voltron-status` to verify the Voltron Desktop agent connection.
2. Then: **"connect to my actionboard pod"** — the `actionboard-pod-connect` skill walks the connect → health → boards → actions flow.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| No releases on the download page | Ask your team for the current build, or use the developer alternative above |
| Desktop app opens but pod won't load | Check you're signed in with the same account that owns the pod; check network/VPN |
| "voltron status" reports disconnected | Make sure Voltron Castle Desktop is running and signed in, then retry |
| Claude can't find the actionboard-ai skills | Install the Actionboard AI plugin first (see the pod-connect skill's prerequisites) |
