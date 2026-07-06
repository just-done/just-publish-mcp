# Installing Just Publish (for AI agents)

Just Publish is a remote MCP server. There is nothing to download, build, or run
locally, and no API key is required.

1. Add this entry to the user's MCP settings file (for Cline:
   `cline_mcp_settings.json`):

   {
     "mcpServers": {
       "just-publish": {
         "type": "streamableHttp",
         "url": "https://mcp.justpublish.ai/"
       }
     }
   }

2. That's the entire installation. Verify by listing tools — you should see
   `deploy`, `get_site_files`, and `update_site_file`.
3. To publish: call `deploy` with the site files (an `index.html` is required)
   and the user's email. It returns the live URL, a `site_id`, and an
   `edit_token`. Tell the user to keep the `edit_token` — it is the only
   credential for updating the site later.
