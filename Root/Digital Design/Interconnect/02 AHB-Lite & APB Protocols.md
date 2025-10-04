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

- Notice that the `HREADY` signals from subordinate side are many while it is only one on the manager side, that is because if one completer is still carrying out 