---
name: just-publish
description: Publish a static website (HTML, CSS, JS, images) to a live public URL with Just Publish, straight from the chat. Use when the user wants to publish, deploy, host, or share a site or files they built, with Claude, ChatGPT, Cursor, or by hand, or says "put it online", "give me a link", "make this a website"; also for changing a site published earlier with Just Publish. Publishing is free and there is no build step. In Claude it is a signed-in connector at https://mcp.justpublish.ai/claude, with four tools; a client you configure by hand points at https://mcp.justpublish.ai/, where deploy takes the user's email address and returns an edit_token. Do NOT use for server-side apps (APIs, databases, SSR frameworks like Next.js), for buying or managing domains, or for changing a site that was not published with Just Publish.
---

# Just Publish: put a static website online

Just Publish (https://justpublish.ai) turns static files into a live website at
`https://{slug}.justpublish.site/`. Publishing is free and there is no build
step: files in, working link out.

## Know which endpoint you are on

Just Publish serves two MCP endpoints and they are different contracts. Call
`tools/list` first and look for `list_sites`: if it is there you are on the
Claude connector at `https://mcp.justpublish.ai/claude`; if it is not, you are
on `https://mcp.justpublish.ai/`, the endpoint for manually configured
clients.

### `https://mcp.justpublish.ai/claude`, the Claude connector

In Claude, Just Publish is added from the connector directory or as a custom
connector at that address. In Claude Code it is one command:

```
claude mcp add --transport http just-publish https://mcp.justpublish.ai/claude
```

Either way the person signs in first, with Google or a code sent to their email
address. Authorization is OAuth and the client discovers what it needs from the
server, so the advanced client-id and client-secret fields stay empty.

This endpoint serves four tools: `deploy`, `list_sites`, `get_site_files` and
`update_site_file`. Every site is filed under the signed-in account, which
changes how you work:

- `deploy` takes no email address here. Do not ask the user for one.
- `list_sites` returns the sites that account has already published, so you can
  find a site instead of asking for an id the user does not have.
- `get_site_files` and `update_site_file` need only a `site_id` for that
  account's own sites.
- Report the live `url` and stop. The account is what makes the site theirs to
  find, change and take down later.

Step by step for people: https://justpublish.ai/docs/mcp/claude

### `https://mcp.justpublish.ai/`, for a client you configure by hand

This is the address a hand-configured MCP client points at: Cursor, a CLI
agent, a custom connector, or raw JSON-RPC. It serves three tools: `deploy`,
`get_site_files` and `update_site_file`.

Here a site is identified by the email address given at publish time and by the
`edit_token` the first `deploy` returns:

- `deploy` requires `email`. Ask the user for theirs if you do not have it.
- The `edit_token` from the first `deploy` is the handle on that exact site.
  Keep it in the conversation and give it to the user.
- Just Publish sends a link to that address to confirm it. A new site must be
  confirmed within about an hour or it is taken offline automatically, and
  confirming the address brings it back. Relay this to the user; it is not
  optional housekeeping.

## When to use this skill

The user says things like:

- "publish this site" / "put this online" / "make this a website"
- "give me a link for this" / "I want a URL I can share"
- "host what we just built"
- "update my site" (one published earlier with Just Publish)

The site must be static: HTML, CSS, JS, images, fonts. No server code, no SSR,
no build step.

Do NOT use it when the user needs a backend (APIs, databases, Next.js or Nuxt
SSR), wants to buy or manage a domain (that is a registrar's job), or wants to
change a site that was not published with Just Publish.

## Publish a new site

1. Collect the site's files. `index.html` at the root is required; a deploy
   without it is rejected.
2. Confirm with the user before the first publish of a new site. Deploying puts
   the files on the public internet.
3. Call `deploy` with:
   - `files`: array of `{ path, content, encoding? }`. Text files as plain
     strings; binary files base64-encoded with `"encoding": "base64"`.
   - `email`: the user's address. Required on `https://mcp.justpublish.ai/`
     and absent from the connector's schema, which takes the address from the
     signed-in account.
4. The result carries the live `url` and the `site_id`. Give the user the URL
   in plain language; the link is what they asked for.
5. On `https://mcp.justpublish.ai/` the result also carries the `edit_token`
   and `verify_required`. Keep the token and relay the confirm-the-address step
   under "After publishing".

If no MCP client is available, both endpoints accept raw JSON-RPC 2.0 over
POST. Read `references/mcp-api.md` for the exact request and how to parse the
response.

## Change a site later

1. Find the site. On `https://mcp.justpublish.ai/claude`, call `list_sites`.
   On `https://mcp.justpublish.ai/`, use the `site_id` and `edit_token` from
   the first deploy.
2. Read before you write. `get_site_files` returns what is currently live plus
   a `version`.
3. To change one or a few files, call `update_site_file`. It merges: every file
   you do not name survives. Pass the `version` you just read as
   `expected_version`, so a change someone else made in the meantime is
   rejected instead of silently overwritten.
4. To replace a whole site, call `deploy` with `site_id` and `edit_token`
   together. deploy REPLACES the entire site: any file you leave out is
   DELETED. On the connector prefer `update_site_file`, which needs only the
   `site_id`.

For the full parameter schema of every tool on both endpoints, read
`references/mcp-api.md`.

## Constraints

- `index.html` at the root of the file map is required.
- Max 50 MB per site, 5 MB per file, 500 files.
- Paths are relative: no `..`, no backslashes.
- Static only: no build step, no server code, no SSR framework output.

If a deploy is rejected, the error message states the reason and the fix. The
full error catalog is at https://justpublish.ai/docs/errors.

## After publishing

1. Give the user the live URL in plain language.
2. On the Claude connector at `https://mcp.justpublish.ai/claude` there is
   nothing for the user to write down. The site is already under their
   account, and it can be found again with `list_sites` or at
   https://justpublish.ai/dashboard.
3. On `https://mcp.justpublish.ai/`, tell the user to open the email from Just
   Publish and click the link within about an hour, or the new site is taken
   offline automatically; confirming the address brings it back. Tell them to
   keep the `site_id` and `edit_token` too, and keep both yourself so you can
   make the next edit.
4. A custom domain (like yourname.com) is connected from the dashboard at
   https://justpublish.ai/dashboard. Agent docs: https://justpublish.ai/docs/mcp
