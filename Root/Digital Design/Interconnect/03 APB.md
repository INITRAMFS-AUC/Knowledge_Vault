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

- In contrast to AHB-Lite APB is **not pipelined**.
- The interface of APB Requester to Completer is _symmetric_, meaning that same interface is on both sides.

# APB Bridge

![[Pasted image 20251028192812.png]]

The APB Bridge exposes a APB requester on one side and an AHB-Lite Completer Interface on the other side. 
This bridge is the glue that separates the high frequency clock domain from the low frequency clock domain.

>[!Quote]
>"All pipelined AHB-Lite transfers directed to the APB bridge from an AHB-Lite manager are automatically backpressured and handled over multiple clock cycles by the APB subsystem ... The APB bridge then relays a response back to the AHB-Lite manager and releases its backpressure. "

## AHB-to-APB State Machine

![[Pasted image 20251028193237.png]]

### IDLE 

"The state machine resets to an IDLE state in which it waits for an AHB-Lite transfer."

"When the bridge receives such a transfer, the state machine then transitions to a SETUP state in which the _PSEL_ signal corresponding to the target APB completer is asserted", The _PSEL_ signal here selects the completer were the request is going to be forwarded to.

### SETUP

- Gives the APB completer opportunity to carry out its request
- Given time to configure the multiplexer between bridge and APB completers, we configure it with _PSEL_ signals to forward the data from the correct completer

### ACCESS



