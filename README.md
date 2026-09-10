# Just Publish MCP Server

**Publish the website you built with AI to a live public URL, straight from chat, with no setup.**

Describe a site to your AI assistant and it publishes the files and hands back a
live URL you can open and share. Static files in, working link out, with no build
step. Built for the person who made a site with AI and just wants it online.

- **Website:** https://justpublish.ai
- **Claude connector:** `https://mcp.justpublish.ai/claude` (OAuth)
- **Endpoint for manually configured clients:** `https://mcp.justpublish.ai/`
- **Registry name:** `ai.justpublish/just-publish`
- **Transport:** Streamable HTTP (remote, nothing to install)

## In Claude

Add Just Publish as a custom connector with the address
`https://mcp.justpublish.ai/claude`. In Claude Code it is one command:

```
claude mcp add --transport http just-publish https://mcp.justpublish.ai/claude
```

Either way you sign in first, with Google or a code sent to your email. Every
site you publish is filed under your account, so you can find it, change it or
take it down at any time, from Claude or from your dashboard.

Step by step: https://justpublish.ai/docs/mcp/claude

Authorization is OAuth, with dynamic client registration and PKCE (S256), so the
client discovers everything it needs from
`https://mcp.justpublish.ai/.well-known/oauth-protected-resource/claude`. A call
without a bearer token is refused with `401` and that same pointer in the
`WWW-Authenticate` header.

### Tools on the connector

| Tool | What it does |
| --- | --- |
| `deploy` | Publish a static site (HTML, CSS, JS, images). Returns the live `url` and a `site_id`. The site is filed under the signed-in account, so no email is asked for. It can also replace an existing site in full, which takes that site's `site_id` and `edit_token` together. |
| `list_sites` | List the sites the signed-in person has published: live URL, `site_id`, when it was published and last updated, any connected custom domain, recent views. Takes no parameters. |
| `get_site_files` | Read a published site's files so the assistant can make a targeted edit instead of rebuilding from memory. Needs only `site_id` for the signed-in person's own sites. |
| `update_site_file` | Change one or a few files in place, leaving every other file untouched. Needs only `site_id` for the signed-in person's own sites. |

## Other MCP clients

For a client you configure by hand, the endpoint is `https://mcp.justpublish.ai/`:

```json
{
  "mcpServers": {
    "just-publish": {
      "type": "streamable-http",
      "url": "https://mcp.justpublish.ai/"
    }
  }
}
```

It serves `deploy`, `get_site_files` and `update_site_file`. Here `deploy` takes
an email address for the site and returns an `edit_token` alongside the live URL:
a link is sent to that address to confirm it, and the token is how a client reads
or changes the site afterwards.

## Resources

Static markdown guidance the client can load into context before it picks a tool:

| Resource | Purpose |
| --- | --- |
| `just-publish-overview` | What Just Publish is and which tool to use when. |
| `just-publish-edit-lifecycle` | The read, edit, update loop. |
| `just-publish-file-conventions` | Paths, encoding, size limits, routing. |
| `just-publish-invariants` | `index.html` required; `deploy` replaces a whole site while `update_site_file` merges. |

## Agent Skill (ChatGPT / Codex / any SKILL.md runtime)

This repo also hosts the **`just-publish` Agent Skill**, a
[SKILL.md-format](https://learn.chatgpt.com/docs/build-skills) package that
teaches an agent the whole publish flow before it ever calls a tool.

To install in Codex or ChatGPT, ask the assistant to install the skill from this
tree:

```
https://github.com/just-done/just-publish-mcp/tree/main/skills/just-publish
```

(Codex's skill-installer takes a GitHub tree URL.) For a manual install, copy
`skills/just-publish/` into `$CODEX_HOME/skills/` (default `~/.codex/skills/`)
or your runtime's equivalent, for example `~/.claude/skills/`.

The same package is served for discovery scanners and other runtimes at
[justpublish.ai/.well-known/agent-skills/index.json](https://justpublish.ai/.well-known/agent-skills/index.json)
(Agent Skills Discovery RFC, sha256-digested). The canonical source lives in the
Just Publish server codebase; this tree is its published mirror and is kept in
lockstep. The two are compared on every server deploy.

## Who it's for

Non-technical builders, and the AI agents working for them. Someone describes a
page, the assistant generates the files, and the site is hosted in one step. If
you can describe a page, you can publish it.

## Good to know

- **Static sites only.** Files in, URL out. No frameworks and no build pipelines.
- **Every site needs an `index.html`** at the top level.
- **Custom domains.** Connect your own domain at [justpublish.ai](https://justpublish.ai).

## About this repository

Public metadata and registry listing for the Just Publish MCP server:
`server.json`, `.mcp.json`, icons, and this README. The service runs at
`mcp.justpublish.ai`; its source is not part of this repo.

## License

MIT. See [LICENSE](./LICENSE).
