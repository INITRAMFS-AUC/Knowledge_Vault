---
tags:
  - csce/digital_design/hardware/interconnect
References:
  - "Arm Fundamentals SoC: Chapter 4: Interconnect"
---
> _This is more or less a summarization of Chapter 4, of Arm Fundamentals of SoC textbook_, with note author's input.

>[!note]- Requestor & Completer
> The Component that is _Requesting_ a resource/computation from another component, the _completer_, which completes this request.

# Synchronous Interconnect Protocol Devving
## Single Word Transactions

^cb00a1

>[!quote]- ARM SoC: Single Word Transaction
>The simplest form of transaction that we need an interconnect protocol to support is the movement of a single word from a requestor to a completer, that is, a single-word write transaction.
### Writing 
To Support *Single Word Write* Transaction we can simply add

| Signal        | Importance                                                      |
| ------------- | --------------------------------------------------------------- |
| WR            | Control Signal to Signify the intention to Write to a completer |
| WRDATA \[BUS] | Data Bus that holds the data to be Written                      |

#### Notes
- The `WR` signal and the `WRDATA` are recieved on the same rising edge.
### Reading
To Support *Single Word Write* Transaction we can simply add

| Signal        | Importance                                                         |
| ------------- | ------------------------------------------------------------------ |
| WR            | _Control Signal_ to Signify the intention to Write to a completer  |
| WRDATA \[BUS] | _Data Bus_ that holds the data to be Written                       |
| RD            | _Control Signal_ to Signify the intention to Read from a completer |
| RDDATA \[BUS] | _Data Bus_ To hold the data to be read by requestor                |
#### Notes
- The `RD` signal is received at rising edge 0 cycle 0, then the `RDDATA` on the _next_ rising edge

![[Pasted image 20251002163252.png]]
### Important notes

>[!warning] Assumption & disadvantages
> - This assumes the _completer_ can read/execute the request every clock cycle (1) ^DW1
> - We’ve assumed until now that there is only is one location, that is, only one register, in the completer (2) ^DW2

>[!tip] Asynchronous Reads
>nothing prevents an interconnect protocol from permitting read data to be returned during cycle C (and, in fact, many protocols do allow this); however, this would imply that the overall read operation is asynchronous and we have stated that the interconnect we are building is synchronous.

---

### Mitigating Drawback (2)

| Signal        | Importance                                                              |
| ------------- | ----------------------------------------------------------------------- |
| WR            | _Control Signal_ to Signify the intention to Write to a completer       |
| WRDATA \[BUS] | _Data Bus_ that holds the data to be Written                            |
| RD            | _Control Signal_ to Signify the intention to Read from a completer      |
| RDDATA \[BUS] | _Data Bus_ To hold the data to be read by requestor                     |
| ADDR \[BUS]   | _Address Bus_ To Hold the address of which the data is to be written to |
#### Waveform

![[Pasted image 20251002164032.png]]

## Burst Transactions

^2d40f7

A Major __Disadvantage__ with the current interconnect that its support for back-to-back rapid transaction is lack-luster and leaves clock cycles to be optimized.
In the general case of a completer with an N-cycle end-to-end read latency, the maximum data throughput will be:
$$
\frac{1}{N} \times freq_{\ interconnect}
$$

 To Mitigate this we can add a `LENGTH` Control Signal to signal to the completer how much data is given to it to write/read.

| Signal        | Importance                                                                   |
| ------------- | ---------------------------------------------------------------------------- |
| WR            | _Control Signal_ to Signify the intention to Write to a completer            |
| WRDATA \[BUS] | _Data Bus_ that holds the data to be Written                                 |
| RD            | _Control Signal_ to Signify the intention to Read from a completer           |
| RDDATA \[BUS] | _Data Bus_ To hold the data to be read by requestor                          |
| ADDR \[BUS]   | _Address Bus_ To Hold the address of which the data is to be written to      |
| LENGTH        | _Control Signal_ to Show how many words to read/write in a burst transaction |

### Read Waveform
![[Pasted image 20251002164741.png]]
### Write Waveform
![[Pasted image 20251002164822.png]]

## Backpressure

### Completer Requestor Backpressure

