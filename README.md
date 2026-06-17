# Just Publish — MCP Server

**Publish a static website to a live public URL, straight from chat.**

[Just Publish](https://justpublish.ai) is an AI-native publishing tool. Hand it
your HTML, CSS, JS, and images from inside an MCP-enabled client and it returns a
live, public URL — no build step, no dashboard, no deploy config.

- **Website:** https://justpublish.ai
- **MCP endpoint:** `https://mcp.justpublish.ai/` (Streamable HTTP)
- **Registry name:** `ai.justpublish/just-publish`

## Connecting

Add the remote MCP server to any client that supports Streamable HTTP transport:

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

Then ask your assistant to publish a site. It will return the public URL.

## About this repository

This is a **metadata-only** repository. It holds the published `server.json`,
the `.mcp.json` plugin manifest, brand icons, and this README for the MCP
Registry listing. The Just Publish server itself is hosted at
`https://mcp.justpublish.ai/` — its source is not part of this repo.

The root `.mcp.json` declares the remote server in the Open Plugins format so
plugin directories can detect it.

## Links

- Product: https://justpublish.ai
- MCP Registry: https://registry.modelcontextprotocol.io
