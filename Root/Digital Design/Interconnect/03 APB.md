---
tags:
  - csce/digital_design/hardware
References:
  - "Arm Fundamentals SoC: Chapter 5:  The Advanced Microcontroller Bus Architecture (AMBA)"
---

>[!summary] Advanced Peripheral Bus
> This is mainly introduced to hook up low performance devices to your SoC, it can integrate well with [[02 AHB-Lite]] serving by introducing an AHB-Lite to APB Bridge which is just an FSM.

In contrast to AHB-Lite APB is **not pipelined**, It is based on _state machine_ logic.

>[!success] What is remarkable in APB you can put it on a different clock domain
> - In short you can have the AHB-Lite to APB link be the 

