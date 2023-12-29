
- - - 

***Iron Law of Processor Performance***

- - - 

- ***Processor Performance → Latency (time it takes to execute program)***
- *Decompose Time/Program into Components to examine trade-offs*

$\frac{Time}{Program} = \frac{Instruction}{Program} \times \frac{Cycles}{Instruction} \times \frac{Instructions}{Cycle}$

***Instructions/Program:*** *depends on source code/compiler technology/ISA*
***CPI:*** *depends on ISA & microarchitecture*
***Time/Cycle:*** *depends on the microarchitecture & base technology*

- ***single-cycle unpipelined has long cycle time as critical path is long***
- ***pipelined has pipeline latches which shorten critical path** (critical path is longest path of all pipeline stages)*

- ***CPI → throughput metric***
- ***lower CPI → higher throughput***
- *throughput: number of instructions can execute in a given time*

- ***different trade-offs between cycles/instruction and time/cycle in microarchitecture***

- - - 
