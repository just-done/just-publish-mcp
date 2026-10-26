# Installing Just Publish (for AI agents)

Just Publish is a remote MCP server with OAuth sign-in. There is nothing to
download, build or run locally, and no API key to paste.

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

2. Connect. The server answers with an OAuth challenge and the client opens a
   sign-in window; the user signs in with their Just account (Google or a code
   sent by email). The client registers itself, so leave any client id and
   secret fields empty.
3. Verify by listing tools. You should see four: `deploy`, `get_site_files`,
   `update_site_file` and `list_sites`.
4. To publish, call `deploy` with the site files (`index.html` at the root is
   required). It returns the live URL and a `site_id`. The site belongs to the
   signed-in account, so do not ask the user for an email address.
5. To change a site later, call `list_sites` to find it, `get_site_files` to
   read it, then `update_site_file` with the changed files.
