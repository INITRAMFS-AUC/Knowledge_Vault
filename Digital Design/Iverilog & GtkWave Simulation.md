---
tags:
  - "#csce/digital_design/software"
  - "#csce/digita_design/simulation"
References:
  - https://steveicarus.github.io/iverilog/usage/gtkwave.html
---
# Iverilog

**Make Sure your test-bench dumps waveform `.vcd` in the output, by appending: **

```verilog
initial
begin
   $dumpfile("<module name>_tb.vcd");
   $dumpvars(0,test);
end
```

## For Compiling

```bash
$ iverilog -o <output_filename> <testbench>.v <verilog_sources>.v

$ vvp <output_filename>

$ gtkwave <vcd_filename>.vcd &
```

## Example

Directory structure
```bash
-rw-rw-r-- 1 waseem waseem  939 Oct  2 14:43 ahbl_master_tb.v
-rw-rw-r-- 1 waseem waseem  291 Oct  2 14:43 readme.md
drwxrwxr-x 2 waseem waseem 4.0K Oct  2 14:55 src/
```

```bash
iverilog -o out ahbl_master_tb.v src/ahbl_master.v src/ahbl_slave.v src/ahbl_splitter_4.v
```

OR you can use `-I` Option or `wildcards *`

```
iverilog -o out ahbl_master_tb.v src/*.v
```
# GtkWave

```
gtkwave <vcd_filename>.vcd &
```

# Automation with A Makefile

```make
# Source files
V_SOURCES = $(wildcard src/*.v)
TB_SOURCE = ahbl_master_tb.v

# Output files
OUT_FILE = out
# Nasty string concatenation trick to maintain consistency
VCD_FILE = $(TB_SOURCE)cd

# Default target: runs the simulation and opens the waveform
all: wave

# Compile the Verilog sources
$(OUT_FILE): $(V_SOURCES) $(TB_SOURCE)
	iverilog -o $(OUT_FILE) $(TB_SOURCE) $(V_SOURCES)

# Run the simulation to generate the VCD file
$(VCD_FILE): $(OUT_FILE)
	vvp $(OUT_FILE)

# Target to run the simulation
sim: $(VCD_FILE)

# Open the waveform in GTKWave
wave: $(VCD_FILE)
	gtkwave $(VCD_FILE) &

# Clean up generated files
clean:
	rm -f $(OUT_FILE) $(VCD_FILE)
```

