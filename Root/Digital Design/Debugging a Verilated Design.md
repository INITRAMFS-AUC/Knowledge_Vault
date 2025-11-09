---
tags:
  - csce/digital_design/simulation
  - csce/digital_design/software
  - csce/digital_design/debugging
---

>[!warning] The following note outlines a _potential_ toolchain for debugging
> It does not provide a complete implemented design, yet

If you use Verilator you can _verilate_ your verilog design, the verilated design can have its input/output exposed to the external system.

>[!note]- Verilated design
> Is the output of verilator when given a verilog project, it is simply code (can be CPP, SystemC) that models your hardware for simulation, this is much faster than [[Iverilog & GtkWave Simulation|Iverilog]]

To Debug a Verilated Design with OpenOCD you could expose the JTAG interface in your verilated design as outlined in [the VPI example of verilator docs:](https://verilator.org/guide/latest/connecting.html#vpi-example)











