---
tags:
  - csce/digital_design/hardware/interconnect
References:
  - "Arm Fundamentals SoC: Chapter 5:  The Advanced Microcontroller Bus Architecture (AMBA)"
---
> _This is more or less a summarization of Chapter 5, of Arm Fundamentals of SoC textbook_, with note author's input

>[!example]- Real World On-chip Buses 
> 
> | Origin              | Acronym      | Name                          |
> | ------------------- | ------------ | ----------------------------- |
> | Arm                 | APB          | Advanced Peripheral Bus       |
> |                     | AHB          | Advanced High-performance Bus |
> |                     | AXI          | Advanced eXtensible Interface |
> | Altera (Intel)      | Avalon Bus   | Avalon Bus                    |
> | IBM                 | CoreConnect  | CoreConnect                   |
> | ST Microelectronics | STBus        | STBus                         |
> | OpenCores           | Wishbone Bus | Wishbone Bus                  |

>[!example]- Real World Off-chip Buses
> 
> 
> | Origin | Acronym               | Name                    |
> | ------ | --------------------- | ----------------------- |
> | Intel  | FSB                   | Front Side Bus          |
> |        | QPI                   | QuickPath Interconnect  |
> |        | UPI                   | Ultra Path Interconnect |
> | AMD    | HT                    | HyperTransport          |
> |        | IF                    | Infinity Fabric         |
> |        | Infinity Architecture | Infinity Architecture   |

> SoCs use on-chip buses, as all their components reside on the same die

# AHB-Lite & APB Protocols
> Originally developed by ARM but then became an open standard.

(Advanced High-Performance Bus Lite) AHB-Lite and (Advanced Peripheral Bus) APB completers are often used together in a system to allow high-performance subsystems to communicate through AHB-Lite, while low-power subsystems interface with their higher-power counterparts using the slower APB protocol to save power.

![[Pasted image 20251004133655.png]]

Instead of _requestors_ and _completers_, another terminology is used, _Managers_ and _Subordinates_.

# AHB-Lite

AHB-Lite provides an alternative solution to this bus utilization challenge by allowing managers to issue bus operations in a **pipelined** fashion. Thus, each operation consists of two phases:
1. an address phase,
2. a data phase.

To support even higher bandwidth, AHB-lite utilizes _wide data buses_.

![[Pasted image 20251004135016.png]]

> [!note] Some High-Level Observations
> - supports _power-of-two bus sizes_ that range from _8 to 1024 bits_.
> - There are extra signals on the Subordinate side, `HREADY` and `HSEL`, that is because multiple subordinates can be connected to the same manager and the AHB-Lite protocol designers decided to expose the extra signaling to handle this multiplicity on the subordinate side of the interconnect.
>

## Implementation

![[Pasted image 20251004153340.png]]

