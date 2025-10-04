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

## AHB-Lite

AHB-Lite provides an alternative solution to this bus utilization challenge by allowing managers to issue bus operations in a **pipelined** fashion. Thus, each operation consists of two phases:
1. an address phase,
2. a data phase.

To support even higher bandwidth, AHB-lite utilizes _wide data buses_.

![[Pasted image 20251004135016.png]]

> [!note] Some High-Level Observations
> - supports _power-of-two bus sizes_ that range from _8 to 1024 bits_.
> - There are extra signals on the Subordinate side, `HREADY` and `HSEL`, that is because multiple subordinates can be connected to the same manager and the AHB-Lite protocol designers decided to expose the extra signaling to handle this multiplicity on the subordinate side of the interconnect.
>

### Implementation

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

`HTRANS` is a 2-bit signal sent from the manager to the subordinate to indicate the type of transfer being requested. 

| HTRANS | Type   | Description                                                                                           |
| ------ | ------ | ----------------------------------------------------------------------------------------------------- |
| b00    | IDLE   | notify the slave that this transfer does not need to be data transferred                              |
| b01    | BUSY   | allows the master to insert idle cycles in the middle of a burst to achieve master-slave backpressure |
| b10    | NONSEQ | indicates a non-sequential transfer (single or first of a burst)                                      |
| b11    | SEQ    | indicates the current transfer is the subsequent transfer of a burst                                  |
