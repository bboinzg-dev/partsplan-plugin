# Reading traps and false findings

A review fails in two ways: it misses a real defect, or it reports one that is not there. These are the reading mistakes that cause both. Check them before a finding goes into the report.

## Identity
- Fix the manufacturer, full ordering code, grade, package, document edition and board revision before quoting a limit. Limits for a different grade or variant (for example an extended-temperature grade with a different capacitor range) make a correct design look wrong.
- A part number printed in a document does not make it that document's subject: mating connectors, sockets, evaluation boards and distributor appendices mention parts they do not specify.
- The same part number from two manufacturers is two parts. Do not quote one manufacturer's limit for another's part, and do not "round" a printed limit.

## Tables
- Determine each footnote's scope from its markers, merged cells, headers and wording; do not extend a row-specific qualification to unrelated ratings, or narrow a table-wide one to a single row.
- Keep absolute maximum, recommended operating and electrical characteristics apart, and keep MIN, TYP and MAX apart. A soft-start time printed as MIN is not the nominal value.
- In family datasheets, a column can belong to a different variant.
- Another row of the same table often holds the deciding condition (start-up voltage next to UVLO, the row whose conditions match the actual operating point).

## Extracted text and numbers
- Confirm decisive table values and ambiguous symbols on the page image, including the applicable row, column, units and footnotes. Text layers can map symbols wrongly (µ read as m, Ω as W, θ as q) and lose superscripts, minus signs or decimal points, even when the number looks plausible.
- Printed page numbers and PDF page numbers differ; cite the PDF page the tool reports and say which kind of page it is.
- A correct number with a wrong citation (another chip's figure, another edition's page) still misleads the next reader; check both. A different copy of the same document can have different page numbers — do not call a user's citation wrong on that basis alone.

## Graphs
- Identify the curve from the legend, not its position: temperature curves can cross, so "the right-hand curve" is not always the hottest. Check axis scales (log vs. linear) and the conditions in the caption.
- Use a figure value from `get_figure_data` only after matching the curve, axes, units and conditions to the page image and confirming that the requested point returned a value. A successful tool status does not establish curve identity or a guaranteed limit; if the curve or axes stay ambiguous, do not report a number.

## Search results and continuations
- No search hit does not mean the datasheet does not say it: try the pin or register name with `mode="lines"`, synonyms and abbreviations, the adjacent pages and the continuation of a table.
- Follow the tool's continuation fields until the needed passage, headers and footnotes have been read; for `get_page_blocks`, pass `next_cursor` unchanged as `cursor`, keeping the same `doc_sha256`, `page`, `include_duplicate_blocks` and `block_id` arguments. Do not treat a partial search or page response as complete.

## Calculations, models and tool runs
- Keep calculated values, values chosen in an application example, typical curves and guaranteed limits apart.
- Model results (DC-bias curves, simulations) hold only for that model's conditions — temperature, frequency, AC amplitude and range.
- In simulations, check that the netlist matches the user's circuit, loads and values: an ideal voltage source on the output forces the result, default-filled values are not the design's values, and an omitted or wrong load prevents conclusions about behaviour under the claimed load. A simulation that runs is not a circuit that works.
- Check that exported values and plots correspond to the raw output's sweep points, frequency, columns and units. A failed tool or solver run does not establish a circuit failure; report only the cause its diagnostics support.

## Strength of the claim
- A recommendation is not a requirement; a value or arrangement that differs from the application example can be valid. Name the requirement the design violates.
- Different package names can share the same land pattern; compare dimensions, not names.
- An erratum or known issue from another silicon stepping, board revision or sheet is a false finding unless the design under review shows it.
- A pull resistor on a strap pin cannot be called wrong without the strap's sampling conditions and any configuration that overrides it.
- One test point (a clamp's voltage at one current and temperature) does not prove protection over the whole range.
- Strong words — "not in the datasheet", "protects", "oscillates", "safe" — need evidence of the same strength. When evidence is missing or conflicting, report "not established" or "conditional" and name the data that would decide it.
- A proposed fix is itself a design change: check it against the same datasheets before recommending it, and say what else it affects.
