# Buck Converter — Design, Simulation & PCB Layout

A 12V → 5V DC-DC buck converter, taken from hand calculation through LTspice simulation to a manufacturable PCB layout in KiCad.

## Specifications

| Parameter                       | Target               |
| -------------------------------- | -------------------- |
| Input voltage (Vin)             | 12 V                 |
| Output voltage (Vout)           | 5 V                  |
| Load current (Iout)             | 1 A                  |
| Switching frequency (fs)        | 100 kHz              |
| Duty cycle (D)                  | ≈ 0.417              |
| Inductor ripple current (ΔI_L)  | 0.2 A (target)        |
| Output voltage ripple (ΔV_out)  | 50 mV / ~1% (target)  |

---

## Part 1: Design & Simulation (LTspice)

### Design approach

The converter was designed from first principles rather than picking arbitrary component values:

1. **Duty cycle** was calculated from the ideal buck relationship: D = Vout / Vin
2. **Inductance** was sized to keep inductor ripple current at a target 20% of load current, using:
   L = (Vin − Vout) × D / (ΔI_L × fs)
3. **Output capacitance** was sized to keep output ripple voltage near 1% of Vout, using:
   C = ΔI_L / (8 × fs × ΔV_out)

This gave calculated values of **L ≈ 146 µH** and **C ≈ 5 µF**, which were then used to build and simulate the circuit rather than just left as theoretical numbers.

### Circuit

Built in LTspice using:

- An ideal voltage-controlled switch (`SW`), driven by a `PULSE` source acting as the PWM gate signal
- A freewheeling diode for the inductor current path when the switch is off
- LC output filter (L1, C1) and a resistive load (R1 = 5Ω, sized for 1A at 5V)

Switch model:

```
.model SW1 SW(Ron=0.01 Roff=1Meg Vt=2.5 Vh=0.1)
```

PWM drive:

```
PULSE(0 5 0 10n 10n 4.17u 10u)
```

### Results

Measured via `.tran` transient simulation and `.meas` directives on steady-state waveforms:

| Metric                            | Value        |
| ---------------------------------- | ------------ |
| Output voltage (Vout)             | ≈ 4.53 V     |
| Average inductor current (IL_avg) | ≈ 0.91 A     |
| Input power (Pin)                 | ≈ 4.54 W     |
| Output power (Pout)               | ≈ 4.10 W     |
| **Efficiency**                    | **≈ 90.3 %** |

- Output voltage settled close to the 5V target; the small gap (~0.47 V) is attributable to non-ideal switch on-resistance and diode forward voltage drop, which aren't captured in the ideal hand calculation.
- Inductor current showed a clean, repeating sawtooth ripple centered near the 1A design load, confirming steady-state operation matched the design intent.
- The ~9.7% power loss is consistent with conduction loss in the switch (Ron) and forward-voltage drop across the freewheeling diode — both of which are excluded from the idealized hand-calculated design.

### Simulation files

- `buckconvertor.asc` — LTspice schematic
- `waveforms/` — simulation output screenshots (Vout settling, inductor current ripple)

---

## Part 2: PCB Design (KiCad)

To take the design beyond simulation, I laid out a real printed circuit board in KiCad, translating the same buck converter topology into a manufacturable design.

### Design approach

Rather than replicating the idealized switch from the LTspice simulation, I designed around the **LM2596S-5.0**, an industry-standard integrated buck regulator IC. This kept the physical topology consistent with the simulated circuit (the same external inductor, freewheeling diode, and output capacitor filter) while avoiding the added complexity of a discrete MOSFET + gate driver + compensator design, which was outside the scope of what I wanted this project to defend in depth.

**Bill of Materials:**

| Ref | Part | Value/Rating |
| --- | --- | --- |
| U1 | LM2596S-5.0/NOPB | TO-263-5, fixed 5V, 3A |
| L1 | Shielded power inductor | 33–68 µH, ≥3A saturation |
| D1 | Schottky diode | 1N5822 / SS34, 3A |
| Cin | Electrolytic | 100 µF |
| Cout | Low-ESR electrolytic | 100–220 µF |
| J1, J2 | 2-pin terminal block | 5mm pitch |

### Layout considerations

The most important layout decision was minimizing the **power loop** — the high di/dt switching path formed by U1's OUT pin, D1, and L1. This loop was routed as short and tight as possible to limit EMI and voltage ringing at the switch node, with input and output capacitors placed immediately adjacent to the pins they filter.

The board uses a 2-layer stackup with a solid ground pour on the bottom copper layer. Since component GND pads sit on the top layer while the ground plane is on the bottom, I added stitching vias at each GND pad to tie the two together — without them, ERC/DRC correctly flagged these as disconnected "islands," which was a useful reminder that a ground plane doesn't automatically connect to pads on a different layer.

Trace widths on the power path (VIN, switch node, Vout, GND) were sized using the IPC-2221 current-carrying capacity formula for ~2A on 1oz copper at a 10°C temperature rise, rather than left at default.

### What I'd flag if building this for real

- The footprint used for L1 is a placeholder — a real build would use the exact manufacturer footprint for a chosen shielded power inductor (e.g. Bourns SRN6045-330M).
- The LM2596's internal switching frequency (~150kHz) differs from the 100kHz PWM used in the LTspice simulation, since the IC has its own fixed oscillator rather than the externally-driven ideal switch modeled earlier.

### PCB files

- `pcb/buck_convertor.kicad_pro`, `.kicad_sch`, `.kicad_pcb` — full KiCad project
- `pcb/gerbers.zip` — Gerber + drill files, fab-ready
- `pcb/BOM.csv` — bill of materials
- `pcb/images/` — schematic, routed layout, and 3D render screenshots

This is a design-only deliverable — the board has not been fabricated or assembled.

---

## Tools

LTspice (SPICE circuit simulation), KiCad (schematic capture & PCB layout)
