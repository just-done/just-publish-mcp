# Just Publish — MCP Server

**Publish a static website to a live public URL, straight from chat.**

[Just Publish](https://justpublish.ai) is an AI-native website-hosting service
for people who want a real business website but don't want to touch code, a
build step, a dashboard, or a deploy config. You describe the site to the AI
assistant you already use — ChatGPT, Claude, Gemini, or any MCP-enabled client —
and it publishes the site for you and hands back a live, public URL.

- **Website:** https://justpublish.ai
- **MCP endpoint:** `https://mcp.justpublish.ai/` (Streamable HTTP)
- **Registry name:** `ai.justpublish/just-publish`
- **Transport:** Streamable HTTP (remote server — nothing to install locally)

## Who it's for

Non-technical business owners. You work with an AI assistant; the assistant does
the publishing through this server. No account, no git, no CLI, no build. Your
email is captured on the first publish (unverified) so you can recover and claim
your site later.

## Connecting

Add the remote MCP server to any client that supports the Streamable HTTP
transport:

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

Then ask your assistant to publish a site — e.g. *"build and publish a landing
page for my bakery."* It returns the public URL.

## What the server exposes

This is a remote MCP server. Once connected, a client sees the following surface.

### Tools

| Tool | What it does |
| --- | --- |
| `deploy` | Publish a new static site (HTML/CSS/JS/images) to a public URL, or fully replace an existing site's files. Returns the live URL, a `site_id`, and an `edit_token` (save it — it's the only key to update the site later). |
| `get_site_files` | Read the files of a site you already published (manifest + text contents + current version) so the assistant can make a targeted edit instead of regenerating the whole site. Requires `site_id` + `edit_token`. |
| `update_site_file` | Change one or a few files in place (merge), leaving every other file untouched. Uses `expected_version` to avoid clobbering a newer change. Requires `site_id` + `edit_token`. |

### Resources

Static, assistant-facing guidance resources (`text/markdown`) that let a client
load Just Publish's rules into the model's context before it picks a tool:

| Resource | Purpose |
| --- | --- |
| `just-publish-overview` | What Just Publish is and which tool to use when. |
| `just-publish-edit-lifecycle` | The safe read → edit → update loop for changing an existing site. |
| `just-publish-file-conventions` | Path rules, text vs. base64 encoding, size limits, routing. |
| `just-publish-invariants` | Hard rules: `index.html` at root required; `deploy` replaces vs. `update_site_file` merges; the `edit_token` is the only auth. |

The server also exposes a UI resource for the publish-result widget in clients
that render MCP resources.

### Authentication

The `edit_token` returned at first publish is the only credential — there is no
login flow. Save it to update the site later. Email sign-in on the website is
used only for account recovery and gated actions (e.g. connecting a custom
domain).

## About this repository

This repository is the **public metadata + registry listing** for the Just
Publish MCP server: the published `server.json`, the `.mcp.json` plugin
manifest, brand icons, and this README. The running server is a hosted remote
endpoint at `https://mcp.justpublish.ai/`; connect to it over Streamable HTTP as
shown above. The server's own source code is not part of this repository.

The tables above describe the live server's actual tools and resources so that
directory crawlers and readers see an accurate picture of what connecting
delivers.

## License

MIT — see [LICENSE](./LICENSE). Covers this metadata repository (README,
`server.json`, `.mcp.json`, icons). The hosted service itself is governed by the
[Just Publish terms](https://justpublish.ai).

## Links

- Product: https://justpublish.ai
- MCP Registry: https://registry.modelcontextprotocol.io
