---
tags:
  - csce/digital_design/debugging
---
There are multiple approaches:
# Iverilog Simulation

>[!success] advantages
> Iverilog is an easy to use simulator that pairs well with GTKWave

>[!error] disadvantages
> - Debugging MCU sim using waveforms is cumbersome. You cannot debug your code output effectively.
> - Neither Iverilog nor GTKwave allow to modify signals at runtime, or use GDB for debugging, as you cannot even compile code to run or modify your MCUs memory.

# CPP Simulation 
These are the families of simulators that convert your RTL design to code.

>[!success] advantage
> These give you more control as they allow you to write wrappers around your code, ultimately what this enables, is writing JTAG servers around your simulated design for OpenOCD (and thus gdb) to connect to. 

>[!error] disadvantage
> You'd have to write your own code, it does not work out of the box
## CXXRTL
Refer to [[Debugging a CXXRTL Design]]

## Verilator
Refer to [[Debugging a Verilated Design]]

# FPGA Prototyping

>[!success] advantage
> This is closer to hardware implementation thus:
> - better performance 
> - can provide area and critical path data (although you would not use it for ASIC prototyping)

>[!error] disadvantage
> - needs FPGA
> - needs an MCU platform for JTAG bitbanging (depends on you FPGA dev board)
> - according to your fpga platform the development board may not have the necessary I/O for anything useful.


