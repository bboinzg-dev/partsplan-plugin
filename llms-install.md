# Installing the PartsPlan MCP server (for AI agents such as Cline)

PartsPlan is a hosted (remote) MCP server. There is nothing to clone, build or run locally, and no API key goes into the configuration.

1. Add this entry to the MCP settings file. In Cline it is `cline_mcp_settings.json` (MCP Servers → Configure → Configure MCP Servers; Cline CLI: `~/.cline/data/settings/cline_mcp_settings.json`):

```json
{
  "mcpServers": {
    "partsplan": {
      "type": "streamableHttp",
      "url": "https://worker.partsplanai.com/mcp",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

   Keep `"type": "streamableHttp"`. Without it Cline uses the legacy SSE transport.

2. PartsPlan signs users in with OAuth. After saving the settings, ask the user to click **Authenticate** next to `partsplan` in Cline's MCP Servers panel and sign in with their PartsPlan account (Google or email) in the browser window that opens. A free plan is available.

3. Check the connection by calling `find_datasheet` with `{"mpn": "AO3400A"}`. The result lists the manufacturer's datasheet; `get_datasheet_map`, `get_page_blocks` and `get_page_image` then open the pages behind an answer.
