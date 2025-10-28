---
tags:
  - csce/digital_design/hardware
References:
  - "Arm Fundamentals SoC: Chapter 5:  The Advanced Microcontroller Bus Architecture (AMBA)"
---

>[!summary] Advanced Peripheral Bus
> This is mainly introduced to hook up low performance devices to your SoC, it can integrate well with [[02 AHB-Lite]] serving by introducing an AHB-Lite to APB Bridge which is just an FSM.

>[!success] What is remarkable in APB you can put it on a different clock domain as it isolates the components on the AHB-Lite interconnect

![[Pasted image 20251028192338.png]]

- In contrast to AHB-Lite APB is **not pipelined**, It is based on _state machine_ logic.
- The interface of APB Requester to Completer is _symmetric_, meaning that same interface is on both sides.

# APB Bridge

![[Pasted image 20251028192812.png]]

The APB Bridge exposes a APB requester on one side and an AHB-Lite Completer Interface on the other side. 
This bridge is the glue that separates the high frequency clock domain from the low frequency clock domain.

