To run your code you will need the following:
- `openocd` running the config file specific to our JTAG probe
- `gdb` connecting to `openocd` and being used to load and debug the code
- `minicom` running at 115200 8N1 to interface with the USBTTL device connected to the FPGA

# OpenOCD
```
%% Run the following in the KWS-SoC-De10 dir%%
sudo riscv-openocd -f openocd.cfg
```

If everything is connected correctly from the pico acting as a jtagprobe to the FPGA we should see:
```
Info : Using CMSIS-DAPv2 interface with VID:PID=0x2e8a:0x000c, serial=E66141040374B02B
Info : CMSIS-DAP: SWD supported
Info : CMSIS-DAP: JTAG supported
Info : CMSIS-DAP: Atomic commands supported
Info : CMSIS-DAP: Test domain timer supported
Info : CMSIS-DAP: FW Version = 2.0.0
Info : CMSIS-DAP: Interface Initialised (JTAG)
Info : SWCLK/TCK = 0 SWDIO/TMS = 0 TDI = 0 TDO = 0 nTRST = 0 nRESET = 0
Info : CMSIS-DAP: Interface ready
Info : clock speed 500 kHz
Info : cmsis-dap JTAG TLR_RESET
Info : cmsis-dap JTAG TLR_RESET
Info : JTAG tap: hazard3.cpu tap/device found: 0xdeadbeef (mfg: 0x777 (Fabric of Truth Inc), part: 0xeadb, ver: 0xd)
Info : [hazard3.cpu] datacount=1 progbufsize=2
Info : [hazard3.cpu] Examined RISC-V core
Info : [hazard3.cpu]  XLEN=32, misa=0x40001105
[hazard3.cpu] Target successfully examined.
Info : [hazard3.cpu] Examination succeed
Info : [hazard3.cpu] starting gdb server on 3333
```

# GDB
This part is a bit more involved as it will depend on your goal. As an example we included code that will let you load a specific program into memory and add breakpoints and step through it using `gdb`.

```
riscv32-unknown-elf-gdb -x gdbinit

%% The following will be executed in gdb %%
file ./soc_test/c/hello_world.elf

load

b 37
%% 37 is the line where we put characters to uart %%

lay next

lay next

lay next

next

next

next

```

# minicom

The following opens minicom with baud rate 115200.

```
sudo minicom -D /dev/ttyUSB0 -b 115200
```

To find the right tty use the following:
```
dmesg | grep tty
```
