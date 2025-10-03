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



