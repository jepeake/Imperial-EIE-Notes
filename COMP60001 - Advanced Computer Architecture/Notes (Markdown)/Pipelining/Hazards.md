
- - - 

***Hazards***

- - - 

***Hazards:*** 
→ *prevent next instruction from executing during its designated clock cycle (preventing full throughput)*

<br>

***Structural Hazard:*** *the hardware cannot support the combination of instructions*
*(an instruction in the pipeline may need a resource being used by another instruction in the pipeline)*

***Data Hazards:*** *instruction depends on result of a prior instruction still in the pipeline*
*(previous instruction has not been completed yet) (dependence on a data value)*

***Control Hazard:*** *caused by delay between fetching instructions & decisions about changes in control flow (branches & jumps)*
*(fetching next instruction even though have not decode the previous instruction yet - which may be a branch/jump)*
*(dependence on the next instruction’s address)*

<br>

→ ***handling hazards generally introduces a bubble into the pipeline and reduces ideal CPI > 1***

- - - 

***Pipeline CPI***

***CPI***
→ *a measure of throughput*

<br>

- *CPI = 1 → pipeline fully utilised*
- *stall → increases CPI → increases throughput latency*

- - - 

***Structural Hazard***

- *imagine a processor has only one memory for both instructions & data*
- *two different stages may need to access the memory at the same time*
- *e.g. the load instruction & instruction 3 here - want to fetch instruction 3 from the same memory that the load instruction is loading value from*

<br>

- *structural hazard caused by **simultaneous access***
- *every time you do a load instruction you cause a simultaneous instruction to stall*

<br>

- ***stall*** *means that instruction 3 must be delayed by one cycle*
- *causes a pipeline* ***bubble***
- ***bubble*** *propagates through the pipeline*

<br>

→ ***simultaneous access of memory (structural hazard) causes a bubble in the pipeline - where one or more stages of the pipeline are idle as the next instruction in the sequence cannot proceed immediately***

- - -

***Resolving Structural Hazards:***

***stalling:***
- → *can resolve in hardware by stalling newer instructions until older instructions have finished with the resource (inserting bubbles)*

***adding more hardware to design:***
- *→ can always be resolved by adding more hardware to the design (in above example → adding a separate instruction & data memory to the machine)*

- - - 

***Data Hazard***

***example:***

- *r1 computed in first instruction & then used in subsequent instructions*
- *r1 value only written to the register file only in the write-back stage*
- *but needed in the 2nd stage of the 2nd, 3rd, 4th, and 5th instructions to be fetched*

<br>

- *for 5th instruction - okay as register file only accessed in clock cycle after the value has changed*
- *for 4th instruction - still okay as data fetched from register file in second half of clock cycle and written in the first half of the same cycle*
- *but not okay - for the second and third instructions*
- *can solve with **forwarding***

- - - 

***Type of Data Hazards***

- ***Data Dependence (True Dependence)*** → *Read After Write (RAW) Hazard*
- ***Anti Dependence*** → *Write After Read (WAR) Hazard*
- ***Output Dependence*** → *Write After Write (WAW) Hazard*

- - - 

***Resolving Data Hazards***

***Stalling:***
→ *wait for hazard to clear by holding dependent instruction in the issue stage*

***Forwarding:***
→ *resolve hazard earlier by bypassing value as soon as available*

***Speculate:***
→ *guess on value & correct if wrong*

- - - 

***Forwarding***

→ ***forwarding values to where it is needed in the next instruction - to resolve data dependencies & prevent stalling***

***example:***

- *result of add instruction computed at the end of the 3rd stage - so can **forward** that value to where it is needed in the next instruction*
- *in this case - forwarding from the output of the ALU to the input of the ALU for the next instruction*
- *can do so for other instructions - but may need to delay the values before forwarding*

- - - 

***Stalling vs Forwarding***

***given a RAW Hazard:***

<br>

***Stalling:***
- *stalling stalls for two cycles - inserting bubbles in pipeline*
- *when first instruction is in WB stage, writing is happening in the first half cycle, second half is doing reads, so second instruction’s D stage can take value of x1 from register file*

<br>

***Forwarding:***
- *forwarding forwards without waiting for WB stage*
- ***no bubbles***

- - - 

***HW Change for Forwarding***

***changes:***
- *add forwarding (bypass) paths*
- *add multiplexors to select where ALU operand should come from*
- *determine MUX Control in ID stage*
- *if source register is the target of an instruction that will not be in WB in time - forward*

<br>

- *expand multiplexors to include forwarding paths*
- *if value needed not yet written back to register - multiplexor must select a forwarding path*

<br>

- *wires from ALU output to mux - for forwarding to next instruction*
- *wires from data memory output to mux - for forwarding to the next-but-one instruction*

<br>

- *multiplexors controlled by **decode unit***
- *extended to control a bigger multiplexor for forwarding - to select values for blue wires*
- ***decode** - keeps track of which registers are going to be updated by instructions that have not been completed*
- ***at decode point** - known where operands are coming from, what other instructions are in flight, & what registers they are going to write to*
- *so can select configuration for multiplexors with this information*

- - - 

***Forwarding Builds the Data Flow Graph***

***data flow graph**: represents the flow of data within a system → used to visualise & analyse the dependencies between different instructions*

<br>

- *values passed directly from the output of the ALU to the input - via forwarding wires that are dynamically configured by the decode unit*
- *values going round & round in the arithmetic unit for this sequence of operations*

<br>

***Multiple ALUs:***

- *execute multiple instructions per clock cycle - with multiple ALUs*
- *needs a dynamic construction of a data flow graph for the allocation of a given sequence of operations onto the ALUs over a sequence of time*
- (*which ALU will execute each instruction & how the data is forwarded*)
- *resource allocation. scheduling, & configuring forwarding paths*

