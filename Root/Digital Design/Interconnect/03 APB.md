---
tags:
  - csce/digital_design/hardware
References:
  - "Arm Fundamentals SoC: Chapter 5:  The Advanced Microcontroller Bus Architecture (AMBA)"
---

>[!summary] Advanced Peripheral Bus
> This is mainly introduced to hook up low performance devices to your SoC, it can integrate well with [[02 AHB-Lite]] serving by introducing an AHB-Lite to APB Bridge which is just an FSM.

>[!success]- What is remarkable in APB you can put it on a different clock domain as it isolates the components on the AHB-Lite interconnect
>This is done through [[#APB Bridge]]

![[Pasted image 20251028192338.png]]

- In contrast to AHB-Lite APB is **not pipelined**.
- The interface of APB Requester to Completer is _symmetric_, meaning that same interface is on both sides.

# APB Bridge

![[Pasted image 20251028192812.png]]

The APB Bridge exposes a APB requester on one side and an AHB-Lite Completer Interface on the other side. 
This bridge is the glue that separates the high frequency clock domain from the low frequency clock domain.

The APB interface acts as the manager and chip selects the completers, unlike the stateful mux in [[02 AHB-Lite]], this one is completely stateless.

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

The state machine then unconditionally transitions from the SETUP state to an ACCESS state
### ACCESS

- both PSEL and PENABLE are asserted 
- The transfer occurs if the completer is ready to handle it
- The state machine _remains in the ACCESS state_ for _as long as is necessary for the completer to perform the transfer._
	- if another AHB-Lite transfer is received by the APB bridge, then its state machine transitions back to the SETUP state in support of a further APB transfer (possibly to a different completer)
	- if no further AHB-Lite transfer is received, then the state machine transitions back to the IDLE state.

## Basic Transfers

### Read
![[Pasted image 20251028202401.png]]
> - As long as there is no requests being done it is in _IDLE_ state, See T0, T3, T7
> - Once there is a request we give time for the _PSEL_ signals to be asserted in _SETUP_ state, T1, T4
> - We unconditionally move from SETUP to we go to _ACCESS_ state, see T2, we remain in this state as long as the completer is required to carry out the request T5, T6. 

### Write









