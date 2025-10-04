# SFAL–VSD SoC Journey: Consolidated README

This repository documents hands‑on learning across fundamentals of SoC design, the VSDBabySoC mixed‑signal SoC, and the RVMYTH–AVSD DAC interface on sky130, covering RTL‑to‑GDS, open‑source EDA flows, and mixed‑signal integration.

![Top Simulation 1](Screenshot from 2025-10-04-16-40-58.png)
![Top Simulation 2](Screenshot from 2025-10-04-16-57-22.png)
![RVMYTH+Dac TB View](Screenshot from 2025-10-04-16-32-46.png)
![DAC TB Transfer](Screenshot from 2025-10-04-16-17-02.png)
![RVMYTH Core TB](Screenshot from 2025-10-04-17-14-55.png)
![VSDBabySoC Waves](Screenshot from 2025-10-04-17-56-18.png)

---

### Objectives

- Understand end‑to‑end SoC flow: spec → RTL → synthesis → floorplan → placement → CTS → routing → signoff → GDS.
- Integrate a RISC‑V core (RVMYTH) with an on‑chip 10‑bit DAC and PLL, and exercise open‑source PnR on sky130.
- Practice IP hardening, LEF generation, and layout edits for mixed‑signal SoCs.

---

### What Was Built and Verified

- Behavioral VSDBabySoC sim: PLL drives RVMYTH; RVMYTH updates r17; r17[9:0] feeds DAC to produce analog OUT.
- Modeling plus pre/post‑synth sims with iverilog/gtkwave; observed equivalence expectations across the flow.
- Pin/LEF preparation, IO planning, and naming/port consistency requirements validated for OpenLANE import.

---

### Fundamentals Recap

- SoC = CPU, memory, interconnect, peripherals/IP on one die for PPA targets.
- Flow: micro‑architecture/IP selection → RTL integration → synthesis → PD (floorplan, PDN, place, CTS, route) → signoff (timing/DRC/LVS) → GDS.
- Open‑source stack: Yosys (synthesis), OpenROAD/OpenLANE (PnR), Magic/KLayout (layout), Netgen (LVS), sky130 open PDK.

---

### Open‑Source Flow Takeaways

- Tech mapping with abc and timing with OpenSTA for library‑accurate netlists pre‑PnR.
- Hierarchical consistency is critical: top module name matches design; port names/directions consistent across RTL, LEF, SDC/constraints.
- Sky130 + OpenLANE enables reproducible RTL‑to‑GDS without proprietary tools.

---

### VSDBabySoC Project

- Architecture: RVMYTH core (digital), PLL (clock gen), 10‑bit DAC (analog), integrated via a wrapper; OUT is analog; r17 shows core activity.
- Simulation: iverilog + gtkwave for RTL/pre‑synth; gate‑level adds sky130 standard cell models for functional checks.
- Physical: harden macro(s), generate LEF, correct pin DIRECTION/USE, then run OpenLANE interactive to placement and inspect DEF in Magic.

---

### Key Lessons

- Mixed‑signal acceptance hinges on proper LEF metadata (CLASS, PIN USE/DIRECTION, layers).
- Magic label→port conversion and LEF regeneration are iterative; ensure names and layers match actual layout connectivity.
- After FP/placement, inspect DEF in Magic to validate geometry, pin spread, and macro integration before routing.

---

### RVMYTH–AVSD DAC Interface Workflow

- Goal: Feed RVMYTH’s 10‑bit output to AVSD 10‑bit DAC; verify mixed‑signal top in sim; drive PnR on sky130.
- Steps: Simulate rvmyth, design/test avsddac Verilog, build top interface, verify with iverilog/gtkwave, prepare LIB/LEF for synthesis and PnR.
- Tools: Yosys, OpenSTA, OpenLANE, Magic, Netgen (later for LVS).

---

### Critical Integration Details

- Generated DAC .lib from Verilog to enable timing‑aware synthesis; included std‑cell Verilog for GLS parity with RTL.
- Produced DAC LEF via Magic; added CLASS and PIN DIRECTION/USE; verified pin naming matches RTL.
- Configured OpenLANE with correct src paths for .v/.lib/.lef; validated interactive flow through floorplan/placement and Magic DEF views.

---

### Commands and Checkpoints

- RVMYTH sim: compile TB + core, view VCD for 10‑bit outputs.
- DAC sim: compile avsddac + TB; confirm transfer characteristics and monotonic code transitions.
- Top sim: compile rvmyth_avsddac + TB; confirm D[9:0]→OUT mapping under core clock.
- Post‑synth sim: include synthesized netlist + sky130 models; check waveform parity.
- OpenLANE interactive: prep design → add_lefs → run_synthesis → init_floorplan → place_io → global_placement_or → detailed_placement → inspect DEF in Magic.

---

### Pitfalls Resolved

- LEF import failures from missing pin metadata; fixed with ports and proper DIRECTION/USE.
- Name mismatches between top design and module/files; standardized across config and RTL.
- Floorplan errors from mismatched port lists; aligned RTL ports with LEF pins and regenerated merged LEF.

---

### Next Steps

- Complete routing, close timing, full signoff, and final GDS for rvmyth_avsddac top.
- Harden PLL macro and run full LVS between schematic/netlist and layout.
- Explore STA corners, IR/EM checks, and SI mitigations to emulate industry‑grade signoff.

---

### Acknowledgments

- SFAL–VSD program structure and methodology for guided progression.
- VSDBabySoC community repos and modeling guides for SoC composition, TBs, and Ubuntu workflows.
- Foundational open‑source RTL‑to‑GDS references that informed practices here.

---

## Quickstart

