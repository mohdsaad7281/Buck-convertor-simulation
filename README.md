# Buck-convertor-simulation
# Buck Converter Design & Simulation

A 12V → 5V DC-DC buck converter, designed by hand calculation and validated through transient simulation in LTspice.

## Specifications

| Parameter | Target |
|---|---|
| Input voltage (Vin) | 12 V |
| Output voltage (Vout) | 5 V |
| Load current (Iout) | 1 A |
| Switching frequency (fs) | 100 kHz |
| Duty cycle (D) | ≈ 0.417 |
| Inductor ripple current (ΔI_L) | 0.2 A (target) |
| Output voltage ripple (ΔV_out) | 50 mV / ~1% (target) |

## Design Approach

The converter was designed from first principles rather than picking arbitrary component values:

1. **Duty cycle** was calculated from the ideal buck relationship: D = Vout / Vin
2. **Inductance** was sized to keep inductor ripple current at a target 20% of load current, using:
   L = (Vin − Vout) × D / (ΔI_L × fs)
3. **Output capacitance** was sized to keep output ripple voltage near 1% of Vout, using:
   C = ΔI_L / (8 × fs × ΔV_out)

This gave calculated values of **L ≈ 146 µH** and **C ≈ 5 µF**, which were then used to build and simulate the circuit rather than just left as theoretical numbers.

## Circuit

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

## Results

- **Output voltage** settled in steady-state close to the 5V target, with a small gap attributable to non-ideal switch on-resistance and diode forward voltage drop (component non-idealities not captured in the ideal hand calculation)
- **Output ripple** measured at approximately 60 mV peak-to-peak, close to the 50 mV design target
- **Inductor current** showed a clean, repeating sawtooth ripple centered near the 1A design load, confirming steady-state operation matched the design intent

*(Efficiency and average power loss figures to be added — measured via `.meas` transient analysis on diode conduction loss and input/output power.)*

## What I'd improve next

- Replace the generic default diode with a Schottky diode model (e.g. 1N5819) to reduce forward voltage drop and improve efficiency, since the default SPICE diode model isn't representative of what would be used in a real design
- Extend the design to a boost converter (5V → 12V) using the same methodology, to compare step-down vs. step-up topologies

## Files

- `buck_converter.asc` — LTspice schematic
- `waveforms/` — simulation output screenshots (Vout settling, inductor current ripple)

## Tools

LTspice (SPICE circuit simulation)
