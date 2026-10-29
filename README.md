# PartsPlan for Claude Code and Cursor

Datasheet evidence for firmware, PCB design and circuit review.

PartsPlan connects your agent to an archive of manufacturer datasheets for electronic components and takes it to the page that answers the question: the exact document, page, table or figure, read from the original.

## What's in this plugin

- `.mcp.json` (Claude Code) and `mcp.json` (Cursor) — the PartsPlan MCP server `https://worker.partsplanai.com/mcp` (remote; you sign in with your PartsPlan account, no API key in the config)
- `skills/partsplan-datasheets/SKILL.md` — how the agent uses the tools and cites pages
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` — the Claude Code plugin manifest and marketplace entry
- `.cursor-plugin/plugin.json` — the Cursor plugin manifest

Nothing runs on your machine: the plugin has no local code, scripts or hooks.

This repository holds only the plugin configuration and the skill. The PartsPlan service runs at [partsplanai.com](https://partsplanai.com/en) under its own [terms](https://partsplanai.com/en/terms).

## Install

- **Claude Code** (2.1.275 or later): `/plugin install partsplan --marketplace bboinzg-dev/partsplan-plugin`
  Older versions: `/plugin marketplace add bboinzg-dev/partsplan-plugin`, then `/plugin install partsplan@partsplan`.
- **Claude (web, desktop, mobile):** add PartsPlan from the connector directory — [claude.ai/directory/connectors/partsplan](https://claude.ai/directory/connectors/partsplan)
- **Cursor Marketplace:** search for "PartsPlan".
- **Manual (Cursor):** add the server to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "partsplan": { "url": "https://worker.partsplanai.com/mcp" }
  }
}
```

The first tool call opens the PartsPlan sign-in page (Google or email). A free plan is available — see [pricing](https://partsplanai.com/en/pricing). Setup for other apps: [partsplanai.com/en/developers](https://partsplanai.com/en/developers).

## Try

- "Using PartsPlan, find the Config register table in the ADS1115 datasheet and write the bit fields as C defines, citing the page."
- "Using PartsPlan, where does the TPS7A4701 datasheet give the dropout voltage at 1 A? Show the table row and page."
- "Using PartsPlan, check this BOM against the manufacturer datasheets: TPS7A4701RGWR, TPS62130RGTR, ADS1148IPWR (Texas Instruments)."

## Tools

| Tool | What it does |
|---|---|
| `check_bom` | Up to 100 BOM or schematic lines in one call: whether that manufacturer's datasheet is in the archive and whether its pages can be read |
| `find_datasheet` | The manufacturer's document for a part number |
| `get_datasheet_map` | Where sections, tables and figures are in a document |
| `search_datasheet` | Pages, excerpts or every matching line for given terms |
| `get_page_blocks` | Page text with source labels |
| `get_page_image` | The original page or a crop of it |
| `get_figure_data` | Graph values with the figure's page, only when they can be read with certainty |
| `reference_circuits` | Application-circuit pages in the manufacturer's datasheet |
| `search_parts` | Candidate parts by description, device type, values or maker |
| `compare_points` | An adjustable comparison rubric with the datasheet pages to read — no scores or verdicts |
| `request_datasheet_collection` | Records a request to collect a missing datasheet |
| `report_problem` | Records a report when an answer did not match the original |

## Data handling

- **Sent to PartsPlan:** only what the agent puts into a tool call — part numbers, manufacturer names, BOM lines, search terms, document and page identifiers — together with your PartsPlan sign-in.
- **Not sent by the plugin:** your project files. If the agent reads a BOM file in your repository, it sends the lines it extracted, not the file.
- **Kept by PartsPlan:** request records (tool, time, result) and how they are processed, as described in the [privacy policy](https://partsplanai.com/en/privacy).

## Limits

- Answers point to the document and page they were read from; missing information is not estimated.
- An empty result covers the PartsPlan archive only — the manufacturer's site may still have the document.
- No stock, price or lead-time data, and no circuit validation.

## Links

[Website](https://partsplanai.com/en) · [Privacy](https://partsplanai.com/en/privacy) · [Terms](https://partsplanai.com/en/terms) · Support: hello@partsplanai.com

## License

The files in this repository are released under the [MIT License](LICENSE).
