# Goal
We want to write a piece of code into the FLASH, Execute it directly from the FLASH.

# Plan

## JEDEC ID (Done)
1. Read the "Electronic Signature" or "JEDEC ID" from the chip to confirm your SPI communication

The Read Identification (RDID) command is a standard instruction used to identify the flash memory's manufacturer and specific device characteristics.

When this command is issued, the device outputs a 24-bit identification code comprising the manufacturer ID followed by two bytes of device identification

In GigaDevice GD25Q32C, the expected Bytes: 0xC8 (Manuf), 0x40 (Type), 0x16 (Capacity). See Page 15 of GD25Q32C-DATASHEET.

## Autopolling (Done)

If you write a command to flash while it is busy, it will ignore the command silently. To prevent this, we poll the `Write in Progress (WIP)` bit inside the `Status Register` until it becomes `0`.

HAL provide the function for that, but there's a structure you need to setup: `QSPI_CommandTypeDef`.

## Enabling Quad mode (Done)

For this we implemented a function `QSPI_EnableQuad` That sets the `QE` bit in `Status Regiser 2`.

## Confirming Quad Mode operational with ID read (Done)

Implemented `QSPI_ReadIDQuad` the returned two bytes (0xc8, 0x15) as expected according to ID definitions table in the Datasheet.

## Read from Flash in memory-mapped mode
Implement QSPI memory-mapped mode and verify default 0xFF read at base address (done)

## Sector Erase and Page Program (TODO)


## The "Manual" Bootloader (TODO)
1. You write code in your internal Flash that acts as a manager.

2. Store: You embed the "external" binary data as a large array in your internal Flash.

3. Copy: Your internal code initializes QSPI in Indirect Mode, erases the external Flash, and uses Page Program to "self-flash" that array into the external chip.

4. Switch: Your code then switches QSPI to Memory-Mapped Mode.

5. Jump: Your code tells the CPU to jump to 0x90000000.
