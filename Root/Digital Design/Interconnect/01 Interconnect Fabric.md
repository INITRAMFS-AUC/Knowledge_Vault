---
tags:
  - csce/digital_design/hardware/interconnect
References:
  - "Arm Fundamentals SoC: Chapter 4: Interconnect"
---
# Interconnect Fabric 

## Why we need it?

The Interconnect Built in [[00 Basic 1 to 1 Interconnect]] only accommodates 1 Requestor and completer.

>[!note]- Interconnect Protocol Vs Interconnect Fabric
>The interconnect protocol affects the types of transactions that can occur between a given requestor–completer pair, whereas the interconnect fabric’s implementation affects the communication patterns that can occur among different requestor–completer pairs.

## How can we connect multiple completers to multiple Requestors?

The naive method would be making many Requestor interfaces for each completer but that would incur a lot of logic and chip area. As you increase the number of requestors, things get problematic as you would need a **complete bipartite** graph topology between requestors and completers, this does not scale well.


![[Pasted image 20251003185851.png]]


Thus we need something to **Multiplex** requestor signals to the needed completer. To do this we need to introduce an **interconnect Fabric**.

## Proposed Design

![[Pasted image 20251003190142.png]]

This is an elegant solution that simply lets requestor address the completers using an `ADDR` signal to the interconnect fabric. This is called the **address space**. All interconnects contain an **address decoder** which takes care of address resolution, this is just a bunch of comparators.

![[Pasted image 20251003190823.png]]

This arrangement of **address decoder** is very bulky, and introduces a long critical path reducing the interconnects maximum clock frequency. We can implement better address mapping to simplify the logic of address decoders.

Lets propose only putting completers's address space on power of 2 addresses.

![[Pasted image 20251003191255.png]]

This way, we can simplify our logic, and instead of using comparators for _both_ the upper bound and lower bound of the address space, we can just compare the $x$ most significant bit eliminating the need for a comparator and a subtractor:

![[Pasted image 20251003191451.png]]

## Arbitration 

When dealing with multiple requestors it is not uncommon to have two requestors request a signal from a completer at the same time. Therefore there must be an intermediary entity, an **arbiter** that takes into account requestor requests.

### Shared Bus Architecture

Shared bus architectures are viable when all completers and requestors share the same interconnect protocol interfaces. 

![[Pasted image 20251004114249.png]]

> All requestor and completer interconnect interfaces have to be guarded by a tri-state gate because only a single requestor–completer pair can be connected to the bus at any given time.

>[!Note] Implementation Note
> we need instead to use an additional out-of-protocol signal between the requestors and the arbiter for this purpose _(thus, shared buses use requestor-side arbitration)._
> The arbiter receives all requestor requests and sequentially grants bus access by enabling the specific requestor and completer’s tri-state gates.

>[!success] Advantages
> - Easy to Implement
> - Can operate at high clock speed
> - Economical in Hardware resources

>[!error] Disadvantages
> Does not Support any notion of concurrency, i.e, requestors cannot communicate with different completers in parallel.
> > [!quote] 
> > Shared-bus architectures can therefore cause requestors in a systems to spend a _significant amount of time waiting for access to the bus_, and waiting times get worse the more requestors exist in the system, which is an _unfavorable situation for SoCs, which often contain many independent accelerator units_ that act as requestors. Shared-bus architectures are, **therefore, rarely used in SoCs.**

### Full Crossbar-Switch Architecture

Full Crossbar-Switch Architectures are **Completer-side arbitration** and this allows for maximum concurrency in the system.

![[Pasted image 20251004115810.png]]
> On the left Completers Switches `2-SW` are closed as there is no requestor collision, On the right Completer's Switch is now open as there is requestor collision.

>[!note]  Implementation Note
>  "full crossbar switches do not require any extra out-of-protocol signals for arbitration purposes because _each switch can see when multiple transactions are simultaneously directed at the same completer_ and can _automatically provide backpressure_ on all requestors except the one that was granted access to the completer. Therefore, **the interconnect protocol itself acts as the arbitration mechanism**"
>  The Crossbar logic itself defines the _arbitration policy_: this can be round-robin priority of requestors, or priority access, basically a scheduler.

>[!error] disadvantages
> Full Cross-bar switches require a lot of resources area-wise on the die. The area of the cross-bar switch grows **quadratically** with the number of requestors and completers in the system. (as it is basically a square matrix of sitches)

### Partial Crossbar-Switch Architectures

>[!quote] 
In real-world SoCs, **some requestors may only ever need connectivity to a subset of the system’s available completers**. For example, while a CPU may need to be connected to most peripherals, a DMA unit may only need interconnection with memories. 

![[Pasted image 20251004123946.png]]

> Each completer switch is sized to only accommodate the requestors that communicate with it.

>[!success] Advantages
> - Offers all the advantages of [[#Full Crossbar-Switch Architecture]] 
> - Less die area than that of the Full Crossbar-Switch

## Design for Flexibility

![[Pasted image 20251004125857.png]]

Just like in software, your interconnect fabric can act as an **adapter** following the _adapter design pattern_, for example, if a completer does not support [[00 Basic 1 to 1 Interconnect#^2d40f7|Burst Transactions]], the interconnect fabric should be tasked with converted burst transaction to [[00 Basic 1 to 1 Interconnect#^cb00a1| Single Word Transactions]] the completer can understand, while stalling the requestor through its interface if needed.

--- 

For real world examples on Interconnects: [[02 AHB-Lite]]

---

