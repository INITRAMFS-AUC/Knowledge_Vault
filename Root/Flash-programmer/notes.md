

# Interfacing with a GD24Q32 Flash Chip using CH341A + flashrom

This tutorial covers the basics of interfacing with a GD24Q32 flash chip (or similar 25-series BIOS chips) using a CH341A programmer and `flashrom`. 

---

## 🔌 Hardware Setup

* **Chip Type:** BIOS 25-series
* **Orientation:** The red wire on the ribbon cable must be positioned on the **opposite side of the RXD label** on the CH341A programmer
* **Connection:** Ensure the clip is securely seated on the chip pins

---

## 1 Detect the Chip 

Before reading or writing, verify that the programmer detects the chip. 

```bash
sudo flashrom -p ch341a_spi
```

**Expected output:**

```text
flashrom v1.2 on Linux 5.x (x86_64)
Found GigaDevice flash chip "GD25Q32(B/C/E/F)" (4096 KB, SPI) on ch341a_spi.
```

---

## 2 Back Up the Existing Firmware

Always perform **two separate reads** and compare them to ensure data stability.

### Read #1

```bash
sudo flashrom -p ch341a_spi -r backup.bin
```

### Read #2

```bash
sudo flashrom -p ch341a_spi -r backup2.bin
```

**Expected output (both runs):**

```text
Reading flash... done.
```

---

## 3 Verify Data Integrity

Compare the two backup files:

```bash
diff backup.bin backup2.bin
```

**Expected output:**

```text
(no output)
```

>  No output means the files are identical
>  If they differ, check your clip connection and repeat the reads

---

## 4 Inspect the Binary

View the first few lines of the backup:

```bash
hexdump -C backup.bin | head -n 16
```

**Example output:**

```text
00000000  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
*
000000a0  5a a5 f0 0f 03 00 00 00  01 02 03 04 05 06 07 08  |Z...............|
```

---

## 5 Create a Test Payload

Generate a 512-byte file filled with `0xAA`:

```bash
python3 -c "print('\xaa' * 512, end='')" > payload.bin
```

---

## 6 Prepare the Final Image

Merge the payload into a copy of your backup:

```bash
cp backup.bin to_flash.bin
dd if=payload.bin of=to_flash.bin conv=notrunc
```

**Expected output:**

```text
1+0 records in
1+0 records out
512 bytes copied, 0.0001 s, 5.1 MB/s
```

---

## 7 Write to the Flash 

Flash the modified binary back onto the chip:

```bash
sudo flashrom -p ch341a_spi -w to_flash.bin
```

**Expected output:**

```text
Reading old flash chip contents... done.
Erasing and writing flash chip... Erase/write done.
Verifying flash... VERIFIED.
```
