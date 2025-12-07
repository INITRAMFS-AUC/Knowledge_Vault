
# Syntax errors

- `ahb_cache_writeback.v` : changed line 437 double semicolon `;;` show up as error `10170`
- `hazard3_intr_decompress.v` : deleted the `EXTENSION_ZCMP` generate if block as `EXTENSION_ZCMP` is not enabled for our SoC. 

>[!warning]
> This is a dirty trick that needs to be done more cleanly.


## Why was it necessary?
Quartus seems to be more strict than Yosys when it comes to syntax. 
- The double semicolon was unavoidable.
- EXTENSION_ZCMP is a bit more unique, it is unclear what exactly upset it. Quartus claims there is a begin without an end but we know this is not true so we beleive it is the ifdef that actually sets it off.

# Synthesis issues
Another important issue is that in `example_soc.v`