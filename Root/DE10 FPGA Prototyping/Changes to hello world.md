Hello world from our simulation code did not work out the box. This is mainly due to incorrect configuration of the uart clock divisor and settings to get the right baud rate.

To fix this we changed `uart_init` to the following:
```
void uart_init() { 
	// 1. Set Baud Rate Divisor for 115200 baud @ 36MHz 
	// Integer 39, Fraction 1/16 -> (39 << 4) | 1 = 0x271 
	UART_DIV = 0x271; 
	// 2. Enable UART (and optionally TX interrupt) 
	UART_CSR |= UART_CSR_EN; }
```