- - - 

***Load-to-Use Stall:***

→ ***data hazard in which the data being loaded by a load instruction is not yet available when needed by the next instruction***

→ *must stall*

→ *cannot be resolved by forwarding*

***example:***

- *here - in load instruction - value r1 is only available at the end of the data memory - as the ALU comes before to calculate offset*
- *but sub instruction needs the value r1 in the execute stage (before the value can be read from data memory)*
- *cannot forward backwards in time*
- ***leads to a pipeline stall due to data hazard***

<br>

***load-to-use stall:***
- *need to configure decode logic to implement pipeline stalls as well as configuring forwarding paths*

- *at decode time - notices that instruction needs r1 that will not be available until the next cycle - decoder stalls that instruction for one cycle so that forwarding can be done - and then carried out forwarding*

- - - 

***Software Scheduling to Avoid Load-to-Use Stall***

***implemented by compiler***

- *simple compiler will compile each statement separately*
- *putting a stall between load & add instructions to avoid data hazard*
- *sequence leads to 2 stalls*

<br>

- *more advanced compiler - would know stall behaviour of instructions & reschedule instructions to fill stalls*
- *stalls - represent missed instruction execution opportunities*
- ***instruction scheduling** - tries to use these opportunities*

- - - 

***Control Hazards***

***What is needed to calculate the next PC?***

- *Unconditional Jump → See Opcode, Add Offset to PC*
- *Jump Registers → See Opcode, Read Register Value + Offset*
- *Conditional Branch → See Opcode, Jump to PC + Offset if Register Condition True*
- *Other Instruction → See Opcode, PC + 4*

<br>

- ***Information: Opcode, PC, Offset, Register Value***

- - - 

***Control Hazards on Branches***

***example:***

- *start decoding branch instruction in the 2nd cycle & fetching 2nd instruction - but, branch if equal may be true and need to fetch instruction at PC+36 rather than PC+4*

<br>

- *could do the comparison in the ALU - and discover the branch outcome at the ALU output - but, **this is still too late & would cause a stall of 3 cycles***

- - - 

***Unconditional Jumps***

- *Unconditional Jump - JAL (PC Relative Jump)*

- *See the Opcode after the Instruction Register + see Jump*
- *→ only know it’s a Jump Instruction in the Decode Stage*
- *next instruction is already being Fetched in Fetch Stage*
- *→ do not want that to be executed*

<br>

- *kill the next instruction*
- *select the PC Jump Address & Jump to new location*

<br>

- *kill bit → inserts a bubble & flushes the pipeline*
- *bubble: hardware mechanism to stall the execution*
- *NOP: software instruction added into sequence*
- *same effect as not changing the hardware state → but NOP adds extra instruction to be executed*
- *can give different hardware behaviours*

- - - 

***Pipelining for Unconditional Jumps***

- *pipeline execution of unconditional PC-Relative Jumps*

<br>

- *when we decode the jump & realise jump to be taken - stall the following instruction by inserting a bubble*
- *changing control flow*

- - - 

***Conditional Branches***

- *need Opcode, PC, Register Value, Offset*

<br>

- *cannot decide until execute stage whether to take branch (is condition met)*

- - - 

***Pipelining for Conditional Branches***

- *need two bubbles - don’t know whether branch is resolved until execution stage, by that time, two more instructions in-flight, one in F stage, and one in D stage*
- *both changing the control flow*

- - - 

***Pipelining for Jump Register***

- *does not rely on condition - but only get register value in execute stage*
- *two bubbles*
- *jumps to a register value rather than PC value*

- - - 

***Why Instruction may not be Dispatched Every Cycle in Classic 5-Stage Pipeline (CPI > 1)***

- ***goal: CPI = 1 → not always possible***

<br>

***full forwarding may be too expensive to implement:***
- → *typically all frequently used paths are provided*
- → *some infrequently used forwarding paths may increase cycle time & counteract the benefit of reducing CPI*

<br>

***loads have two-cycle latency:***
- → *instruction after load cannot used load result **(load-to-use stall)***
- → *load values are not available until after the memory read → leads to 2-cycle latency*

<br>

***Jumps/Conditional Branches may cause bubbles:***
- → *kill following instructions if no delay slots*

- - - 

***Early Branch Prediction***

- ***solution to control hazards for branches***

<br>

- *adds extra hardware to the decode stage - to **determine branch direction & target earlier***
- *still have a one-cycle delay (better than 2-cycle delay), as have to fetch and start executing the next instruction*
- *if the branch is actually taken, block the MEM and WB stages and fetch the correct instruction*

<br>

- *extra hardware adds the current PC value to the immediate operand*
- *checks the branch outcome by checking if the branch operands are equal (BEQ instruction)*
- *if they are equal - multiplexer chooses the branch target value for next PC*

<br>

- *move branch determination as early as possible in the pipeline*
- *still some delay - but **minimised***

- - - 

***Simultaneous Multithreading***

- ***if there was no stalls - could finish one instruction per cycle***
- *if we had no hazards we could do it without forwarding - & decode/control would be simpler*

<br>

***solution: executing two threads concurrently***

<br>

- *maintain two PCs*
- *even cycle - fetch from PC0*
- *odd cycle - fetch from PC1*
- *odd-even interleave (alternating)*
- *thread 0 reads & writes thread-0 registers (no register dependencies between threads - disjoint register sets)*
- *no register to register hazards between adjacent pipeline stages*

<br>

- ***greatly reduces stalls & reduces complexity - may be able to run at a higher clock rate***

- - - 

