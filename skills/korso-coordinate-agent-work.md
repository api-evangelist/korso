---
name: Coordinate agent work in a shared Shepherd workspace
description: >-
  Claim a file area before editing, keep the claim alive, and release it when done, so multiple AI
  coding agents in the same repo stop producing merge conflicts and overwriting each other.
api: mcp/korso-mcp.yml
server: '@korso/shepherd'
transport: stdio
operations:
  - sync
  - work
  - done
generated: '2026-07-19'
method: generated
source: https://korsoai.com/docs
---

# Coordinate agent work in a shared Shepherd workspace

Shepherd claims are **advisory**: they make what you are about to touch visible to every other agent
and human in the workspace. They do not lock the filesystem. The value comes from claiming *before*
you edit and releasing promptly.

## Prerequisites

The `shepherd` MCP server must be configured with `HUB_URL` and either `SHEPHERD_TOKEN` (Korso-hosted)
or `TEAM_TOKEN` (self-hosted). The server fails at startup if neither is set. The repo must already be
linked — see `korso-opt-repo-in-out.md`.

`work`, `done`, `announce`, and `sync` are gated on an active coordination session. `link`, `unlink`,
and `decline` are not.

## Steps

1. **Read the room first.** Call `sync` (no arguments) before planning and before editing shared files.
   It returns the current workspace landscape: active claims, your own active claims, recent
   announcements, and presence. If someone already holds a claim overlapping what you intended to do,
   route around it or `announce` to negotiate rather than editing into their claim.

2. **Claim the area.** Call `work` before making a meaningful change:

   ```json
   {
     "intent": "Update Shepherd quickstart docs",
     "pathGlobs": ["docs-site/**", "shepherd/docs/shepherd-mcp-quickstart.md"],
     "ttlSeconds": 3600
   }
   ```

   - `intent` is required, max 2048 characters. Write it for a teammate, not for yourself.
   - `pathGlobs` is required, max 64 entries. Choose globs specific enough to help teammates route
     around you — `src/**` when you are editing one file is antisocial.
   - `ttlSeconds` is optional; the server default applies when omitted.

   **Save the returned `workItemId`.** You need it to release the claim.

3. **Do the work.** Calling `work` or `sync` again refreshes your presence and renews your active
   claims without creating a new claim, so a long task should periodically `sync`.

4. **Release the claim.** Call `done` with the id from step 2:

   ```json
   { "workItemId": "2f4d5b46-1a6e-49d8-9302-9bdf73aeb21a" }
   ```

   Call `done` **even if the task ended without a code change** — an abandoned claim otherwise reserves
   attention until its TTL expires.

## Rules

- One claim per coherent unit of work. Do not batch unrelated areas into one `work` call.
- A claim that expires by TTL before `done` is treated as no longer active; the work is not tracked.
- If the hub is unreachable, Shepherd reports the session is proceeding uncoordinated rather than
  blocking. Keep working, but assume nobody can see your claim.
- Shepherd returns one-line human-readable advisories rather than error codes. Read them — they
  usually tell you the exact next call to make.
- Never put a token in a committed file. The `.shepherd` marker holds only the workspace slug.
