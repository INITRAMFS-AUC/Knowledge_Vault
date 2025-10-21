
## Plan to build the KWS SoC
1. Establish a prototype **MVP**. For this use **SDK** without hardware!
2. Answer questions list of **1.5 Elements of SoC Solutions**.
	1. Computing Power needed -->  algorithms
	2. electrical power needed.
	3. Battery Power / external Power.
	4. bandwidth of the data to be managed and/or transferred, and the number of operations per second involved
	5. communications needed
		- wireless/RF (e.g. Bluetooth, Wi-Fi, LoRa, SigFox, ZigBee, GSM, 3G, 4G, 5G, etc.)
		- cabled (e.g. Ethernet 10/100/1G, CAN bus, USBxx, etc.)
	6. **memory** type and size that will be needed or could be expected.
		- Will the processor be required to **start executing code following a Reset**? If so, then nonvolatile memory will be required.
	7. extension connectors
		- Standard (PCIe, VME, etc.)
		- serial (UART, USB, etc.)
		- proprietary
		- for debugging (JTAG, SWD, etc.)
	8. devices to include, such as accelerometers.
	9. where the programmable interfaces will be mapped: in the memory space, or in a specific input/output (IO)-space?


### Non-volatile memory for Hazard & SKY130
| External Flash Memory                                                                     | On -chip flash memory                                                                                                                                                                                             |
| ----------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The CPU accesses external memory through an AHB-attached flash controller and bus wrapper | [SONOS flash memory](https://skywater-pdk.readthedocs.io/en/main/rules/device-details.html) macro provided for SKY130. Instantiate it in your SoC layout and connect it to your AHB5 interconnect as an AHB slave |
In **RTL simulation (CXXRTL/Verilator for Hazard3)**, you **do not run the physical SONOS device**. Instead instantiate a behavioral flash model: usually a simple memory image for reads plus a controller model that simulates erase/program latencies and failures. For boot testing you can preload the flash from an ELF/bin file.