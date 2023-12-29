
- - - 

***Classic 5-Stage RISC Pipeline***

- - - 

***RISC Instruction Set: MIPS***

***RISC***
→ *every instruction same width (32bits) fixed-value encodings*

<br>

***Types of Instructions:***
- *Register-Register*
- *Register-Immediate*
- *Conditional Branch*
- *Jump*

<br>

***Register-Register:*** *2 sources registers + destination register + opcode*

***Register-Immediate:*** *source register + destination register + immediate + opcode*
- *add constants to a register (such as when accessing value in a struct in C)*
- *if immediate operand is negative - needs sign extending*

***Conditional Branch:*** *target of branch address needs to fit into encoding - so immediate value is relative address to PC current value*
- *add current PC to immediate operand (scaled by 4 as each instruction is 4 bytes) to give destination address*

***Jump (Unconditional Branch):*** *opcode + target - can branch to any address in the address space*

<br>

***Number of Registers**: determined by the width of the register specifier field (5bits = 2^5 registers)*

<br>

***Trade Off:***
*More Registers → Encoding has more bits for Register Accessing →  Fewer Bits for Immediate Operands, Branch Targets etc.*

- - - 

***Fetch-Decode-Execute Machine***

- *To execute the MIPS instruction set we need a machine that fetches them and does what each instruction says*
- ***Fetch-Decode-Execute***

<br>

- *1. Reads the instruction from memory & increments the PC*
- *2. Take the 2 source registers in the encoded instruction & look them up in the register file - take the immediate operand and sign extend*
- *3. Perform the operation using the ALU*
- *4. If a branch, use that result to update the program counter*
- *5. If a store, store that operand into memory*
- *6. If a load, collect that operand from memory*
- *7. Update the target registers*

- - - 

***Datapath Hardware*

- *fully pipelined*
- *synchronous write & asynchronous read*

<br>

*5 Stages:*
- ***Instruction Fetch***
- ***Instruction Decode***
- ***Register Fetch***
- ***Execute & Address Calculation***
- ***Memory Access**
- ***Writeback***

<br>

- *Use current PC as an address into memory & increment PC by 4*
- *Pass register operands from instructions just fetched to the register file to look up those values in registers*
- *Collect immediate operand from the instruction encoding & sign extend it*
- *Use Multiplexers to select where the operands for the arithmetic operations are going to come from*
- *If a Load Instruction, result from ALU used as address to fetch data from memory*
- *Access Data from Memory & put into a memory access register*
- *Write back, & update any registers from this execution*

- - - 

***Pipeline Buffers***

***Introduce Buffer Registers between the Pipeline Stages***

- *Once Instruction Fetched, Stash the Instruction in a Pipeline Buffer Latch*
- *In Next Instruction Cycle, can Fetch the Next Instruction while the Stashed Instruction is passed to next stage of the pipeline*
- *Then, the register operands that are fetched from the register file can be stashed in the next buffer register, so in the next clock cycle, the second pipeline stage can be operating on the next instruction*
- *an instruction that is fetched progressively moves down the pipeline with each successive clock tick*

<br.

***how is this different from without pipelining?***
→ ***shortens the critical path***

- *without pipelining, datapath is a combinational logic structure, that can execute one instruction per cycle*
- *depth of design, number of gates on the longest path, would determine the clock cycle time - therefore determining the clock rate at which the processor can run
- *as this is a long path - clock rate would be slower*

- *in pipelining, can run at a far higher clock rate, as clock rate determined by the time in which the slowest pipelining stage can be run - much shorter than the entire path*
- *logic depth of each of the pipelining stages controlled - to enable us to run at the highest clock rate*

<br>

***Needs a Control Unit***

- *multiplexers, ALU, and Data Memory need to be configured to know what operation to perform or what data to pass*
- *decoding these instructions (opcodes) tells us what the operation is and can configure accordingly*

- - - 

***Pipelined Instructions***

- *Processor is initially IDLE - at 1st cycle, all pipeline stages are IDLE apart from the 1st one which is fetching instruction*
- *In the next clock cycle, the instruction that we fetched is passed into the instruction decode/register fetch phase, and the instruction fetch phase is available to be used in cycle 2 to fetch instruction 2*
- *Continues in the next cycle, and so on, successive overlap of the sequence of 5 instructions*

<br>

- *At each cycle we fetch a new instruction*
- *And pass the preceding instruction to the next stage of the pipeline*

<br>

- *At cycle 5, the pipeline is fully-occupied, all stages are busy, in parallel*
- *ideally, this would continue indefinitely*
- *There is a **fill phase & drain phase** in which not all pipeline stages are fully occupied*

- - - 

***Speedup of Pipelining***

- *Pipelining **doesn’t help the latency of a single instruction***
- ***latency:*** *time have to wait from the start of the instruction to the end of the instruction*

<br>

- *it helps **reduce the throughput of the entire workload**
- ***throughput**: the number of instruction that can be processed per unit time (instructions per clock cycle)
- *can start an instruction every clock cycle & finish an instruction every cycle - but no one instruction completes any faster*
- *but, can run at a very high clock rate, as reduced logic depth of each pipeline stage*

<br>

- ***pipeline rate limited by slowest pipeline stage (critical path)***
- *potential speedup = number of pipe stages*

<br>

- *speedup comes from parallelism - for free - no new hardware*
- *don’t have to add any additional logic to add pipeline stages, but there is a slight cost in adding the latches (transistor counts, energy etc.)*
- *so cannot ‘overpipeline’ - clock rate would become dominated in time spent in the latches*

<br>

- ***speedup reduced by: unbalanced pipeline stage lengths, fill & drain times***

- - - 
