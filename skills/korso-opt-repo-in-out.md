---
name: Opt a repository into or out of Shepherd coordination
description: >-
  Link a repo to a Shepherd workspace with a committed marker, unlink it repo-wide, or decline
  coordination locally without ever writing a marker.
api: mcp/korso-mcp.yml
server: '@korso/shepherd'
transport: stdio
operations:
  - link
  - unlink
  - decline
generated: '2026-07-19'
method: generated
source: https://korsoai.com/docs/mcp-tools/link
---

# Opt a repository into or out of Shepherd coordination

These three tools are **never gated behind an existing coordination session** — unlike `work`, `done`,
`announce`, and `sync`, they run even in a repo that has never been linked, since that is exactly the
problem they solve. Most agents run `link` once per repo; after that the coordination tools just work.

## Link a repo

```json
{ "workspace": "acme-corp" }
```

`workspace` is optional. Behavior when you omit it:

- **Self-hosted** — there is exactly one valid workspace (the hub's configured one), so it auto-links
  immediately.
- **Korso-hosted** — it lists the workspaces your account belongs to. Exactly one, it auto-picks.
  Several, it lists the slugs and asks you to **check with your human user**, then call `link` again
  with the chosen slug. None, it tells you to create one in the Shepherd dashboard first.

Effects:

- Writes a committed `.shepherd` marker at the repo root containing **only the workspace slug**
  (`{ "workspace": "acme-corp" }`) — **never a token**. Commit it so every teammate who clones the repo
  auto-joins the same workspace.
- Takes effect immediately in the current session; no restart.
- Clears any prior `decline` for this repo.

You can only link to a workspace you are a member of. An invalid slug is rejected — self-host rejects
anything but its single configured workspace; hosted rejects any slug your account is not a member of
and lists the valid ones. If the hub is unreachable while listing hosted workspaces, `link` reports
that and changes nothing rather than erroring.

## Unlink a repo

`unlink` takes no fields — `{}`. It:

- Removes the committed `.shepherd` marker. **This is repo-wide and affects everyone.**
- Automatically records a local decline so you are not immediately re-prompted to link on the next
  session or tool call.
- Tears down any active coordination session immediately — the hub is told the agent left, so presence
  and claims drop right away rather than lingering until the session goes stale (~2 minutes without a
  heartbeat).

## Decline coordination

`decline` takes no fields — `{}`. It is a "don't ask again" for **one user**, with no marker created or
committed. It is **local-only**: a teammate in the exact same repo can still link it themselves.

If the repo is already linked, `decline` does nothing to the marker and returns an advisory pointing
you to `unlink` instead — a committed marker always wins over a local decline.

## Choosing between them

| Situation | Tool |
|---|---|
| Repo should coordinate with the team | `link` |
| Repo was linked and should stop, for everyone | `unlink` |
| Repo was never linked and you personally don't want to be asked again | `decline` |

Run `link` at any time to reverse a `decline` or an `unlink`.
