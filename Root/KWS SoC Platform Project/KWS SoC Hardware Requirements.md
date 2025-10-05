---
tags:
  - csce/digital_design/hardware
---
[[00 Basic 1 to 1 Interconnect#^2d40f7|Burst Transactions]] are not needed as AHB-Lite already does pipelining, in context of AHB-lite as mentioned in [[02 AHB-Lite#^1d6007|Burst transaction section in AHB-Lite Document]]:

![[02 AHB-Lite^backpressure_ahb]]


![[02 AHB-Lite#^c87933]]

`HMASTLOCK` is not needed as we would not have two managers fighting on a single address