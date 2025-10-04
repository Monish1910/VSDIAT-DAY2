# VSDIAT DAy-2:🚀

This README compiles my learnings and hands-on work across **Fundamentals of SoC Design**, the **VSDBabySoC mixed-signal SoC**, and the **RVMYTH–AVSD DAC interface**, focusing on **RTL-to-GDS**, **open-source EDA flows**, and **mixed-signal integration** using **sky130**.

---

## 🎯 Objectives

* **Understand end-to-end SoC design flow:** spec → RTL → synthesis → floorplan → placement → CTS → routing → signoff → GDS.
* **Integrate a RISC‑V core (RVMYTH)** with an on-chip 10‑bit DAC and PLL, and exercise open-source PnR with sky130.
* **Practice IP hardening**, LEF generation, and layout edits necessary for tool acceptance in mixed-signal SoCs.

---

## 🏗️ What I Built and Verified

* Simulated VSDBabySoC behaviorally: PLL drives RVMYTH; RVMYTH updates register `r17`; values feed the DAC to produce analog `OUT`.
* Explored modeling, **pre- and post-synth simulations** with `iverilog`/`gtkwave`, and observed equivalence expectations across the flow.
* Studied pin/LEF preparation, IO planning, and key naming/port consistency requirements for OpenLANE.

**Simulation Results (GTKWave)**
![Waveform of the `VSDBabySoC` mixed-signal design](/Screenshot from 2025-10-04 17-56-18.png)

---

## 🧠 Fundamentals of SoC Design (My Understanding)

An SoC integrates a CPU, memory, interconnect, and peripherals/IPs on a single die to achieve power, area, and performance goals for a given application.

**Flow overview:**
1.  Start with micro-architecture and IP selection.
2.  Perform RTL integration.
3.  Synthesize to gate-level.
4.  Physical design (floorplan, PDN, placement, CTS, routing).
5.  Signoff timing/DRC/LVS.
6.  Export GDS.

**Open-source stack highlights:** **Yosys** for logic synthesis, **OpenROAD/OpenLANE** for PnR, **Magic/KLayout** for layout, **Netgen** for LVS, with **sky130** as the open PDK.

---

## 🚀 Open-Source EDA Flow Takeaways

* Tech mapping with **abc** and timing analysis with **OpenSTA** ensure library-accurate gate-level results before PnR.
* **Hierarchical consistency matters:** Top module name must match design name; port names and directions must be consistent across RTL, LEF, and constraints to avoid integration failures.
* **Sky130 open PDK** plus **OpenLANE** makes reproducible RTL-to-GDS learning feasible without proprietary tools.

**Testbench Source View**
![Terminal output of the pre-synthesis simulation flow](/Screenshot from 2025-10-04 16-17-02.png)

---

## 🍼 VSDBabySoC Project (What I Did)

**Architecture:** RVMYTH core (digital), PLL (clock gen), 10‑bit DAC (analog), integrated via a wrapper; `OUT` is the analog output; `r17` values demonstrate core activity.

**Simulation stack:** `iverilog` + `gtkwave` for RTL/pre-synth validation; gate-level sims include standard cell models for post-synth checks of functional equivalence.

**Physical design focus:** Harden macro(s), add LEF, ensure pin directions/uses (signal/power/ground) are correct, then run OpenLANE interactive stages through placement.

**Key lessons from VSDBabySoC**
* **Mixed-signal acceptance hinges on proper LEF:** Pins, classes, and `USE`/`DIRECTION` metadata are essential; otherwise PnR tools reject the macro.
* Pin label → port conversion in Magic is essential; LEF regeneration is iterative and must reflect actual layout connectivity and layers.
* After floorplanning/placement, inspecting DEF with Magic validates geometry, pin spreads, and macro integration before proceeding to routing.

**Gate-Level Netlist Schematic**
![Waveforms of a gate-level netlist in GTKWave](/Screenshot from 2025-10-04 16-40-58.png)

---

## 🔌 RVMYTH–AVSD DAC Interface (My Workflow)

**Goal:** Feed RVMYTH’s 10‑bit digital output into a 10‑bit AVSD DAC, verify mixed-signal top in simulation, then drive PnR with sky130.

**Steps:** Simulate `rvmyth`, design/test `avsddac` in Verilog, build top-level interface, verify with `iverilog`/`gtkwave`, then prepare `LIB`/`LEF` for synthesis and PnR.

**Tooling:** **Yosys** for synthesis, **OpenSTA** for timing, **OpenLANE** for PnR, **Magic** for LEF and layout edits, **Netgen** for LVS in later stages.

**Critical integration details I applied:**
* Generated `.lib` for DAC from Verilog to enable timing-aware synthesis; included stdcell Verilog for gate-level sims to compare pre vs post behavior.
* Produced LEF for the DAC via Magic; added missing fields (`CLASS`, `PIN DIRECTION`/`USE`) to satisfy OpenLANE import, and verified pin naming matches RTL.
* Configured OpenLANE project with accurate `src` directories for `.v`/`.lib`/`.lef` and validated interactive flow up through floorplan/placement views in Magic.

**Commands and checkpoints I used:**
* `rvmyth` sim: clone `rvmyth`, compile `tb` + core, view VCD in `gtkwave` for 10‑bit outputs.
* `DAC` sim: compile `avsddac` + `tb`; confirm transfer characteristics and code transitions in VCD.
* `Top` sim: compile `rvmyth_avsddac` + TB; confirm `D[9:0]→Out` mapping and activity under core clock.
* **Post-synth sim:** include synthesized netlist and `sky130` stdcell models; ensure waveform parity with RTL.
* **OpenLANE interactive:** `prep design`, `add_lefs`, `run_synthesis`, `init_floorplan`, `place_io`, `global_placement_or`, `detailed_placement`, and inspect DEFs in Magic.

**DAC Simulation Results**
![GTKWave simulation of the DAC and its output values](/Screenshot from 2025-10-04 16-57-22.png)
![Another view of the DAC simulation showing different values](/Screenshot from 2025-10-04 17-14-55.png)

---

## ⚠️ Pitfalls I Resolved

* **LEF import failures** due to missing pin metadata; resolved by converting labels to ports and adding `DIRECTION`/`USE` classes per pin type.
* **Name mismatches** between top design name and file/module names causing synthesis errors; fixed by standardizing names across config and RTL.
* **Floorplan errors** from mismatched port lists; corrected by aligning RTL port lists with LEF pins and regenerating merged LEF before placement.

---

## 📈 Next Steps and Extensions

* Complete routing and full signoff for the integrated SoC, close timing, and generate final GDS for the `rvmyth_avsddac` top.
* Add PLL macro hardening and formal LVS between schematic/netlist and layout to validate mixed-signal correctness end to end.
* Explore STA corners, IR/EM checks, and signal integrity fixes to emulate industry-grade signoff on sky130.

---

## 🙏 Acknowledgments and References

* **SFAL–VSD program structure** and **open-source SoC design methodology** for guided progression from fundamentals to project execution.
* **VSDBabySoC community repos** and **modeling guides** for SoC composition, testbenching, and simulation workflows on Ubuntu.
* Additional foundational **SoC design notes** and **open-source flow descriptions** that informed RTL-to-GDS practices in this journey.
