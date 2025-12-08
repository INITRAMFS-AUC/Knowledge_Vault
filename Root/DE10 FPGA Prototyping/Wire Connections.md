
| **Wire**          | **Pico Pin** | **Color** | **DE-10 Pin** |
| ----------------- | ------------ | --------- | ------------- |
| **tck**           | GPIO 19      | Green     | GPIO 0        |
| **tdi**           | GPIO 18      | Red       | GPIO 1        |
| **tdo**           | GPIO 21      | Yellow    | GPIO 2        |
| **tms**           | GPIO 14      | Orange    | GPIO 3        |
| **trst_n**        | GPIO 15      | Brown     | GPIO 4        |
| **uart_rx**       | NO           | Blue      | GPIO 5        |
| **uart_tx**       | NO           | White     | GPIO 6        |
| **nReset**        | GPIO 16      | Black     | GND           |
| **Common Ground** | GND          | Black     | GND           |
`uart_rx` and `uart_tx` are then connected to a USB TTL device to allow us to get the UART output on our machine.

Thi