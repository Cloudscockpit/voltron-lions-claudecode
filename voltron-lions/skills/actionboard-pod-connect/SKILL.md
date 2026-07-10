---
name: actionboard-pod-connect
description: Context and connection guide for linking Voltron Lions to an actionboard.ai cloud AI pod. Use when the user asks to "connect to actionboard", "run a remote pod action", "use the cloud pod", "bridge to Voltron Desktop", or when a mission's Skills Gap analysis flags a remote-pod-action capability.
---

# Connecting Voltron Lions to an actionboard.ai Cloud AI Pod

Voltron Lions runs missions locally inside Claude Code. When a mission needs actions that live on an **Actionboard AI pod** (remote actionlists, RAOARA workflows, Voltron Desktop bridging), Voltron Main delegates through the **actionboard-ai plugin** rather than reimplementing pod connectivity.

## Prerequisites

Check these in order and report the first one missing to the user:

1. **actionboard-ai plugin installed.** Verify the skills `actionboard-ai:connect-pod`, `actionboard-ai:list-actions`, and `actionboard-ai:execute-action` appear in the available skills list. If absent, tell the user to install the Actionboard AI plugin first and stop — do not attempt raw HTTP calls to a pod.
2. **Pod credentials.** The user needs their pod URL and access credentials from their actionboard.ai account (Actionboard AI dashboard → their AI pod). Never ask the user to paste secrets into chat if the plugin's connect flow can prompt for them; never hardcode credentials in files.
3. **Pod reachable.** Connectivity is verified as part of the connect flow below — do not assume a previously connected pod is still healthy.

## Connection flow

Run these as Skill invocations, one at a time, surfacing each result to the user:

| Step | Skill | Purpose |
|------|-------|---------|
| 1 | `actionboard-ai:connect-pod` | Connect to the user's pod and verify health |
| 2 | `actionboard-ai:voltron-status` | Check the Voltron Desktop agent connection status (needed only for desktop-bridged actions) |
| 3 | `actionboard-ai:list-boards` | List actionboards under the pod so the user can pick one |
| 4 | `actionboard-ai:list-actions` | List available actions on the connected pod |

After step 4, the pod's actionlist is known. Report it to the user in a table (action name → description → parameters) before executing anything.

## Executing remote actions during a mission

When Voltron Main's mission plan includes a `remote-pod-action` (see the skills-registry actions map):

1. The action MUST appear in the Lion Assignments table like any local task, with Voltron Main as the responsible Lion and the pod action name in the Task column.
2. Remote actions are gated by the same go/no-go approval as local work — never execute a pod action before the user approves the mission.
3. Execute via `actionboard-ai:execute-action`, passing the action name and parameters. Validate required parameters against the actionlist BEFORE invoking; a malformed call wastes a round-trip to the pod.
4. Record the result (or failure) in the Mission Log exactly like a Lion report: `Status: done | blocked`, plus the pod's response payload summary.
5. For multi-step remote work with fuzzy scope, prefer `actionboard-ai:raoara` (the pod's six-phase Recognize → Acquire → Organize → Apply → Reflect → Amplify workflow) over chaining many individual actions.

## Failure handling

- **Connection refused / auth failure** → report the exact error, suggest the user re-check pod status in their actionboard.ai dashboard, and mark the assignment `blocked`. Do not retry more than once without user input.
- **Action not found on pod** → re-run `actionboard-ai:list-actions` (the actionlist may have changed) and show the user the closest matches.
- **Long-running action** → surface the pod's progress updates if available; do not block the rest of the mission — independent local Lion assignments can proceed in parallel.

## Security rules

- Credentials live in the actionboard-ai plugin's own configuration — Voltron Lions never stores, logs, or echoes them.
- Pod actions run with the pod's permissions, which may touch production systems. Treat every `remote-pod-action` as outward-facing: confirm with the user even if the mission was already approved, when the action's description suggests irreversible effects (deploys, sends, deletes).
