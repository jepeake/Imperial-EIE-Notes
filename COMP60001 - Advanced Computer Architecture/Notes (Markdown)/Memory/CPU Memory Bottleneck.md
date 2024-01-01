
- - - 

***CPU Memory Bottleneck***

- - - 

***Computer Architecture Terminology***

***Latency (in seconds or cycles)*** 
→ *time taken for a single operation from start to finish*

***Bandwidth (in operations/second or operations/cycle)*** 
→ *maximum rate at which operations can be performed (maximum throughput) 
(bus → maximum rate at which data transferred)*

***Occupancy (in seconds or cycles)*** 
→ *time during which the unit is blocked on an operation (structural hazard)*

- - - 

***Examples***

***Occupancy < Latency***
- *→ e.g. 3-stage pipelined multiplier (latency = 3 cycles)*
- *→ first stage is blocked for one cycle - blocking the unit from receiving other values (occupancy)* 
- *→ so occupancy is less than latency for that functional unit*

***Occupancy > Latency***
- *→ e.g. DRAM - has completed operation (latency done) - but is still blocked from a new operation as needs to precharge (occupancy)*

***Bandwidth > 1/Latency***
- *→ pipelined multi-stage functional unit - can finish an operation every cycle but single operation takes multiple cycles*

***Bandwidth < 1/Latency***
- *→ DRAM - finished operation but still blocked*

- - - 

***CPU Memory Bottleneck***

 → ***performance of high-speed computers usually limited by memory bandwidth & latency (number of memory accesses per unit time & time taken for a single access)***

***memory bandwidth:***
- *→ every instruction needs to be accessed from instruction memory*
- → *some number of instructions access data memory*

***memory latency:***
- *→ time taken to perform memory access operation* 
- *→ >> processor cycle time*

***memory occupancy:***
- *→ time memory bank is occupied*

- - - 

***Processor-DRAM Gap***

- *DRAM getting denser & denser - but speed not increasing significantly (compared to processor speeds)*

- - - 

***Physical Size Affects Latency***

***bigger memory***
- → *slower to access*
- → *must travel longer distances to access data*
- → *longer fan-out & need to drive multiple loads/more data*
- → ***increasing memory access latency with bigger chips***

- - - 

