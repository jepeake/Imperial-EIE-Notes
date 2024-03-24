
- - - 

***Dynamic Scheduling with Tomasulo***

- - - 

***Dynamic Scheduling, Out-of-Order Execution, Register Renaming***

- - - 

***Instruction Parallelism:***

***Bypassing Stalls:*** *instructions behind a stall can be allowed to continue provided data dependence/hazards allow*

- *when an instruction stalls save the state of that instruction*
- *instructions are issues in order, have dependencies analysed, and can then be executed out-of-order*
- *when operands are available, allow execution of the stalled instructions to continue*


- *add instruction depends on the divide instruction - subtract instruction does not depend of either of them - want subtract instruction to proceed while add instruction waits for execution of divide instruction*

- *when an instruction is fetched - may be able to execute the instruction straight away or may need to **shelf that instruction until the operands are available***
- *for example: add instruction must remain on shelf until the operand F0 is available*
- *may also have to wait for functional units to be available

- ***lifecycle of instruction:** instruction is issued, sometime later will be begins execution, then completes execution and then passing operands to instructions that depend on it*

- *instructions still need to be **issued in order** (in a dynamically scheduled pipeline)*
- *but can have **out-of-order execution** & **out-of-order completion***

- - - 

***Data Dependencies & Hazards:***

*what constrains execution order?*

***True Dependence/RAW:***

- *instruction J is data dependent on instruction I*
- *J tries to read operand before I writes it*
- *or J is data dependent on instruction K which is data dependent on instruction I*

- *this is a **true dependence***
- ***Read After Write (RAW) Hazard:*** *if true dependence causes a hazard in the pipeline*

- - - 

***Name Dependencies:***

***Name Dependence:*** *when two instructions use the same register or memory location, called a name, but no flow of data between the instructions associated with that name*

- - - 

***Anti-Dependence/WAR:***


- *instruction J writes operand before instruction I reads it (overwriting)*
- *here: I will read the incorrect value of r1 as has been overwritten*
- *anti-dependence results from reuse of r1*

- ***Write After Read (WAR) Hazard:*** *if the anti-dependence caused a hazard in the pipeline*

- - - 

***Output Dependence/WAW:***


- *instruction J writes operand before instruction I writes it*
- *here: mul depends on r1, but will get the value of r1 from I rather than J*
- *output dependence results from reuse of r1*

- ***Write After Write (WAW) Hazard:*** *if the anti-dependence caused a hazard in the pipeline*

- - - 

***Dynamic Scheduling:***

- *rename decode stage to issue*
- ***issue:*** *decode instructions, check for structural hazards (is there a functional unit available, is there a shelf can put instruction on if necessary)*

- ***read operands:*** *step in which operands are read (if not read in issue stage), wait until there are no data hazards*
- *still need to check operands in issue stage - to check that they are ready*
- *done by looking at registers & checking if marked as being used by an in-flight instruction*
- *if operands ready, collect them at issue stage - otherwise, discover where they will come from and make them available in the RO step*

- *instructions are issued in order but may stall at the read operands stage while others execute*

- - - 

***Tomasulo’s Algorithm:***

- *decouples the dispatch of instructions from the availability of their operands → allowing out-of-order execution*


- ***Issue**: collects operands from instructions source registers, consults registers, & allocates instruction to a reservation station (shelf)*

- ***Reservation Station**: shelf where an instruction sits when waiting to use a functional unit & not ready to go yet - holds opcode and two operands*
- *Both operands need to be present before functional unit can proceed*
- *May have to wait for operands to be computed*