- Notice that the `HREADY` signals from subordinate side are many while it is only one on the manager side, that is because if one completer is still carrying out a request _on the bus_ it needs to finish before the next request is issued, this ensures requests are carried out **in order** in the pipelined nature of this implementation.
- The **address decoder** is _combinational_, that means it is _stateless_ yet the multiplexer must be _stateful_, the multiplexer must be stateful to avoid the _infinite stalling_ problem outlined in [[00 Basic 1 to 1 Interconnect#^e99960|00 Basic 1 to 1 Interconnect]]
- `HSELx` is outputted from **address decoder**, it selects the correct subordinate based on the address requested from the manager.
- In contrast to the implementation in [[00 Basic 1 to 1 Interconnect]], we do not have a `RD` signal to subordinates, that is because we can use the `off` signal of `HWRITE` to signify read operations.

>[!warning] Due to the binary nature of the `HWRITE` signal, we cannot put bus in idle 
### Basic Operations
#### Write Operations
Assume these set of Write operations 

| Address | Data |
| ------- | ---- |
| 0x0     | 0xA  |
| 0x4     | 0xB  |
| 0x8     | 0xC  |
| 0xC     | 0xD  |
| 0x10    | 0xE  |
![[Pasted image 20251004162523.png]]
#### Read Operation Waveform
![[Pasted image 20251004163118.png]]

### New Signals 
#### `HTRANS`

`HTRANS` is a 2-bit signal sent from the manager to the subordinate to indicate the type of transfer being requested. 

| HTRANS | Type   | Description                                                                                           |
| ------ | ------ | ----------------------------------------------------------------------------------------------------- |
| b00    | IDLE   | notify the slave that this transfer does not need to be data transferred                              |
| b01    | BUSY   | allows the master to insert idle cycles in the middle of a burst to achieve master-slave backpressure |
| b10    | NONSEQ | indicates a non-sequential transfer (single or first of a burst)                                      |
| b11    | SEQ    | indicates the current transfer is the subsequent transfer of a burst                                  |
Operation of `HTRANS` signal:

![[Pasted image 20251004164601.png]]


#### `HSIZE`

Assume we are saving `uint8_t` or `uint16_t` in C, this operation would consume a lot of bus cycles if it is emulated as the data bus width would be fixed to `32` bit data buses, therefore we need a way for the requestor to signal to the completer the size of the data transaction so it can be done in one bus cycle.

`HSIZE`, a 3-bit signal that indicates the size of a single transfer in bytes. 

| HSIZE | Size (bytes) | Size (bits) |
| ----- | ------------ | ----------- |
| 0b000 | 1            | 8           |
| 0b001 | 2            | 16          |
| 0b010 | 4            | 32          |
| 0b011 | 8            | 64          |
| 0b100 | 16           | 128         |
| 0b101 | 32           | 256         |
| 0b110 | 64           | 512         |
| 0b111 | 128          | 1024        |
##### Waveform
![[Pasted image 20251004170856.png]]

#### `HBURST`

##### Why are Bursts Supported in AHB-Lite?

In [[00 Basic 1 to 1 Interconnect]] bursts were introduced to allow back-to-back transactions, but this is already enabled by default by the **pipelined nature** of AHB-Lite, however bursts are supported because this can _allow completers to prepare_ their internals for the data given in bursts possibly _relieving some backpressure exerted on requester_.

---
##### Definitions

>[!note] Each bus burst transfer is called a beat by AMBA.

>[!note] Incrementing bursts
>- The address of the next beat is computed as $(addr_{prev} + HSIZE)$, i.e. successive beats always have incrementing addresses. 

> [!note] Wrapping bursts 
>- The address of the next beat is computed as $(addr_{prev} + HSIZE)\ \%\ (\#beats × HSIZE)$, i.e. successive beats’ addresses wrap when they cross an address (or block-size) boundary defined by the product of the number of beats (defined by `HBURST`) and the data width (in bytes; defined by `HSIZE`).

##### Implementation

>[!warning] 
>Note that the total amount of data transferred in a burst is defined by multiplying the _number of beats_ by the _amount of data in each beat_ (as conveyed through the `HSIZE` bus). This definition implies that `HSIZE` **must** remain **constant** throughout the burst and that the manager cannot change the data width between beats.

| HBURST | Name   | Description                      |
|--------|--------|----------------------------------|
| 0b000  | SINGLE | Single-beat burst                |
| 0b001  | INCR   | Incrementing burst of undefined length |
| 0b010  | WRAP4  | 4-beat wrapping burst            |
| 0b011  | INCR4  | 4-beat incrementing burst        |
| 0b100  | WRAP8  | 8-beat wrapping burst            |
| 0b101  | INCR8  | 8-beat incrementing burst        |
| 0b110  | WRAP16 | 16-beat wrapping burst           |
| 0b111  | INCR16 | 16-beat incrementing burst       |
![[Pasted image 20251004174720.png]]

### Backpressure

#### Subordinate to Manager

- During a data phase, subordinates can deassert `HREADY` to insert wait states if they require more time to respond to a transfer, with the address phase of the next transfer only accepted when `HREADY` is high again.

#### Manager to Subordinate 

>[!error] Due to the pipelined nature of AHB-Lite, Address phase cannot be stalled
> **However**, does it make sense for the manager to stall a single word transaction, it should support the word widths that it requests, otherwise it would use `SEQ` state.
> **Also**, does it make sense for the manager to stall an `IDLE` state, it would just stall itself this way.
> So AHB-Lite _only supports stalling for burst transactions_ were the manager _runs out of memory in internal buffers_.










