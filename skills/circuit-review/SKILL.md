---
name: circuit-review
description: Review a finished schematic, BOM or PCB layout before fabrication and point out problems with manufacturer-datasheet evidence (document and page). Covers exact part variants and errata, pinouts and footprints, every power state (start-up, shutdown, unpowered domains, back-powering, reset defaults), protection coordination, stability, interfaces and timing, layout against the datasheet's layout guidance, and manufacturing files. Use when someone asks to check, review or sanity-check a circuit or board ("review my schematic", "what did I miss", "before I order PCBs"). It reports findings; it does not validate or redesign the circuit.
---

# Design review: point out problems, with the page that shows them

Many engineers finish a design and then ask an AI to review it before ordering boards. In this review the job is to find what is wrong or unproven in someone else's design — not to redesign it and not to approve it. Every finding names the place in the design, the condition under which it fails, and the evidence that sets the limit. A review with no findings is a record of what was checked, never a statement that the board will work.

## 1. Inputs and scope
- Start with what the user provided. Ask only for information that would change a finding: rails with tolerances and transients, load currents, operating temperature, layer stack-up, the fabricator's minimum track/gap/drill, the silicon revision when an erratum depends on it.
- A BOM supports identity and documented rating and package checks; suitability depends on the operating conditions supplied. A schematic or netlist supports connection and power-state checks. Placement and routing checks need PCB data or sufficiently informative images; leave unseen details unassessed and say so.
- Out of scope unless asked: a full firmware audit, choosing replacement parts, and requirements the design never claimed (raise those as questions, not defects).

## 2. Inventory the BOM
- Submit identifiable lines to `check_bom` in batches of up to 100. Each line is `{mpn, brand, ref, qty}`: `mpn` is required, `brand` is the manufacturer as printed, `ref` carries the RefDes. A part known only by its value (for example "10 µF 0603 ceramic") has no part number to submit: keep it out of the call, list it in the ledger as value-only, and ask for the exact part when its rating, dielectric or footprint matters. Never invent a manufacturer or ordering code.
- `found` establishes that a document is available, not that the line was reviewed.
- Keep a review ledger per RefDes or functional group: document availability → evidence read → conditions applied → conclusion or limitation. Every line ends in a conclusion or a stated limitation.

## 3. Choose what to examine by risk, not by "main IC"
Real defects often sit outside the main ICs. Give every part in these groups a review question and applicable evidence, however small:
- the power path and its control: regulators, load switches, power multiplexers, MOSFETs, diodes, fuses, inductors, transformers — their pin maps as well as their ratings
- protection: TVS diodes, ESD arrays, clamps, and resistors meant to limit fault current
- connectors and anything with a hand-made footprint (RF connectors, modules)
- parts and resistors that set modes or defaults: enable and charge-enable pins, strap and boot pins, pull-up/pull-down networks, DNP positions
- parts between voltage or power domains: level shifters, buffers, bus repeaters, isolators
- capacitors that a regulator's stability or a reference depends on

Reuse shared documents and read the sections a question needs; do not request every map kind for every part. Keep reading for any relevant check that is still unanswered, including checks that could reveal a new finding — do not stop because no defect is suspected yet. The number of calls is not a measure of review quality.

## 4. Fix the identity of each part you examine
- Resolve each distinct manufacturer and ordering code with `find_datasheet(mpn, brand)` and reuse the result for repeated instances. When `requested_maker_document` is present, treat it as the candidate and confirm in the document that it covers the exact part. Another manufacturer's document for the same part number describes a different part, and its limits can differ. If identity stays unresolved, report that limitation.
- Read the `ordering` or device-comparison table for the exact code on the BOM. Suffixes change behaviour: operating mode (a light-load PFM variant may be unable to sink current that a forced-PWM variant can), pinout, grade and temperature range.
- Prefer the manufacturer's full document for the exact device and note the edition used. A translation or short summary can omit sections; when the available document cannot establish a needed condition, say so and consult the manufacturer's full datasheet or hardware design guide where possible, labelled as outside PartsPlan.
- Errata sit at the end of family datasheets or in separate errata sheets (search for "errata", "erratum"). Apply only items for the silicon stepping and board revision under review.

## 5. Read sections, not snippets
The condition is usually printed next to the number: another row of the same table (start-up voltage vs. UVLO), a footnote, the MIN/TYP/MAX column, the test conditions, or a paragraph further on.
- Go to the section with `get_datasheet_map(doc_sha256, kind=...)` — kinds include `pinmap`, `package`, `land_pattern`, `ordering`, `abs_max`, `rec_op`, `elec_char`, `timing`, `thermal`, `app_circuit`, `layout`, `figure`.
- Read the page with `get_page_blocks`, following its continuation cursor until the passage, headers and footnotes have been read. Confirm decisive table values, units, signs and footnotes on `get_page_image`.
- A page opened for one question often answers another: the first page usually carries the pin table, the ordering options and the key limits. Record every open check the page settles, not only the one you came for.
- `search_datasheet(doc_sha256, q=[...], mode="lines")` locates mentions of a pin, setting or condition in the stored text; it does not guarantee every occurrence in the PDF. Start with the relevant pin, mode and condition terms, then use `q=["must not","do not","should not","cannot","not recommended"]` in the applicable sections to catch remaining constraints.

