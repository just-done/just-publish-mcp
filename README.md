# Just Publish MCP Server

**Publish and update the website you built with AI at a live public URL. Sign-in required.**

Just Publish puts the website you built with AI live at a public address. Your
assistant makes the files; Just Publish publishes them at a `justpublish.site`
address that anyone can open, and keeps them there. You sign in once with your
Just account, and every site belongs to the account that published it. A
published site can later run at a domain you own.

- **Website:** https://justpublish.ai
- **MCP endpoint:** `https://mcp.justpublish.ai/`
- **Registry name:** `ai.justpublish/just-publish`
- **Transport:** Streamable HTTP (remote, nothing to install)
- **Authorization:** OAuth 2.1 with dynamic client registration. The client
  opens a sign-in window the first time (Google or a code sent by email).

## Connecting

Add the endpoint to your client's MCP servers and sign in when it asks:

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

The client discovers the authorization server from the endpoint, so there is
no API key, client id or secret to paste. In Claude, add the connector at
`https://mcp.justpublish.ai/claude`; in Claude Code:

```bash
claude mcp add --transport http just-publish https://mcp.justpublish.ai/claude
```

Step-by-step guides for each assistant and editor are at
https://justpublish.ai/install.

## Tools

| Tool | What it does |
| --- | --- |
| `deploy` | Publishes a site from its files and returns its public address. |
| `get_site_files` | Reads the files of a published site. |
| `update_site_file` | Changes one file of a published site and puts the change live. |
| `list_sites` | Lists the sites on the signed-in account. |

The account comes from the sign-in, so `deploy` takes no email address and the
signed-in person's own sites need only a `site_id`.

## Resources

Static Markdown guidance the client can load before it picks a tool:

| Resource | Purpose |
| --- | --- |
| `just-publish-overview` | What Just Publish is and which tool to use when. |
| `just-publish-edit-lifecycle` | The read, edit and update loop. |
| `just-publish-file-conventions` | Paths, encoding, size limits, routing. |
| `just-publish-invariants` | `index.html` required; `deploy` replaces, `update_site_file` merges. |

## Agent skill

This repository also hosts the `just-publish` skill in the
[SKILL.md format](https://learn.chatgpt.com/docs/build-skills), which teaches an
agent the publish flow before it calls a tool. Add it, with the Just Domain
skill, to any agent that reads Agent Skills:

```bash
npx skills add just-done/skills
```

To install this copy in Codex, give the skill installer this tree:

```
https://github.com/just-done/just-publish-mcp/tree/main/skills/just-publish
```

or copy `skills/just-publish/` into `~/.codex/skills/` (or `~/.claude/skills/`).
The same package is listed for discovery at
[justpublish.ai/.well-known/agent-skills/index.json](https://justpublish.ai/.well-known/agent-skills/index.json).

## Good to know

- **Static sites only.** HTML, CSS, JavaScript, images and fonts, with
  `index.html` at the root. No build step and no server code.
- **Your own domain.** Connect a domain you own from the dashboard at
  [justpublish.ai](https://justpublish.ai).

## About this repository

Public metadata and the registry listing for the Just Publish MCP server:
`server.json`, `.mcp.json`, icons, the skill and this README. The server runs
at `https://mcp.justpublish.ai/`; its source is not part of this repository.
Just Publish is operated by Just Done LLC.

## License

MIT. See [LICENSE](./LICENSE).
