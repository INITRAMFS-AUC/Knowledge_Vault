# Goal
We want to write a piece of code into the FLASH, Execute it directly from the FLASH.

# Plan

## JEDEC ID
1. Read the "Electronic Signature" or "JEDEC ID" from the chip to confirm your SPI communication

The Read Identification (RDID) command is a standard instruction used to identify the flash memory's manufacturer and specific device characteristics.

When this command is issued, the device outputs a 24-bit identification code comprising the manufacturer ID followed by two bytes of device identification

In GigaDevice GD25Q32C, the expected Bytes: 0xC8 (Manuf), 0x40 (Type), 0x16 (Capacity). See Page 15 of GD25Q32C-DATASHEET.

## Autopolling

If you write a command to flash while it is busy, it will ignore the command silently. To prevent this, we poll the `Write in Progress (WIP)` bit inside the `Status Register` until it becomes `0`.

HAL provide the function for that, but there's a structure you need to setup: `QSPI_CommandTypeDef`.

## Enabling Quad mode

For this we implemented a function `QSPI_EnableQuad` That sets the `QE` bit in `Status Regiser 2`.

## Confirming Quad Mode operational with ID read

Implemented `QSPI_ReadIDQuad` the returned two bytes (0xc8, 0x15) as expected according to ID definitions table in the Datasheet.