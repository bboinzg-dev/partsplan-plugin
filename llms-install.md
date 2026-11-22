# Installing the PartsPlan MCP server (for AI agents such as Cline)

PartsPlan is a hosted (remote) MCP server. There is nothing to clone, build or run locally, and no API key goes into the configuration.

1. Add this entry to the MCP settings file. In Cline it is `~/.cline/data/settings/cline_mcp_settings.json`, used by both the VS Code extension and the Cline CLI (Customize → MCP → Edit Configuration; older versions: MCP Servers → Configure → Configure MCP Servers):

```json
{
  "mcpServers": {
    "partsplan": {
      "type": "streamableHttp",
      "url": "https://worker.partsplanai.com/mcp",
      "timeout": 300,
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

   Keep `"type": "streamableHttp"`. Without it Cline uses the legacy SSE transport. `timeout` (seconds) is Cline's MCP request time limit (60 by default).

   Edit the file in place: Cline reloads it when the file changes. If `partsplan` does not appear under Customize → MCP → Installed (for example after the file was replaced rather than edited), open **Edit Configuration** and save once.

2. PartsPlan signs users in with OAuth. After saving the settings, ask the user to click **Authenticate** next to `partsplan` (Customize → MCP → Installed; older versions: the MCP Servers panel) and sign in with their PartsPlan account (Google or email) in the browser window that opens. If the sign-in takes longer than a minute, Cline may stop with "timed out after 60s"; click **Authenticate** again. A free plan is available.

3. When connected, `partsplan` shows a green dot and lists its tools. Check the connection by calling `find_datasheet` with `{"mpn": "AO3400A"}`. The result lists the manufacturer's datasheet; `get_datasheet_map`, `get_page_blocks` and `get_page_image` then open the pages behind an answer.
