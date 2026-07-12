# Just Publish — MCP API reference

Endpoint: `https://mcp.justpublish.ai/` — MCP over Streamable HTTP
(JSON-RPC 2.0 over POST). No auth is required to connect; the only
credential in the system is the per-site `edit_token`.

## Tools

### `deploy` — publish or replace a whole site

| Argument | Type | Required | Notes |
|---|---|---|---|
| `files` | array of `{ path, content, encoding? }` | yes | Must include `index.html` at the root. Text content as plain strings (`utf-8`, the default); binary content base64-encoded with `"encoding": "base64"`. |
| `email` | string | yes | Associated with the site. Captured unverified at first deploy; verification is only required later for custom domains and recovery. |
| `site_id` | string | no | Pass to publish over an existing site. Omit to create a new one. |
| `edit_token` | string | no | Required when `site_id` is passed. |

Returns (in `structuredContent`): `url`, `site_id`, `edit_token`,
`files_uploaded`, `total_bytes`, `created`.

deploy REPLACES the whole site — any file you leave out is DELETED. For
small edits prefer `update_site_file`.

### `update_site_file` — merge-edit specific files

| Argument | Type | Required | Notes |
|---|---|---|---|
| `site_id` | string | yes | From the first deploy. |
| `edit_token` | string | yes | From the first deploy. |
| `files` | array of `{ path, content, encoding? }` | yes | Only these files are written; every other file survives. Cannot delete files and cannot remove `index.html`. |
| `expected_version` | integer | no | Recommended: the `version` from `get_site_files`. If the site changed in the meantime the update is rejected with a conflict — re-read and re-apply. Omit for last-writer-wins. |

Returns: `site_id`, `url`, `version`, `files_written`, `files_total`.

### `get_site_files` — read what's currently live

| Argument | Type | Required | Notes |
|---|---|---|---|
| `site_id` | string | yes | |
| `edit_token` | string | yes | |
| `paths` | array of strings | no | Limit the read to specific files. Omit to list everything. |

Returns the site's files and its current `version` (use it as
`expected_version` in `update_site_file`).

The recommended edit loop: `get_site_files` → modify content →
`update_site_file` with `expected_version`.

## Raw JSON-RPC (no MCP client available)

The endpoint accepts plain JSON-RPC 2.0 over POST. The `accept` header must
name BOTH `application/json` and `text/event-stream` (a bare `*/*` is
rejected by spec):

```
curl -s https://mcp.justpublish.ai/ \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{
    "jsonrpc": "2.0", "id": 1, "method": "tools/call",
    "params": {
      "name": "deploy",
      "arguments": {
        "email": "user@example.com",
        "files": [
          { "path": "index.html", "content": "<!doctype html><h1>Hi</h1>" }
        ]
      }
    }
  }'
```

The response is SSE-framed: take the first `data:` line and parse it as
JSON. The live URL is `result.structuredContent.url`; keep
`result.structuredContent.site_id` and `.edit_token` for updates.

## Errors

Every rejection returns a tool error whose message states the reason and the
fix (e.g. a missing root `index.html`, an oversized file, an invalid path).
The complete, always-current error catalog — every error id, the exact
message, and the recovery step — is at:

https://justpublish.ai/docs/errors

## Limits

- 50 MB per site, 5 MB per file, 500 files.
- Paths are relative to the site root — no `..`, no backslashes.
- `index.html` at the root is required on every `deploy`.

Full product docs for agents: https://justpublish.ai/docs/mcp
Source and issues: https://github.com/just-done/just-publish-mcp
