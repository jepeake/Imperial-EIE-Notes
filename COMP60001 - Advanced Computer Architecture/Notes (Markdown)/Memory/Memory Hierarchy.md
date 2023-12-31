
- - - 

***Memory Hierarchy***

- - - 

***Memory Hierarchy***

![[Pasted image 20231117155613.png|500]]

***capacity:*** *registers << SRAM << DRAM*

***latency:*** *registers << SRAM << DRAM*

***bandwidth:*** *on-chip BW (between CPU & RF/SRAM) >> off-chip BW (between RF/SRAM & DRAM)*

- *wider on-chip bandwidth (more operations per second) compared to off-chip
- → ***want to cache data on-chip - as close as possible → faster access***

- ***SRAM → low latency access***
- ***DRAM → high latency access***

- - - 

***Levels of Memory Hierarchy***

![[Pasted image 20230814185234.png|600]]

***cache***
- *data fetched from main memory & allocated into higher level of cache memory - expecting that may have to access that data again*
- *limited capacity of cache → faster access times*
- ***capacity vs access time trade off***
- *hierarchy can include L1, L2, L3 cache - different capacities & different access times*

***registers***
- *→ **explicitly managed memory for operands that may be reused***

- - - 

***Management of Memory Hierarchy***

***small/fast storage e.g. registers:***
- *address usually specified in instruction*
- *→ for 32bit registers, only need 5bits to access (addresses)*
- *generally implemented directly as a register file*

***larger/slower storage e.g. main memory:***
- *address usually computed from values in registers*
- → *need to do address calculation from register(offset) + PC for main memory*

***management:***
- *generally implemented as a hardware-managed cache hierarchy (hardware decides what is kept in fast memory)*
- ***cache is invisible to the programmer (hardware managed)***
- ***software can provide hints (prefetching data)***

- - - 
