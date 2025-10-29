---
tags:
  - csce/digital_design/hardware
References:
  - "[[02 AHB-Lite]]"
  - "[[00 Basic 1 to 1 Interconnect]]"
---
# SoC Design

## Need Investigation

### Flash memory
[[KWS SoC Notes#^d58e3d|Youssef Notes]] list a Flash memory module for sky130, SONOS Flash memory, we need to know if module is an IP/ or not.

## Included Design Choices w/ Substantiation

- SRAM
### Direct Memory Access (DMA) Controllers

[[DMA Controllers]] are needed to relieve CPU clock cycles, an example for this would be moving memory from the KWS sample FIFO to Internal memory, done by DMA, while CPU is put in low power or is processing another FIFO block.

## Excluded Design Choices w/ Substantiation
### Burst Transactions

[[00 Basic 1 to 1 Interconnect#^2d40f7|Burst Transactions]] are not needed as AHB-Lite already does pipelining, in context of AHB-lite as mentioned in [[02 AHB-Lite#^1d6007|Burst transaction section in AHB-Lite Document]]:

![[02 AHB-Lite#^5d269c]]


Bursts are not needed as: 
> I _think_ this might be needed if completers dynamically allocate/reallocate memory internally, which should not be the case in our SoC.
- [[03 ABP|ABP]], which were I/O would connect to, does not support burst transactions.


### Unneeded AHB-Lite Specs

[[02 AHB-Lite#^c87933|HMASTLOCK]] is not needed as we would not have two managers fighting on a single address

[[02 AHB-Lite#^9537cc|HRESP]] is not needed as it is used mainly in hot, plug in system which is not the case for KWS SoC.

[[02 AHB-Lite#^d9e54d|HPROT]] is also not needed in simpler SoCs, unneeded overhead.


