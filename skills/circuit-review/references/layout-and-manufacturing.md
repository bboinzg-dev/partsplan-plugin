# Layout and manufacturing

Layout findings are strongest when they tie the copper to a manufacturer's layout section (`get_datasheet_map(doc_sha256, kind="layout")`, or `search_datasheet` with "layout", "placement", "routing") or to current, return path and assembly risk. Say which basis each finding has: manufacturer requirement, fabrication rule, or general practice. Right angles, via counts and autorouter traces on their own are optional improvements, not defects.

## General physics to apply
- **Placement decides most of it.** Distances fixed at placement cannot be recovered by routing. Check grouping by function and current path (input, power stage, logic, analog), connectors at the edge, and high-current or switching areas away from sensitive inputs.
- **Ground is copper with resistance and inductance.** At high frequency a signal's return current flows in the reference conductor directly beneath the trace. A slot or split under a fast signal forces the return around it, enlarging the loop (more emission and coupling).
- **Splits: ask why first.** If the gap is an isolation barrier or a deliberate domain boundary, keep it; look at how the signal and its return are meant to cross (the isolator itself, or the path the design intends). If the gap is accidental, routing around it or closing it may be the fix.
- **Return paths at layer changes.** Identify each fast signal's reference conductor and how its return current transfers at every layer change. Evaluate the existing coupling and any proposed return component against signal, power-domain and isolation requirements; do not prescribe a capacitor between different nets or across an isolation barrier only because the references differ.
- **Two-layer boards.** Keeping one side as a mostly unbroken ground area with short traces preserves return paths; long traces on that side cut the plane into islands.
- **Current capacity.** Track width, copper weight and via count sized for the current with a standard current-capacity method; vias grouped near the pins that carry the current. A track that narrows to enter a fine-pitch pin can be legitimate.
- **Measure before you report.** Before reporting a geometry violation, check the actual copper polygons, pad rotation and shape, and the measurement method: overlapping bounding boxes are not necessarily overlapping copper, and endpoint distance is not routed length.

## Datasheet-driven items
- **Switching converters.** Identify the high-di/dt loops of the actual topology from the manufacturer's layout guidance (for a buck, the input capacitor and switches; for a boost, the switch, rectifier and output capacitor). Assess loop area, switch-node size and coupling, the return geometry beneath, and feedback routing away from the switch node. Treat layer changes or vias as defects only when they violate an applicable requirement or create a supported electrical or manufacturing risk.
- **Sense and branch points.** Being on the same net is not a Kelvin connection. Check where output-sense, current-sense, current-limit and set-resistor lines actually branch off and return (for example at the output capacitor or a dedicated ground pin), as the pin descriptions and layout section specify.
- **Bypass capacitors.** Evaluate the whole supply-pin → capacitor → return loop, including planes and vias, rather than via count alone. Check the capacitor's value and type against every applicable section; if the pin table and the layout section disagree, report the conflict instead of silently preferring one.
- **Exposed pads and ground pins.** Thermal and ground vias as the datasheet shows; a ground pin that the layout section says needs its own vias should not depend on a neighbouring part's vias.
- **Precision analog.** Check sense-loop area, coupling, imbalance and parasitics against the required bandwidth and accuracy and the device's layout guidance (some converters advise against vias beneath them). Same-layer routing, length matching and a solid ground reference are means whose necessity depends on those requirements.
- **Isolation barriers.** Creepage and clearance per the isolator datasheet and the applicable standard; no copper bridging the barrier except the intended components; a defined common-mode return path when the design relies on one.

## Manufacturing files
- **DRC and ERC results.** Tie each result to the checked files, tool version, active rules and exclusions. Confirm the project's rules and severities are the intended fabricator rules before trusting "0 violations": scripted edits and saves can reset project rules to defaults and silence checks. Separate violations, documented waivers and checks that were not run.
- **Schematic–PCB parity.** Compare actual connectivity, pin-to-pad mapping and assigned footprints between schematic and PCB. Footprint-filter metadata alone does not establish an electrical mismatch; review ERC violations and justified exclusions separately.
- **Default options.** Verify that each assembly variant implements its stated default through fabricated connections, populated parts, solder bridges or explicit jumper assembly instructions, and that the shipped default is distinguishable from an option the user must select.
- **DNP vs. required.** Positions marked DNP that a function depends on (pull resistors on enable pins, for example).
- **Order codes.** Complete manufacturer codes, including termination and packaging, consistent across BOM and documentation.
- **Release consistency.** Schematic, PCB, BOM, assembly and fabrication outputs belong to the same intended release, through an explicit revision or manifest mapping (labels need not be identical); the reviewed snapshot is the one being fabricated.
- **Silkscreen.** Reference, pin-1 and polarity marks are correct for each part type, unambiguous, manufacturable and visible where needed after assembly (not hidden under connector housings or a mating plug). Apply actual body, pad and fabrication clearances rather than treating every courtyard overlap as a defect.
- **3D and mechanics.** Component heights, connector orientation, keep-outs under springs, shields or mounting hardware.
- **Documentation matches the board.** Claims such as "no split crossings" or "two vias per ground pin" are checked against the copper.

## Schematic readability
Report readability as an optional improvement unless it hides an error. The test is whether a reader can trace connections, power domains and current paths without misunderstanding: islands of symbols joined only by global labels hide filter topology and force/sense separation; ground symbols should name their domain; text should not overlap wires or other text; exports should be legible at the resolution they will be viewed.
