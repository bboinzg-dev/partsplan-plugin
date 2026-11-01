---
name: partsplan-datasheets
description: Look up electronic-component datasheets with PartsPlan while writing firmware, designing a PCB or reviewing a circuit — find the manufacturer's document for a part number, go to the page with the register map, pinout, electrical table, graph or application circuit, and cite the document and page. Use for ratings, pinouts, register maps, package drawings, BOM checks and comparing candidate parts; check live distributor stock and prices only when the user asks for them.
---

# PartsPlan datasheets

PartsPlan keeps an archive of manufacturer datasheets for electronic components and shows where each section, table and figure sits in a document. It returns locations and the printed source; the engineering judgement stays with the user and the conversation.

## Connect
This plugin adds the PartsPlan MCP server (`https://worker.partsplanai.com/mcp`). The first time a tool runs, the app (Claude Code or Cursor) opens the PartsPlan sign-in page (Google or email; a free plan is available). Call limits depend on the account; when a limit is reached the tool says so and when it resets.

## The basic path (one part, one question)
1. `find_datasheet(mpn, brand)` — the requested manufacturer's document comes first; documents from other manufacturers for the same part number are listed separately and describe other parts.
2. `get_datasheet_map(doc_sha256, kind=...)` — jump to a section: `abs_max`, `elec_char`, `rec_op`, `thermal`, `pinmap`, `package`, `land_pattern`, `figure`, `app_circuit`, `ordering` …
3. Read the page:
   - text: `get_page_blocks(doc_sha256, page)`
   - tables: the page image is the reference — read the table region with `get_page_image`
   - graphs: `get_figure_data` (numbers only when they can be read with certainty) or `get_page_image` for the figure region
4. Cite the document and page (`doc_sha256` + page) for every number used.

`search_datasheet(doc_sha256, q=[...], mode="context")` finds terms, register names, conditions and footnotes inside one document; `mode="lines"` returns every matching line in document order.

## In the editor
- Writing a driver: find the register map or command table with `search_datasheet`, read that page before writing bit fields, and put the document and page in a code comment.
- Pin assignments: read the pin description table (`kind=pinmap`) for the exact package on the board.
- A BOM in the repository: pass up to 100 lines per call to `check_bom` to see, for each line, whether that manufacturer's datasheet is in the archive and whether its pages can be read.
- Picking parts: `search_parts` by description, device type, values or maker. `compare_points(source, candidates, device)` returns an adjustable rubric (hard lines, weighted items summing to 100 and checks) with the datasheet pages to read for each item and part — no scores, rankings or compatibility verdicts.
- Reference designs: `reference_circuits(mpn)` gives the application-circuit pages in the manufacturer's datasheet.
- A missing datasheet the user wants: `request_datasheet_collection` records a collection request.
- A wrong answer: `report_problem` with the document, page and block.

## Stock and prices
Use `check_stock` only for an explicit stock or price request. An exact part number is required; ask for it when missing. A request accepts up to three exact part numbers and uses five plan units per part number. Do not add stock checks to datasheet, BOM or driver tasks without a request.

Show each distributor's reported price breaks, stock, lead time and lifecycle labels when supplied, with the retrieval time and currency. Keep the distributor attribution and data notices. Include only returned product links; element14 results have no product link. Report unavailable or failed lookups as such, not as zero stock. Do not convert currencies, compare prices across currencies, estimate missing values, rank sellers or recommend a purchase. Longer orderable codes listed in `candidates` were not checked; check one by calling again with it.

PartsPlan does not store stock-query results. Distributor results have retrieval times rather than datasheet page citations. PartsPlan does not place orders or process payments.

## What the answers mean
- A value always comes with the document and page it was read from; missing values are not estimated.
- "No document" covers the PartsPlan archive only. The manufacturer's site can still answer — say which parts of an answer came from outside PartsPlan.
- When a text block is marked as read from an image of the page, confirm numbers on the page image.
- PartsPlan does not validate circuits.
