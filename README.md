# Just Publish — MCP Server

**Publish the website you built with AI to a live public URL — straight from chat, no setup.**

Describe a site to your AI assistant — ChatGPT, Claude, Gemini, or any MCP-enabled
client — and it publishes the site and hands back a live URL. No git, no CLI, no
build step, no dashboard, and no login to fumble through. You hand it your files,
it hands back a working link. Built for the person who made a site with AI and
just wants it online.

- **Website:** https://justpublish.ai
- **MCP endpoint:** `https://mcp.justpublish.ai/`
- **Registry name:** `ai.justpublish/just-publish`
- **Transport:** Streamable HTTP (remote — nothing to install)

## How it works

Call `deploy` with your files (HTML, CSS, JS, images) and an email. You get back a
live `url`, a `site_id`, and an `edit_token`. Keep the token and you can update the
same site anytime — either a full redeploy or a single-file edit. That's the whole
flow.

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

## Agent Skill (ChatGPT / Codex / any SKILL.md runtime)

This repo also hosts the **`just-publish` Agent Skill** — a [SKILL.md-format](https://learn.chatgpt.com/docs/build-skills)
package that teaches an agent the full publish flow (including saving the
`edit_token` and the email-verification step) before it ever calls a tool.

Install into Codex / ChatGPT with the skill installer, pointing at this tree:

```
$skill-installer install https://github.com/just-done/just-publish-mcp/tree/main/skills/just-publish
```

Or copy `skills/just-publish/` into your agent's skills directory
(`~/.agents/skills/`, `~/.claude/skills/`, or your runtime's equivalent).

The same package is served for discovery scanners and other runtimes at
[justpublish.ai/.well-known/agent-skills/index.json](https://justpublish.ai/.well-known/agent-skills/index.json)
(Agent Skills Discovery RFC, sha256-digested). The canonical source lives in the
Just Publish server codebase; this tree is its published mirror and is kept in
lockstep — the two are compared on every server deploy.

## Who it's for

Non-technical builders and AI agents that generate a static site and need it hosted
in one step. If you can describe a page, you can publish it.

## Good to know

- **No accounts required.** Edit access is held by the `edit_token` returned at
  publish time — save it to update the site later.
- **Static sites only** — files in, URL out. No frameworks or build pipelines.
- **Custom domains** — connect your own domain at [justpublish.ai](https://justpublish.ai).

## About this repository

Public metadata + registry listing for the Just Publish MCP server: `server.json`,
`.mcp.json`, icons, and this README. The server runs at
`https://mcp.justpublish.ai/`; its source is not part of this repo.

## License

MIT — see [LICENSE](./LICENSE).
