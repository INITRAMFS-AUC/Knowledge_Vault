
# Syntax errors

- `ahb_cache_writeback.v` : changed line 437 double semicolon `;;` show up as error `10170`
- `hazard3_intr_decompress.v` : deleted the `EXTENSION_ZCMP` generate if block as `EXTENSION_ZCMP` is not enabled for our SoC. 

>[!warning]
> This is a dirty trick that needs to be done more cleanly.


## Why was it necessary?
Quartus seems to be more strict than Yosys when it comes to syntax. 
- The double semicolon was unavoidable and had to go.
- `EXTENSION_ZCMP` is a bit more unique, it is unclear what exactly upset it. Quartus claims there is a begin without an end but we know this is not true so we beleive it is the ifdef that actually sets it off.

# Synthesis issues
Another important issue is that in `example_soc.v` we had to change `dmi_prdata` from a reg to a wire as it is being used as an input or output so cannot be a reg.

# Pins
We used the User manual for pin assignment.
This involved just mapping our inputs and outputs to GPIOS.

The only exception was the necessity for a clock PLL in order to resolve the default 50MHz clock into a 36MHz.

# UART
UART is the reason we chose 36MHz in order to get a baud rate of 9600.

