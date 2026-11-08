# Design checks

Each group lists what to look for in the user's design, where the deciding text usually is in the datasheet (`get_datasheet_map` kind, or the functional description when no kind fits), and the failure it prevents. Use the groups the inputs allow; skip what the inputs cannot support and say so.

## 1. Identity and variants
- **Exact ordering code.** Compare the full BOM code with the `ordering` or device-comparison table: operating mode (PFM/auto vs. forced PWM), fixed vs. adjustable output, enable polarity, grade, temperature range, pinout options. *Failure:* a light-load-mode regulator used where the output must sink current rises above its setpoint; a different pinout option shorts a rail.
- **Errata and silicon stepping.** Search the family datasheet and errata sheet. Apply an erratum only when the stepping and the conditions match (for example, an input-leakage erratum that requires pull-downs below a stated value). *Failure:* a known silicon issue reproduced, or a false finding from an erratum that does not apply.
- **Second sources.** The same part number from another manufacturer can have different limits, pinout details or internal structure. Use the document of the manufacturer on the BOM.

## 2. Pins and footprints
- **Pin map.** For each IC, compare the symbol pin numbers and names, the netlist and the footprint pad numbers with `pinmap`. The symbol's pin numbers and the footprint's pad numbers are defined separately and can disagree. Check unused pins (left open, tied, or required to be connected) and pins that must not float.
- **Same package, different pinout.** SOT-23, SOT-223, SOIC and QFN parts often swap functions between manufacturers and variants (pin 1 = GND on one regulator and IN on another; a tab tied to OUT on one and to GND on another). Never infer a pinout from the package name; quote the datasheet's pin table or top view.
- **Exposed or thermal pad.** What it must connect to (ground, a specific pin, or left floating), plus the via and paste guidance.
- **Hand-made footprints.** RF connectors, modules and unusual connectors: compare the contact assignment, the view direction (top or bottom view), mirroring and keep-outs with the manufacturer drawing (`package`, `land_pattern`). *Failure:* a signal contact landing on a ground pad.
- **Land pattern.** Compare pad size and gap with the manufacturer's recommended land pattern when one is printed; a generic library footprint can be larger or smaller.
- **Polarity and pin 1.** Diodes, electrolytics, LEDs and multi-pin diode packages: a blanket "dot = cathode" legend is wrong for parts whose pin 1 is an anode.

## 3. Ratings and margins
- **Absolute maximum vs. recommended operating.** Absolute-maximum ratings are stress limits whose exceedance may cause damage; staying within them does not establish correct operation or reliability. Check every supply, input and output pin against the highest rail including tolerance, start-up overshoot, load-dump or inductive transients, and negative excursions.
- **Guaranteed vs. typical.** Check the datasheet's guarantee notes, grade and conditions before using a MIN or MAX value as a design bound. TYP values can support an explicitly labelled estimate, never a guaranteed margin or a fault window.
- **Conditions.** Each limit applies at its printed conditions (temperature, supply, frequency, load). A performance figure stated at an exact clock frequency or a specific load does not carry over to other operating points.
- **Device minimum supply.** Besides the voltage a calculation needs, check the device's own recommended minimum supply.
- **Derating and safe operating area.** Read graphs on the page image. Use a value from `get_figure_data` only after matching the curve, axes, units and conditions to the page image and checking that the requested point actually returned a value.

## 4. Stability and power components
- **Regulator output capacitor.** Minimum capacitance and, for some LDOs, an ESR window: a ceramic capacitor can be below the minimum ESR and oscillate. Check both the capacitance range and the ESR range in `rec_op` or the application section.
- **Effective capacitance.** For voltage-dependent dielectrics such as Class II MLCCs, evaluate the exact part's effective capacitance with DC bias, tolerance, temperature and aging; do not infer the loss from package size alone, and do not treat typical model results as a guaranteed minimum.
- **Inductors.** Peak current against the manufacturer's saturation definition and its temperature conditions; RMS current against the permitted temperature rise; inductance tolerance, current-dependent inductance and winding resistance at the operating temperature; the inductance range the regulator allows.
- **Transformer-based supplies.** Winding-ratio definitions, centre taps, polarity, rectifier losses and rail limits over input and load tolerance. Check the source's loading and the start-up peak demand across the whole power path; average capacitor-charging current alone does not bound the peak.
- **Compensation and feedback.** Divider values against the reference and the input bias current; feed-forward or compensation parts as the application section specifies.

