---
name: just-publish
description: Publish a static website (HTML, CSS, JS, images) to a live public URL with Just Publish, straight from the chat. Use when the user wants to publish, deploy, host, or share a site or files they built, with Claude, ChatGPT, Cursor, or by hand, or says "put it online", "give me a link", "make this a website"; also for changing a site published earlier with Just Publish. Publishing is free and there is no build step. The MCP server is https://mcp.justpublish.ai/ (in Claude, the connector at https://mcp.justpublish.ai/claude); the person signs in once with a free account, every site is filed under it, and four tools are served. Do NOT use for server-side apps (APIs, databases, SSR frameworks like Next.js), for buying or managing domains, or for changing a site that was not published with Just Publish.
---

# Just Publish: put a static website online

Just Publish (https://justpublish.ai) turns static files into a live website at
`https://{slug}.justpublish.site/`. Publishing is free and there is no build
step: files in, working link out.

## Connect and sign in

The MCP server is `https://mcp.justpublish.ai/`. In Claude it is the connector
at `https://mcp.justpublish.ai/claude`, added from the connector directory or
as a custom connector; in Claude Code it is one command:

```
claude mcp add --transport http just-publish https://mcp.justpublish.ai/claude
```

Cursor, VS Code, a CLI agent or any other MCP client adds
`https://mcp.justpublish.ai/` instead. Either way the person signs in the first
time, with Google or a code sent to their email address. Authorization is
OAuth 2.1 and the client discovers what it needs from the server, so the
advanced client-id and client-secret fields stay empty.

Both addresses serve the same four tools: `deploy`, `list_sites`,
`get_site_files` and `update_site_file`. Every site is filed under the
signed-in account, which changes how you work:

- `deploy` takes no email address. Do not ask the user for one.
- `list_sites` returns the sites that account has already published, so you can
  find a site instead of asking for an id the user does not have.
- `get_site_files` and `update_site_file` need only a `site_id` for that
  account's own sites.
- Report the live `url` and stop. The account is what makes the site theirs to
  find, change and take down later.

Step by step for people: https://justpublish.ai/docs/mcp/connect (Claude:
https://justpublish.ai/docs/mcp/claude)

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
3. Call `deploy` with `files`: an array of `{ path, content, encoding? }`. Text
   files as plain strings; binary files base64-encoded with
   `"encoding": "base64"`.
4. The result carries the live `url` and the `site_id`. Give the user the URL
   in plain language; the link is what they asked for. The result also carries
   an `edit_token`; you do not need it for the account's own sites, so do not
   ask the user to save it.

If no MCP client is available, the server accepts raw JSON-RPC 2.0 over POST
with a bearer token from the sign-in. Read `references/mcp-api.md` for the
exact request and how to parse the response.

## Change a site later

1. Find the site with `list_sites` (on `https://mcp.justpublish.ai/` or
   `https://mcp.justpublish.ai/claude`, whichever you are connected to).
2. Read before you write. `get_site_files` returns what is currently live plus
   a `version`.
3. To change one or a few files, call `update_site_file`. It merges: every file
   you do not name survives. Pass the `version` you just read as
   `expected_version`, so a change someone else made in the meantime is
   rejected instead of silently overwritten.
4. To replace a whole site, call `deploy` with `site_id` and the `edit_token`
   it returned. deploy REPLACES the entire site: any file you leave out is
   DELETED. Prefer `update_site_file`, which needs only the `site_id`.

For the full parameter schema of every tool, read `references/mcp-api.md`.

## Constraints

- `index.html` at the root of the file map is required.
- Max 50 MB per site, 5 MB per file, 500 files.
- Paths are relative: no `..`, no backslashes.
- Static only: no build step, no server code, no SSR framework output.

If a deploy is rejected, the error message states the reason and the fix. The
full error catalog is at https://justpublish.ai/docs/errors.

## After publishing

1. Give the user the live URL in plain language.
2. There is nothing for the user to write down. The site is already under their
   account, and it can be found again with `list_sites` or at
   https://justpublish.ai/dashboard.
3. A custom domain (like yourname.com) is connected from the dashboard at
   https://justpublish.ai/dashboard. Agent docs: https://justpublish.ai/docs/mcp
