# Just Publish — MCP Server

**Publish a static website to a live public URL, straight from chat.**

Describe a site to your AI assistant — ChatGPT, Claude, Gemini, or any
MCP-enabled client — and it publishes the site and hands back a live URL. No git,
no CLI, no build.

- **Website:** https://justpublish.ai
- **MCP endpoint:** `https://mcp.justpublish.ai/`
- **Registry name:** `ai.justpublish/just-publish`
- **Transport:** Streamable HTTP (remote — nothing to install)

## Connecting

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

Then ask your assistant to publish a site. It returns the public URL.

## Tools

| Tool | What it does |
| --- | --- |
| `deploy` | Publish a new static site (HTML/CSS/JS/images), or fully replace an existing one. Returns the live URL, a `site_id`, and an `edit_token`. |
| `get_site_files` | Read a published site's files so the assistant can make a targeted edit. Requires `site_id` + `edit_token`. |
| `update_site_file` | Change one or a few files in place (merge). Requires `site_id` + `edit_token`. |

## Resources

Static markdown guidance the client can load into context before it picks a tool:

| Resource | Purpose |
| --- | --- |
| `just-publish-overview` | What Just Publish is and which tool to use when. |
| `just-publish-edit-lifecycle` | The read → edit → update loop. |
| `just-publish-file-conventions` | Paths, encoding, size limits, routing. |
| `just-publish-invariants` | `index.html` required; `deploy` replaces vs. `update_site_file` merges; `edit_token` is the only auth. |

Plus a UI resource for the publish-result widget.

## About this repository

Public metadata + registry listing for the Just Publish MCP server: `server.json`,
`.mcp.json`, icons, and this README. The server runs at
`https://mcp.justpublish.ai/`; its source is not part of this repo.

## License

MIT — see [LICENSE](./LICENSE).
