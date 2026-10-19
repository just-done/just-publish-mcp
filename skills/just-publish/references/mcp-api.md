# Just Publish: MCP API reference

Just Publish is one MCP server over Streamable HTTP (JSON-RPC 2.0 over POST),
reached at two addresses that serve the same four tools:

- `https://mcp.justpublish.ai/` for Cursor, VS Code, a CLI agent, a custom
  connector, or raw JSON-RPC.
- `https://mcp.justpublish.ai/claude` for the Claude connector (exactly that,
  with no trailing slash). In Claude Code:

```
claude mcp add --transport http just-publish https://mcp.justpublish.ai/claude
```

## Sign-in

Authorization is OAuth 2.1 with dynamic client registration and PKCE (S256).
The client discovers everything it needs from
`https://mcp.justpublish.ai/.well-known/oauth-protected-resource` (for
`https://mcp.justpublish.ai/`) or
`https://mcp.justpublish.ai/.well-known/oauth-protected-resource/claude` (for
`https://mcp.justpublish.ai/claude`); a call with no bearer token is refused
with `401` and that same pointer in the `WWW-Authenticate` header. The person
signs in with Google or a code sent to their email address. Each address has
its own audience-bound token.

Every site is filed under the signed-in account, so `deploy` has no `email`
parameter, and `get_site_files` / `update_site_file` authorize on ownership
with `site_id` alone for that account's own sites.

## `list_sites`

Served at `https://mcp.justpublish.ai/` and `https://mcp.justpublish.ai/claude`.
No parameters at all; the identity comes from the access token.

Returns, most recently updated first: the live `url`, `site_id`, when each site
was published and last updated, any connected custom domain, and how many views
it got in the last 48 hours. Use it before `get_site_files` or
`update_site_file` when you do not have a `site_id`, instead of asking the user
for a code they may not have.

## `deploy`: publish or replace a whole site

| Argument | Type | Required | Notes |
|---|---|---|---|
| `files` | array of `{ path, content, encoding? }` | yes | Must include `index.html` at the root. Text content as plain strings (`utf-8`, the default); binary content base64-encoded with `"encoding": "base64"`. |
| `site_id` | string | no | Pass, with `edit_token`, to publish over an existing site. Omit to create a new one. |
| `edit_token` | string | no | Required when `site_id` is passed. Passing `site_id` on its own is rejected. |

Returns `url`, `site_id`, `files_uploaded`, `total_bytes`, `created`, and an
`edit_token` on first create. The site belongs to the signed-in account, so do
not show that token to the user or ask them to save it. For a later change,
prefer `update_site_file` with `site_id` alone.

deploy REPLACES the whole site: any file you leave out is DELETED.

## Reading and editing a site

### `update_site_file`: merge-edit specific files

| Argument | Type | Required | Notes |
|---|---|---|---|
| `site_id` | string | yes | From the first deploy, or from `list_sites`. |
| `edit_token` | string | no | Not needed for the signed-in account's own sites. |
| `files` | array of `{ path, content, encoding? }` | yes | Only these files are written; every other file survives. Cannot delete files and cannot remove `index.html`. |
| `expected_version` | integer | no | Recommended: the `version` from `get_site_files`. If the site changed in the meantime the update is rejected with a conflict; re-read and re-apply. Omit for last-writer-wins. |

Returns: `site_id`, `url`, `version`, `files_written`, `files_total`.

### `get_site_files`: read what is currently live

| Argument | Type | Required | Notes |
|---|---|---|---|
| `site_id` | string | yes | |
| `edit_token` | string | no | Not needed for the signed-in account's own sites. |
| `paths` | array of strings | no | Limit the read to specific files. Omit to list everything. |

Returns the site's files and its current `version` (use it as
`expected_version` in `update_site_file`).

The recommended edit loop: `get_site_files`, modify content, then
`update_site_file` with `expected_version`.

## Raw JSON-RPC (no MCP client available)

The server accepts plain JSON-RPC 2.0 over POST. The `accept` header must name
BOTH `application/json` and `text/event-stream` (a bare `*/*` is rejected by
spec), and every call needs an `authorization: Bearer` header carrying an
access token from the sign-in:

```
curl -s https://mcp.justpublish.ai/ \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -H 'authorization: Bearer <access token>' \
  -d '{
    "jsonrpc": "2.0", "id": 1, "method": "tools/call",
    "params": {
      "name": "deploy",
      "arguments": {
        "files": [
          { "path": "index.html", "content": "<!doctype html><h1>Hi</h1>" }
        ]
      }
    }
  }'
```

The response is SSE-framed: take the first `data:` line and parse it as JSON.
The live URL is `result.structuredContent.url`; keep
`result.structuredContent.site_id` for later changes.

## Errors

Every rejection returns a tool error whose message states the reason and the
fix (a missing root `index.html`, an oversized file, an invalid path). The
complete, always-current error catalog, with every error id, the exact message
and the recovery step, is at:

https://justpublish.ai/docs/errors

## Limits

- 50 MB per site, 5 MB per file, 500 files.
- Paths are relative to the site root: no `..`, no backslashes.
- `index.html` at the root is required on every `deploy`.

Full product docs for agents: https://justpublish.ai/docs/mcp
Connecting, step by step: https://justpublish.ai/docs/mcp/connect
Source and issues: https://github.com/just-done/just-publish-mcp
