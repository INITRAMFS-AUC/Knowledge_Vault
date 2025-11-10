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

>[!example] Example 1 from docs
>```verilog
> module our #(
>     parameter WIDTH /*verilator public_flat_rd*/ = 32
>  ) (input clk);
>     reg [WIDTH-1:0] readme   /*verilator public_flat_rd*/;
>     reg [WIDTH-1:0] writeme  /*verilator public_flat_rw*/;
>     initial $finish;
>  endmodule
>```

> We can use this technique to _expose JTAG Pins to VPI_

>[!example] Example 2 on hazard3 Example SoC
> The JTAG Interface
>```verilog
>module hazard3_jtag_dtm #(
>	parameter IDCODE          = 32'h0000_0001,
>	parameter DTMCS_IDLE_HINT = 3'd4,
>	parameter W_PADDR         = 9,
>	parameter ABITS           = W_PADDR - 2 // do not modify
>) (
>	// Standard JTAG signals -- the JTAG hardware is clocked directly by TCK.
>	input  wire               tck,
>	input  wire               trst_n,
>	input  wire               tms,
>	input  wire               tdi,
>	output reg                tdo,
>	...
>```
>
> The Actual SoC
>```verilog
>module example_soc #(
>	parameter DTM_TYPE   = "JTAG",  // Can be "JTAG" or "ECP5"
>	parameter SRAM_DEPTH = 1 << 15, // Default 32 kwords -> 128 kB
>	parameter CLK_MHZ    = 12,      // For timer timebase
>
>	`include "hazard3_config.vh"
>) (
>	// System clock + reset
>	input wire               clk,
>	input wire               rst_n,
>
>	// JTAG port to RISC-V JTAG-DTM
>	input  wire              tck    /*verilator public_flat_rw*/,
>	input  wire              trst_n /*verilator public_flat_rw*/,
>	input  wire              tms    /*verilator public_flat_rw*/,
>	input  wire              tdi    /*verilator public_flat_rw*/,
>	output wire              tdo    /*verilator public_flat_rd*/,
>...
>```


We can then access these vpi connections throughout our `cpp` `vpi` wrapper:

> example on first example
```cpp
vpiHandle vh1 = vpi_handle_by_name((PLI_BYTE8*)"TOP.our.readme", NULL);
if (!vh1) vl_fatal(__FILE__, __LINE__, "sim_main", "No handle found");
const char* name = vpi_get_str(vpiName, vh1);
const char* type = vpi_get_str(vpiType, vh1);
const int size = vpi_get(vpiSize, vh1);
printf("register name: %s, type: %s, size: %d\n", name, type, size);  // Prints "register name: readme, type: vpiReg, size: 32"
```

What is needed to be done is to develop a JTAG VPI wrapper and Server that communicates with OpenOCD Over TCP for debugging _your circuit_

There are different Implementation/sources online for a JTAG Server:
1. The Actual `jtag_vpi` interface in `openocd` [source code](https://github.com/openocd-org/openocd/blob/6a3abda0/src/jtag/drivers/jtag_vpi.c)
2. This [JTAG Implementation on Github](https://github.com/fjullien/jtag_vpi)
## Toolchain Diagram

![[Untitled-2025-10-19-1159(1).svg]]