- *registers can hold a value + a tag*
- ***Tag**: if tag NULL, value is valid (no instruction currently in execution in machine producing a value for that register*
- *if there is an instruction in execution that produces a value for that register, tag value will be the id of the function unit that will produce the value for that register*

- - - 

***steps**:*

- *when executing an instruction, the issue unit takes the source operands for the instruction*
- *a functional unit is allocated to the instruction*
- *if the value for each source register is valid (tag is NULL), the value in the register will be copied to the operand in the allocated functional unit*
- *the destination register will have it’s tag updated with the id of the allocated functional unit - where the result is going to come from*
- *when another instruction has the same destination register - it will overwrite the tag at issue time


- *at issue time, if the value of the source register is not valid (tag is not NULL), the tag is copied to the operand of the allocated functional unit*
- *i.e. - if that source register is depending on instructions that are currently in flight, the functional unit that it is dependent on will be copied to the operand rather than the value itself (as this is the value that has been put in the tag field)*

- *when an instruction finishes - the common data bus connects all of the functional unit to all of the registers, and all of the reservation stations*
- *the registers monitor the common data bus, if the registers current value is not valid, the tag indicates which functional unit is going to produce its result, so the register can monitor for any values that carry that tag*
- *when a functional unit finishes executing, it broadcasts onto the common data bus with a tag saying that this result is from that functional unit*
- *so the register that depends on that value can wait until it sees that tag, and pick the value up*
- *updating the destination register with the executed value*

- *the reservation station also holds in the operand field the tag of the value that it is waiting for, so it can also monitor the common data bus and take that value*

- *when the execution finishes, the result gets passed to the dependent functional unit directly*

- - - 

- ***common data bus**: dynamically managed to forward the result from functional units that produce it to functional units that depend on it - without going through registers*

 - ***registers have tags overwritten with the functional unit they depend on** - so even if an instruction execution finishes after the value has been overwritten, that value will just be ignored by the register monitoring the data bus*

- ***register is more of a tag that connects dependencies** - rather than storing a value*
- *at issue time, we are **dynamically decoding the dependency structure of the program** & arranging the forwarding wiring based upon this* 

****

***Read After Write Hazard:***

- *Tomasulo solves the issues faced with read-after-write (RAW), write-after-read (WAR), & write-after-write (WAW) hazards*

***RAW:***
- *for read-after-write (RAW), the first write instruction at issue time will put the value of the functional unit it is using into the tag field of the destination register it is writing to*
- *the read instruction will then copy the tag of that register it is reading from, given the value is invalid, which will be the functional unit producing the value from the write instruction → then putting that tag as the source operand of the reservation station it is using*
- *so once the write instruction has executed, it will pass the value to the reservation station of the read instruction directly - removing the read-after-write error*

***WAR:***
- *for write-after-read (WAR), the read instruction will either copy the value from the register to its operand if valid, or copy the tag of the register if invalid*
- *the write instruction, will then update the tag of the destination register that it is writing to (and the other instruction is reading from)*
- *as the read instruction has the value itself, or the tag of the functional unit it is dependent on in its reservation station, it does not matter if the write executes before the value is fully read, the value was already in its reservation station before the issue time of the write instruction*

***WAW:***
- *for write-after-write (WAW), the first write instruction at issue time will put the value of the functional unit it is using into the tag field of the destination register it is writing to*
- *the second write instruction (writing to the same destination register) will put the value of the functional unit it is using into the tag field of the same destination register (overwriting the value put by the first write instruction)*
- *this way → any following instructions reading the value from that register will get the tag of the functional unit of the second write instruction*
- *this eliminates any possible WAW error*

- - - 

***Stages of Tomasulo Algorithm:***

- ***Issue:*** *get instruction from FP Op Queue*
- *if reservation station free (no structural hazard), control issues instruction & sends operands (renames registers → by effectively renaming the destination register to the reservation stations)*

- ***Execute:*** *operate on operands (execute)*
- *when both operands ready, then execute*
- *if not ready, watch common data bus for result*

- ***Write Result:*** *Finish Execution (WB)*
- *write on common data bus to all awaiting units, mark reservation stations available*

*two buses:*
- *normal data bus: data + destination (used at issue)*
- *common data bus: data + source (used at WB)

- - - 

***Register Renaming***

***register renaming → used to eliminate the false data dependencies that arise due to the reuse of architectural registers by successive instructions***

- *by eliminating these dependencies, the processor can execute more instructions in parallel (bypassing stalls) & improving instruction-level parallelism*

- *Tomasulo’s Algorithm uses register renaming by effectively **renaming the destination register of an instruction to the reservation station used by that instruction***

- - - 

***Tomasulo Drawbacks:***

- ***Complexity***
- *many associated stores at high speed - multiple comparators monitoring bus in parallel*

- ***Performance Limited by Common Data Bus***
- *each common data bus must go to multiple functional units, high capacitance, high wiring density*
- *number of functional units that can complete per cycle limited to one*
- *two functional units may complete at same time - then need multiple common data buses, more functional unit logic etc.*

- ***no precise interrupts***

- - - 

***Why can Tomasulo Overlap Iterations of Loops?***

→ ***iterate whole loop iterations in parallel with each other***

- *would have same instructions running in parallel referring to same registers*
- *need register renaming (as Tomasulo allows with Reservation Stations)*

- ***Register Renaming: multiple iterations use different physical destinations for registers (dynamic loop unrolling)***

- *Reservation Stations: permit instruction issue to advance past integer control flow operations, & buffer old values of registers (avoiding WAR stall)*

- - - 

***Summary:***

- *Tomasulo overcomes WAR and WAW hazards by **dynamically allocating operands to reservation stations at issue time** (decoupling operands from register files) - register files can be reused while reservation stations provide pathway for those instructions to complete correctly*

- *Tomasulo’s CDB is a kind of forwarding path - that routes operands from completing FUs to where they are needed*

- *Tomasulo’s Scheme relies on **associative tag matching** (comparator needed at every register & every reservation station)*

- *Tomasulo’s Scheme **enables multiple FUs to operate in parallel** (even across loop iterations) - allows to dynamically adapt to unexpected execution orders (such as cache misses)*

- - -