## 5. Protection coordination
- **Normal operation.** A TVS's reverse stand-off voltage should be at or above the highest normal rail voltage; check where its leakage is specified and what that leakage does at the operating voltage.
- **Fault voltage at every protected pin.** For the stated fault waveform and source impedance, bound the clamp current and the voltage at each protected pin, including overshoot from intervening impedance. Compare it with the absolute maximum of every device that sees that node: supply pins, inputs, and MOSFET gate-source limits when the gate is referenced elsewhere. Follow rails joined through ferrite beads or zero-ohm links; protecting one supply pin does not protect the others. A clamp voltage specified at the rated pulse current neither proves protection nor proves a violation under an unspecified surge. When the fault specification is missing, report protection as not established; report a rated-current comparison as a conditional risk only when it shows a possible limit violation, stating the pulse conditions and that their applicability to this design is unconfirmed.
- **Where the fault current goes.** Trace terminal → clamp → rail. Determine whether the injected current exceeds what the rail's load and sink paths can absorb, and whether the resulting rail voltage exceeds a limit. A series resistor limits fault current only when it lies in the actual current path.
- **Negative excursions.** Some regulators and ICs allow a pin only a few hundred millivolts below ground. Compare the actual negative excursion and the clamp's forward voltage at the relevant current, temperature and pulse duration with that limit; a TVS's forward drop often exceeds it, and no clamp type is adequate by label alone.
- **Leakage and precision.** Clamp leakage at the operating voltage (often specified only at a lower voltage) adds error in high-impedance or sensing circuits.
- **Scope.** Report what the protection covers and what it does not; do not add threat levels the design never claimed, but ask.

## 6. Interfaces, logic levels and timing
- **Thresholds.** VOH/VOL of the driver against VIH/VIL of the receiver over the supply range, and the actual node voltage when resistor networks, LEDs or pull-ups share the line. *Failure:* a node sitting between VIL max and VIH min reads undefined.
- **Buffers and repeaters.** Read the connection rules: some bus repeaters' buffered sides must not be connected to each other; some translators need a specific side on the lower voltage.
- **Output current.** Outputs driving clamped inputs or LEDs stay within the recommended output current, including states where the receiving side is unpowered.
- **Timing.** Use the datasheet's own definitions of clock frequency, data rate and pulse width when choosing timing and supply-current rows; edges per clock do not by themselves establish a bit rate. Include duty-cycle range, propagation delays and load capacitance, and calculate reset and start-up waits at the relevant oscillator limit. Make sure a timing parameter belongs to the pin it is applied to.
- **Frames and status.** CRC or checksum covering every enabled field; status bits read where the datasheet says they are valid.
- **Supplied firmware sequences.** When the hardware depends on a supplied sequence, check reset, start and calibration order, worst-case completion times, data-ready re-arming and frame validity, and that gain, reference and excitation settings are restored after diagnostics or calibration; also the handling of timeouts, CRC errors and resets. Missing firmware evidence is a limitation, not a schematic defect.

## 7. Analog and precision
- **Input bias and common mode.** Evaluate how external resistors load an input's internal bias network and change common mode, signal swing and settling; report a defect when the resulting operating point violates a requirement. Check common-mode range, output swing and the input network the datasheet shows.
- **Excitation and reference.** Current-source compliance, reference buffer settings and the supply minimum together.
- **Kelvin sensing.** Sense lines from the pads the resistor or reference datasheet specifies, routed together.
- **Error budgets.** Bound guaranteed error conservatively; use RSS only with justified distributions and correlations, and state the resulting confidence. Include calibration residuals, drift, self-heating and uncorrected terms; do not treat a typical INL as a guaranteed residual.
- **Filters and settling.** Evaluate the whole RC network including the sensor, cables and multiplexer, with the real sampling sequence.

## 8. Thermal
- Estimate junction temperature with a method that fits the actual board and boundary conditions, at the real operating point (worst input, load and ambient), and compare it with the maximum junction temperature. Do not interchange θJA and ψJT: ψJT relates junction temperature to the package-top temperature under its stated conditions and is not a junction-to-ambient resistance. Check the copper area and thermal vias the datasheet's figures assume.

## 9. Supporting parts
- **Crystals.** Load capacitance of the crystal against the oscillator requirements, the load capacitors and stray capacitance; drive level.
- **MOSFETs.** RDS(on) at the gate voltage actually applied; peak and RMS current; gate charge vs. the driver; body-diode direction in power paths.
- **Relays.** Coil voltage range including tolerance, contact rating for the load type (AC/DC, inductive), and the exact ordering code's coil and contact options.
- **Enable, reset and supervisors.** Thresholds, hysteresis and timing; the state of the pin while the controlling part is unpowered or in reset.
