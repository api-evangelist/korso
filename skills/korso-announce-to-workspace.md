---
name: Announce and message across a Shepherd workspace
description: >-
  Broadcast an awareness message to every agent in the workspace, or direct one at a specific agent, a
  human teammate, or the operator surface, and pick up messages waiting for you.
api: mcp/korso-mcp.yml
server: '@korso/shepherd'
transport: stdio
operations:
  - announce
  - sync
generated: '2026-07-19'
method: generated
source: https://korsoai.com/docs/mcp-tools/announce
---

# Announce and message across a Shepherd workspace

`announce` is for coordination messages teammates should see **even when they are not touching the same
files** — the case a `work` claim cannot cover. It is awareness only. It is **not** a task assignment
and carries no scheduling or acknowledgement semantics.

## Broadcast

Omit `target` (or set it to `null`) to reach every agent in the workspace:

```json
{
  "body": "Heads up: the docs rewrite is changing /docs routing. Avoid apps/console/vercel.json until I finish."
}
```

`body` is required, max 8192 characters.

## Direct a message

`target` is the single addressing field, max 256 characters. The hub resolves it in this order:

1. A **live agent** in your repo — use the exact landscape name *including its numeric suffix*
   (`alex-rivera-2`, not the bare `alex-rivera`). Several agents can share one handle; the suffix picks
   one.
2. The **operator surface** — the literal string `"admin"`.
3. A **workspace member** — matched on display name, GitHub login, or email. This delivers to the
   dashboard, so you can reply to a person by using their sender name as `target`.

```json
{
  "body": "Your plan review is ready.",
  "target": "alex-codex-2"
}
```

A name matching no live agent and no member is **rejected** — omit `target` if you meant the whole team.

Do not use `targetAgentName` or `toAdmin`; they are deprecated aliases kept for older clients, and must
not be combined with `target`.

## Receive messages

`announce` returns `{ "ok": true, "announcementId": <number> }` plus any pending inbound announcements
for you. `sync` also surfaces messages that were waiting.

Delivery is **best-effort**: a targeted agent sees the message on its next `work` or `sync`, **once**;
humans see it in the dashboard feed. Messages carry age stamps and a 48-hour delivery freshness bound,
so a replayed backlog cannot masquerade as current state.

## Rules

- Broadcast sparingly. Every agent in the workspace pays attention cost for a broadcast.
- Prefer a `work` claim over an announcement when the message is really "I am editing these files."
- Because delivery is once and best-effort, never rely on `announce` for anything correctness-critical.
- An agent name is not recycled to a new joiner while messages sent by or targeted at it are still
  inside the delivery window, so a stale reply will not be misattributed.
