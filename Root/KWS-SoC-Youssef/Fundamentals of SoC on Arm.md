
![[Pasted image 20251021212445.png]]
## Concepts
- **A system on a chip** is “simply” every element of a computer on a single integrated circuit. However, SoCs cannot accommodate enough memory for a full operating system such as Linux and, thus, some external memory must be provided.
- 
## Chapters Of interest
- Chapter 3: **Architecture of the Arm Cortex-M processor & ISA.**
- Chapter 4: **Internal buses in terms of general interconnections**
- Chapter 5: **Specific Arm buses**
- Chapter 6: **Hardware signals and standard protocols used in SoC**


![[Pasted image 20251021213938.png]]

### Processor Cores
Selection Criteria of microprocessors.
- **size of the data bus**, in terms of 4, 8, 16, 32 or 64 bits of data path.
- **size of the address bus** needed to access the volume of memory that will be required.

![[Pasted image 20251021215709.png]]
### Registers and the Control Unit
To write or read the content of these registers, access is realized through one or more internal data buses.
![[Pasted image 20251021220127.png]]
![[Status register and some specific registers.png]]
The code is held in a memory external to the processor itself, but it can be on the same die. The contents of the **PC** register is passed to the external **address bus** to select the instruction. To transfer data in and out of the processor, two specific buffers access the **external data bus**.
Sometimes a bidirectional buffer is used, with a **tri-state buffer** allowing the direction of transfer to be differentiated.
![[Tri-state Buffer.png]]
When both instructions and data are made available through the same pairing of address bus and data bus, the architecture is described as **von Neumann**. It uses **Unified memory space**. Using a common bus reduces the physical lines, and allows the shared bus to be wide.
![[memory and buses in von Neumann architecture.png]]
![[memory and buses in Harvard architecture.png]]

### 1.5.3 Memory
a **memory map**, a model of the memories available on a system.
![[Pasted image 20251021223241.png]]
### 1.5.4 Memory Classes
