
| **Wire**          | **Pico Pin** | **Color** | **DE-10 Pin** |
| ----------------- | ------------ | --------- | ------------- |
| **tck**           | GPIO 19      | Green     | GPIO 0        |
| **tdi**           | GPIO 18      | Red       | GPIO 1        |
| **tdo**           | GPIO 21      | Yellow    | GPIO 2        |
| **tms**           | GPIO 14      | Orange    | GPIO 3        |
| **trst_n**        | GPIO 15      | Brown     | GPIO 4        |
| **uart_rx**       | NO           | Blue      | GPIO 5        |
| **uart_tx**       | NO           | White     | GPIO 6        |
| **nReset**        | GPIO 16      | Black     | GND (GPIO 11)?|
| **Common Ground** | GND          | Black     | GND           |
`uart_rx` and `uart_tx` are then connected to a USB TTL device to allow us to get the UART output on our machine.
Do no forget a common ground for the USB TTL device.

## QSPI Flash Connections

| **Wire**      | **Pico Pin** | **Color** | **DE-10 Pin** |
| ------------- | ------------ | --------- | ------------- |
| xip_csn       | GPIO 7       | Grey      | PIN_AH3       |
| xip_sck       | GPIO 8       | Purple    | PIN_AH4       |
| flash_io[0]   | GPIO 9       | Blue      | PIN_AH5       |
| flash_io[1]   | GPIO 10      | Green     | PIN_AG1       |
| flash_io[2]   | GPIO 12      | Yellow    | PIN_AG3       |
| flash_io[3]   | GPIO 13      | Orange    | PIN_AG5       |