We Know [[#^DW2|Assumption (1)]] is not true as the completer may need to process the request for many clock cycles.

>[!Note] Terminiology
>The mechanism through which a completer communicates its unavailability to a requestor is referred to as _backpressure_ and a requestor waiting on a completer’s acknowledgement in this context is said to be _stalled_

#### Write Transaction w/ Backpressure
To realize this feature, all we need is to implement a `READY` signal.

| Signal        | Importance                                                                                               |
| ------------- | -------------------------------------------------------------------------------------------------------- |
| WR            | _Control Signal_ to Signify the intention to Write to a completer                                        |
| WRDATA \[BUS] | _Data Bus_ that holds the data to be Written                                                             |
| RD            | _Control Signal_ to Signify the intention to Read from a completer                                       |
| RDDATA \[BUS] | _Data Bus_ To hold the data to be read by requestor                                                      |
| ADDR \[BUS]   | _Address Bus_ To Hold the address of which the data is to be written to                                  |
| LENGTH        | _Control Signal_ to Show how many words to read/write in a burst transaction                             |
| READY         | _Control Signal_ From Completer to Requestor signaling that the Completer is busy carrying out a request |
- A transaction starts only when `WR/RD` and `READY` are both asserted in the same clock cycle.

>[!warning] Avoid Infinite Requestor Stalling
> if a requestor wants to start a transaction, then it should assert its control signals and keep them asserted until the completer’s READY signal is asserted, Two Design Decesions are employed here:
> 1. the requestor must not check the `READY` signal before deciding to assert its control signals because otherwise the completer may never know of the requestor’s intention to start a transaction
> 2. transactions cannot be interrupted once started because a requestor cannot deassert its control signals early, and all data words must be transmitted for both entities to acknowledge the transaction has finished.

^e99960

##### Waveform

![[Pasted image 20251002190522.png]]
> Notice That the Completer Asserts/Deasserts the `READY` signal for each data write `B` `C` & `D`.

#### Read Transaction w/ Backpressure

Reading from completors may also incur many clock cycles, There needs to be a control signal that registers that the data within the `RDDATA` bus is valid.

For a read transaction, the RD signal does not convey the validity of the `RDDATA` bus, but rather the validity of the control signals relevant to starting a transaction (ADDR, LENGTH). We need an additional completer-to-requestor signal to convey the validity of the `RDDATA` bus.

- So `RDDATAVALID` is born: once completer fetches data, the `RDDATAVALID`, is asserted.

| Signal        | Importance                                                                                                    |
| ------------- | ------------------------------------------------------------------------------------------------------------- |
| WR            | _Control Signal_ to Signify the intention to Write to a completer                                             |
| WRDATA \[BUS] | _Data Bus_ that holds the data to be Written                                                                  |
| RD            | _Control Signal_ to Signify the intention to Read from a completer                                            |
| RDDATA \[BUS] | _Data Bus_ To hold the data to be read by requestor                                                           |
| ADDR \[BUS]   | _Address Bus_ To Hold the address of which the data is to be written to                                       |
| LENGTH        | _Control Signal_ to Show how many words to read/write in a burst transaction                                  |
| READY         | _Control Signal_ From Completer to Requestor signaling that the Completer is busy carrying out a request      |
| RDDATAVALID   | _Control Signal_ from completor to requestor signaling that the data in the `RDDATA` BUS is ready to be read. |
##### Waveform

![[Pasted image 20251002192210.png]]

### Requestor Completer Backpressure

>[!Faq] Why would a Requestor Exert Backpressure on Completers?
> > after all, shouldn’t a requestor know what it wants to write to the completer before initiating a burst transaction? 
>
> This is the case when only a small amount of data needs transmitting, but if large amounts of data are involved some requestors may not know the extent of the payload they want to transmit at the outset. A common example would be a requestor reading from a FIFO.
> 
> ![[Pasted image 20251003182710.png]]

#### Write Transaction

To Implement Requestor to Completer Backpressure, you can do this by deasserting the `WR` signal then asserting it again.

![[Pasted image 20251003183017.png]]

#### Read Transaction

>[!warning] We cannot Approach it as Write Transaction
>Asserting RD when all of the data words from the previous transaction have not yet been received would be equivalent to the requestor telling the completer it wanted to start a new transaction as soon as the current one had terminated

Some protocols allow requestors to signal for completers future transactions, this done by giving the Completer a notice so that it can ensure its internal buffers accommodate the burst transaction.

We are not those protocols so we will add a new signal `RDDATAREADY` which when deasserted by the requester, the requester can stall the completer, and the completer will assert\hold the current data in `RDDATA` bus until `RDDATAREADY` is asserted again.

![[Pasted image 20251003184041.png]]

## Final Interconnect Protocol Interface Signals

| Signal        | Importance                                                                                                                                                     |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| WR            | _Control Signal_ to Signify the intention to Write to a completer                                                                                              |
| WRDATA \[BUS] | _Data Bus_ that holds the data to be Written                                                                                                                   |
| RD            | _Control Signal_ to Signify the intention to Read from a completer                                                                                             |
| RDDATA \[BUS] | _Data Bus_ To hold the data to be read by requestor                                                                                                            |
| ADDR \[BUS]   | _Address Bus_ To Hold the address of which the data is to be written to                                                                                        |
| LENGTH        | _Control Signal_ to Show how many words to read/write in a burst transaction                                                                                   |
| READY         | _Control Signal_ From Completer to Requestor signaling that the Completer is busy carrying out a request                                                       |
| RDDATAVALID   | _Control Signal_ from completor to requestor signaling that the data in the `RDDATA` BUS is ready to be read.                                                  |
| RDDATAREADY   | *Control Signal* from requestor to completer signaling that the completer needs to stall to wait for the requestor to read the next `RDDATA` word transaction. |

---

Next on how to connect multiple requestors & completers: [[01 Interconnect Fabric]]

---

