---
tags:
  - csce/digital_design/simulation
  - csce/digital_design/software
  - csce/digital_design/debugging
---

# Conceptual Flow

>[!warning] The following note outlines a _potential_ toolchain for debugging
> It does not provide a complete implemented design, yet

If you use Verilator you can _verilate_ your verilog design, the verilated design can have its input/output exposed to the external system.

>[!note]- Verilated design
> Is the output of verilator when given a verilog project, it is simply code (can be CPP, SystemC) that models your hardware for simulation, this is much faster than [[Iverilog & GtkWave Simulation|Iverilog]]

To Debug a Verilated Design with OpenOCD you could expose the JTAG interface in your verilated design as outlined in [the VPI example of verilator docs:](https://verilator.org/guide/latest/connecting.html#vpi-example)


```verilog
 module our #(
     parameter WIDTH /*verilator public_flat_rd*/ = 32
  ) (input clk);
     reg [WIDTH-1:0] readme   /*verilator public_flat_rd*/;
     reg [WIDTH-1:0] writeme  /*verilator public_flat_rw*/;
     initial $finish;
  endmodule
```

> We can use this technique to _expose JTAG Pins to VPI_

```verilog
module hazard3_jtag_dtm #(
	parameter IDCODE          = 32'h0000_0001,
	parameter DTMCS_IDLE_HINT = 3'd4,
	parameter W_PADDR         = 9,
	parameter ABITS           = W_PADDR - 2 // do not modify
) (
	// Standard JTAG signals -- the JTAG hardware is clocked directly by TCK.
	input  wire               tck /*verilator public_flat_rw*/,
	input  wire               trst_n /*verilator public_flat_rw*/,
	input  wire               tms /*verilator public_flat_rw*/,
	input  wire               tdi /*verilator public_flat_rw*/,
	output reg                tdo /*verilator public_flat_rd*/,
	...
```

We can then build our `sim_`

## Diagram

![[Untitled-2025-10-19-1159(1).svg]]






