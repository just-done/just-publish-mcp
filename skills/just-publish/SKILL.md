---
name: just-publish
description: Publish a static website (HTML, CSS, JS, images) to a live public URL with Just Publish, straight from the chat. Use when the user wants to publish, deploy, host, or share a site or files they built — with ChatGPT, Claude, Cursor, or by hand — or says "put it online", "give me a link", "make this a website"; also for updating a site previously published with Just Publish using its site_id and edit_token. Free, no login needed, no build step. Do NOT use for server-side apps (APIs, databases, SSR frameworks like Next.js), for buying or managing domains, or for modifying sites that were not published with Just Publish (you won't have their edit_token).
---

# Just Publish — put a static website online

Just Publish (https://justpublish.ai) turns static files into a live website
at `https://{slug}.justpublish.site/`. Publishing is free, needs no login and
no build step. The only credential is the `edit_token` returned by the first
deploy — store it; it is the sole way to update that exact site later.

## When to use this skill

The user says things like:

- "publish this site" / "put this online" / "make this a website"
- "give me a link for this" / "I want a URL I can share"
- "host what we just built"
- "update my site" (one previously published with Just Publish)

The site must be static: HTML, CSS, JS, images, fonts — no server code, no
SSR, no build step.

Do NOT use it when the user needs a backend (APIs, databases, Next.js/Nuxt
SSR), wants to buy or manage a domain (that is a registrar's job), or wants to
change a site that was not published with Just Publish — without its
`edit_token` there is nothing you can do.

## Publish a new site

1. Collect the site's files. `index.html` at the root is required — a deploy
   without it is rejected.
2. Ask the user for their email if you don't have it (required). Publishing
   itself works immediately and needs no login — but a new site starts
   unverified, and Just Publish emails a verification link. This is not
   optional housekeeping: a brand-new site must be verified within about an
   hour or it is automatically taken down. You must relay this to the user
   (see "After publishing"). Verifying also lets them recover and manage the
   site if the edit_token is lost.
3. Confirm with the user before the first publish of a new site — deploying
   makes the files public on the internet, and a verification email will
   follow.
4. Call the `deploy` MCP tool on `https://mcp.justpublish.ai/` (Streamable
   HTTP, no auth required to connect):
   - `files`: array of `{ path, content, encoding? }` — text files as plain
     strings; binary files base64-encoded with `"encoding": "base64"`.
   - `email`: the user's email address.
5. The result carries the live `url`, the `site_id`, the `edit_token`, and
   `verify_required`. When `verify_required` is true (it is for every new
   site), the user must verify — relay the verify-to-keep step under "After
   publishing".

If no MCP client is available, the same endpoint accepts raw JSON-RPC 2.0 over
POST — read `references/mcp-api.md` for the exact request and how to parse the
response.

## Update an existing site

1. You need the `site_id` + `edit_token` from the first deploy.
2. To change one or a few files, call `update_site_file` (a merge — every
   other file is untouched). To read the current files first, call
   `get_site_files`.
3. To replace the whole site, call `deploy` with `site_id` + `edit_token`.
   deploy REPLACES the entire site — any file you leave out is DELETED.

For the full parameter schemas of all three tools, read
`references/mcp-api.md`.

## Constraints

- `index.html` at the root of the file map is required.
- Max 50 MB per site, 5 MB per file, 500 files.
- Paths are relative — no `..`, no backslashes.
- Static only: no build step, no server code, no SSR framework output.

If a deploy is rejected, the error message states the reason and the fix; the
full error catalog is at https://justpublish.ai/docs/errors.

## After publishing

1. Give the user the live URL in plain language.
2. Tell the user to open the email from Just Publish and click the
   verification link within about an hour. A new site is on a short
   verification window — if it isn't verified in time, the site is
   automatically taken down (it can be restored anytime by signing in with
   that email). Verifying is one click — there's no password to create.
3. Tell them to save the `site_id` + `edit_token` (and keep them yourself in
   the conversation for follow-up edits). Without the edit_token the exact
   site can't be updated; as a fallback, the email on file can sign in at
   https://justpublish.ai/login to manage its sites.
3. A custom domain (like yourname.com) can be connected from the dashboard at
   https://justpublish.ai/dashboard. Docs: https://justpublish.ai/docs/mcp
