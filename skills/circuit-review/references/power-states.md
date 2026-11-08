# Power states and sequences

Many field failures happen outside steady operation. Walk the design through each state below and ask what every pin connected across a domain or supply boundary sees. The deciding text is usually in `abs_max` (input voltage relative to supply, injection current), `elec_char` (leakage, internal pull-ups, reset defaults), `pinmap` (pin descriptions) and the functional description.

## States to walk through
| State | Ask |
|---|---|
| Cold start | Does every regulator start at the lowest input? Start-up voltage can differ from the UVLO threshold and the minimum operating voltage. Can the source deliver the start-up demand (soft-start inrush, capacitor charging, peak load) within its rating? |
| Ramp and sequencing | Are rails required in an order? Do enable, power-good and reset follow the datasheet sequence? Is anything driven before its supply is valid? |
| Normal | Covered by the other checks. |
| Brown-out | Do monitors and supervisors operate at the voltage they are meant to detect? Check that a measurement used for a start-up or shutdown decision actually runs in that power state. |
| Charging, when present | Distinguish start, restart, continued charging and boost operation; apply each mode's temperature thresholds, hysteresis and configuration, and check that the control and measurement functions needed for those decisions operate in that state. |
| Shutdown and discharge | Where does stored charge go when power is removed? Can a regulator's output stay above its input (reverse current)? Do input pins see voltage while their supply collapses? |
| One domain absent | A powered device driving an unpowered one, an isolator with one side off, a module without its main supply, a connector whose supply pin is unpowered. |
| Plug and unplug order | With two or more sources, consider every order of connecting and disconnecting them. |
| After reset, before firmware | Default pin states, internal pull-ups and pull-downs, strap sampling, outputs of I/O expanders and GPIOs. |

## Mechanisms to look for
- **Back-powering through inputs.** Establish the actual input and protection structure before assuming a path to an unpowered supply. Check power-off tolerance, Ioff or partial-power-down features, injection-current limits and supply-relative input limits, then decide whether the resulting current can power the device or rail, or disturb its reset.
- **Protection arrays with a supply pin.** Rail-steering ESD arrays connect each I/O through internal diodes to a supply pin. Check whether any signal can feed that pin when its supply is absent, and what the rest of the system concludes from that pin's voltage.
- **Body diodes in power paths.** In power multiplexers and load switches built from MOSFETs, check whether a switched-off MOSFET's body diode carries load current in some plug order; the drop and dissipation can exceed what the package allows.
- **Undefined outputs.** Isolators, level shifters and buffers can have undefined or default outputs while one side is unpowered; do not assume a logic level the datasheet does not guarantee.
- **Control inputs before firmware runs.** For each enable, mode or reset input, work out its voltage before firmware runs from all internal and external pulls that are active at reset (including other parts' default pull-ups), their tolerances and the assembly options (DNP). Compare it with the input's thresholds, and check that a function the system needs at power-up can start without firmware that is not yet running.
- **Strap and boot pins.** Pull resistors on pins sampled at reset select boot mode, flash voltage or addresses. Check the sampling time, the required level and any configuration that overrides it before calling a pull resistor wrong.
- **Sources tied together.** Two power inputs tied together with an instruction to connect only one: check what happens when both are connected, and whether the design makes that safe or impossible.
- **Driving before qualification.** A host driving chip-select or clock before the other side's logic supply is valid can back-power it or leave it undefined. Sequencing or a suitable interface may prevent this, but verify the whole circuit: open-drain signalling alone does not remove back-powering if its pull-up stays powered or a connected pin lacks power-off tolerance.

## Reporting power-state findings
For each finding, name the supply state, the evidenced current path, the affected node and the requirement it violates. Cite the internal structure from the datasheet (functional diagram, pin description) together with the user's connections, and separate an established consequence from a conditional one.