## 6. The checks
Work through the groups the inputs allow. The main text only names each check; the failure patterns behind them are in the reference files — open `references/design-checks.md` and, when a schematic is available, `references/power-states.md` before writing the report.
1. **Pins and footprints — for every IC, including regulators, references and other three-terminal parts**: compare each symbol's pin numbers with the exact part's pin table, not with a neighbouring part's. The same package can carry different pinouts: positive and negative linear regulators (78xx-style vs. 79xx-style) put input, ground and output on different pins, and a symbol copied from one to the other is a classic error. Also the exposed-pad connection, and hand-made footprints against the manufacturer drawing (contact assignment, view direction, mirroring). See `references/design-checks.md`.
2. **Every power state**: cold start, normal operation, brown-out, shutdown, charging modes, one supply or domain absent, plug and unplug order, the moments after reset before firmware runs. See `references/power-states.md`.
3. **Ratings and margins**: absolute maximum vs. recommended operating vs. guaranteed characteristics, at every corner of supply tolerance, temperature and transients; logic thresholds against the voltage the node actually reaches.
4. **Stability and power components**: output-capacitor minimum and ESR window, effective capacitance of the exact capacitor, compensation as specified, inductor and transformer limits.
5. **Protection coordination**: stand-off voltage and leakage at the normal rail; the voltage each protected pin sees under the stated or expected fault (clamp current, source impedance, intervening impedance, gate-source references) against every device on that node, including rails joined through ferrite beads; where the clamp current returns; negative excursions against the pins' limits. When the fault specification is missing, report protection as not established; report a rated-current comparison as a conditional risk only when it shows a possible limit violation, stating the pulse conditions and that their applicability to this design is unconfirmed.
6. **Interfaces, timing and supplied firmware sequences**: connection rules of buffers and repeaters, timing at the relevant oscillator and load limits, timing parameters applied to the right pin, and — when the hardware depends on a supplied start-up or calibration sequence — its order and timeouts.
7. **Layout against the datasheet's layout guidance**: high-di/dt loops of the actual topology, switch node, feedback and sense branch points, exposed-pad vias, bypass loops, reference planes and splits, sense loops, current capacity. See `references/layout-and-manufacturing.md`.
8. **Manufacturing files**: how each default option is implemented, DNP vs. required parts, complete order codes, release consistency across files, pin-1 and polarity marks, and DRC/ERC results tied to the files and rules actually used.

## 7. Before reporting a finding, try to refute it
A wrong finding costs the user time and trust. For each candidate, check (more in `references/reading-traps.md`):
- right manufacturer, ordering code, grade and package; the table column, row or footnote really applies to this variant
- typical vs. guaranteed; the printed conditions match the design (temperature, frequency, load, supply)
- the value is actually outside the range — do not call 4.7 µF too small when the stated range starts at 2.2 µF
- the erratum or known issue applies to this silicon stepping and board revision
- the graph curve is the right one (legend, crossing curves, log axes), checked on the page image
- extracted text can carry wrong symbols or lost signs and decimal points (µ read as m, Ω as W): decisive values are confirmed on the page image
- a recommendation is not a requirement, and an implementation that differs from the application example can still be valid
- state the consequence precisely, without inflating it

## 8. Report
Order: **findings** by severity → **conditional risks** (the condition that makes each real) → **not established** (missing data and what it would decide) → **scope** (what was and was not checked) → **optional improvements** (readability, maintainability; kept apart from defects).

Write each finding in this form:

```text
[severity · confidence] <RefDes / pin / net / location> — <the problem in one sentence>
- Observed: <design fact, with file / sheet / location>
- Applies when: <operating or fault condition>
- Evidence: <manufacturer, exact part number, document edition, doc_sha256 + PDF page, table or figure and its printed conditions — or the design file or fabrication rule it rests on>
- Reasoning and consequence: <the violation or calculation, with inputs, units and uncertainty>
- To confirm or refute: <the smallest missing check or data>
```

- **Severity:** critical (damage or core function lost) · major (start-up, protection, communication or accuracy requirement not met) · minor (local assembly or documentation issue) · suggestion (no failure shown).
- **Confidence** is a separate axis: confirmed · conditional · not established. A high severity never raises the confidence.
- **Evidence by type:** a datasheet-based finding needs both the applicable text and the violation in the user's design. A connection, silkscreen or file mismatch can rest on the user's files alone; do not attach a datasheet citation that does not support it. A calculation shows inputs, units and tolerances and keeps guaranteed and estimated values apart. A general layout principle says whether it is a manufacturer requirement, a fabrication rule or a general practice. When sources conflict, show both and hold the finding.
- Never call the design validated, safe or ready. "No findings" in a group means none were found with the evidence available.

## What the answers mean
- PartsPlan supplies the datasheet evidence; the review is a reading of it. Missing evidence means not established by these documents, not that the content does not exist or that the design is wrong.
- Copy doc_sha256 values exactly from tool output. If a document cannot be opened, say so; do not fill the gap from memory or from another manufacturer's document.
- `related_documents` is a response field listing further documents, not an assessment of applicability; check each item's publisher, type and conditions. Items marked as computed (such as MLCC DC-bias curves) are model results, not manufacturer guarantees.
- Label information from outside PartsPlan as such. `request_datasheet_collection` and `report_problem` are write actions used only on the user's request; `report_problem` is for problems with PartsPlan's documents or tools, not for defects in the user's circuit.
