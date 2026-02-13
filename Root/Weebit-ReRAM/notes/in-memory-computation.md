# In-Memory Computing (IMC) with ReRAM

## The Core Mechanism

Neuromorphic computing uses ReRAM to perform math directly where the data is stored. It exploits **Ohm’s Law** and **Kirchhoff’s Current Law** to execute Matrix-Vector Multiplication (MVM) at the physical level.

* **Input ($V$):** Represented as **Voltage** applied to the horizontal rows.
* **Weights ($G$):** Stored as **Conductance** ($G = 1/R$) within each ReRAM cell.
* **Output ($I$):** Resulting **Current** summed at the bottom of each vertical column.

### The Physics of the Calculation

1. **Multiplication:** As voltage passes through a cell, the current produced is $I = V \times G$ (Ohm's Law).
2. **Addition:** All currents in a single column merge together naturally (Kirchhoff’s Law).


---

## Matrix Multiplication Mapping

In the crossbar array below, each box represents a memory cell. The labels ($a1, b1, c1, d1$, etc.) represent the specific conductance value of that cell.

![4x4 Crossbar array mapping math to hardware](../images/IMC-Maths.png)
*Figure 2: 4x4 Crossbar array mapping math to hardware.*

The output for each column (e.g., **Out1**) is the dot product of the input vector and the weight column:

$$Out_1 = (V_{in1} \cdot a_1) + (V_{in2} \cdot a_2) + (V_{in3} \cdot a_3) + (V_{in4} \cdot a_4)$$

### Why this is "Neuromorphic"

* **Efficiency:** Computation happens at the speed of electricity; no data is moved between a CPU and RAM.