
- - - 

***Dynamic Scheduling with Precise Exceptions***

- - - 

*Dynamic Scheduling, Out-of-Order Execution, Register Renaming **with Speculative Execution***

- - - 

***Handling Precise Interrupts in Tomasulo***

*Tomasulo had:*
- *in-order issue*
- *out-of-order execution*
- *out-of-order completion*

- *out-of-order completion → does not allow precise exceptions
- *cannot find a precise breakpoint in the instruction stream*
- *do not have a consistent program state*
- *e.g. suppose we have a page fault, or a divide-by-zero exception*

- *the same issue is found with branch speculation*
- *if we speculate that a branch is not taken, executing instructions, & then the branch is taken (misprediction)*
- *we do not have a consistent program state to return to in Tomasulo (to undo the speculatively executed instructions)*

- ***solution: add a commit stage***
- *commits the state in issue order*

- - - - 

***Four Steps of the Speculative Tomasulo Algorithm***

- *added a commit stage*
- *maintains a reorder buffer structure*
- *committing instructions only at the point we know they have been correctly executed*
- *up to the commit point → can throw previous instructions array (if mispredicted branch)*

- - - 

***Tomasulo with Reorder Buffer***

- *ROBs introduced to manage the results of out-of-order executed instructions to ensure in-order commit of instructions*


- *added Reorder Buffer, Commit Unit, & Commit-side Registers*


- ***Reorder Buffer: a queue/circular buffer/FIFO***
- *when an instruction is issued → also allocated a reorder buffer entry (pushed onto queue)*
- *reorder buffer monitors common data bus for any instruction that completes & marks completed instructions*
- *when reorder buffer entry marked as complete → eligible for commit*

- ***Commit Unit: processes instructions in issue order***
- *instructions are issued in order*
- *may sit on a shelf (in reservation station) for some time*
- *may execute & complete out of order*
- *→ then processed, in order by commit unit*
- *takes the instructions from the ROB, when the value at the head of the queue is marked as complete*
- *updating the commit side values of the registers*

- ***Commit-side Registers***
- *only externally visible state of the machine*
- *maintains a consistent program state (which can be returned to in the case of an exception/mispredicted branch)*

- *machine split into speculative side & commit side*

***Issue:***
- *as before, but ROB also allocated*
- *one ROB entry for each instruction*
- *ROB entry holds destination register + either the result value (instruction completed)/tag where it will come from (instruction in-flight)*

***Write-Back:***
- *as before (broadcasting to common data bus), but ROB entry with matching tag is also updated (ROB monitors the common data bus)*
- *ROB entry for instruction 1 holds value for F0*
- *ROB entry for instruction 3 holds another value for F0*
- *values for F0 are overwritten in the issue-side registers, but ROB maintains the different values of F0 → before they are committed*

***Commit:***
- *commit unit processes ROB entries in issue order*
- *each instruction waits in turn and commits when its operands are completed (instruction are committed when they are at the head of the queue & have completed)*
- *committed registers updated with values from ROB*
- *commit-side F0 is updated first with result from MUL1 & then result from MUL2*

- - - 

***Registers***

*now have two sets of registers:*
- ***issue-side registers***
- ***commit-side registers***

*issue-side registers: update speculatively*

*commit-side registers: update when speculation resolves*
- *maintain a consistent program state (a precise point in the program execution where all preceding instructions have completed & all following instructions are in flight)*
- *allows us to return to a precise point/consistent state given a exception/page fault/mispredicted branch*

- - - -

***Conditional Branch***

*→ add a conditional branch instruction to the issued instructions*
- *predict the conditional branch as not taken → fall through to instructions 4 & 5*

- *as we progressively issue the instructions, we fill up the reorder buffer with the instructions*

- *assume we predicted the branch as not taken*

***branch misprediction:***
- *when the branch (predicted not taken) reaches the head of the commit queue → could be a **misprediction** (branch was actually taken)*
- *→ all issued but not committed instructions (after the branch) are erroneous*
- *→ trash all of the ROB entries*
- *→ use commit-side registers to reset values of issue-side registers (F0, F1, F2, F3) → reinstating the register state at conditional branch point*
- *→ correct branch target instructions are fetched & issued (refilling the machine & restarting execution)*

- *assuming functional units are not stopped*
- *→ they eventually complete & broadcast their values, but no issue-side registers with matching tags (issue-side register updated with values from commit-side registers), or ROB entries awaiting them (ROB entries been trashed)*
- *reservation stations of uncompleted speculatively-executed instructions cannot be re-used until the functional unit has completed*

- *branch misprediction penalty → takes some time to refill & restart the machine will useful work*

- - - -

- *Tomasulo + ROBs → does not roll-back as soon as the branch is found to be mispredicted (waits for branch instruction to reach the head of the commit queue)*

- *Processor Architects looking for ways to give a more immediate roll-back*
- *→ e.g. fetching useful instructions as soon as the branch condition is calculated*

- - - -

***What if a Second Conditional Branch is encountered, before the outcome of the first is resolved?***

- *→ can speculate on the outcome of both branches*

- *assume predict first branch to be not taken*
- *when it reaches head of the commit queue, branch should be taken → misprediction*
- *all ROBs will be trashed → including the second conditional branch instruction*

- - - 

***Store Instructions***

- *stores are buffered in the ROB → & committed only when the instruction is committed*
- *a load instruction can be issued while several stores are uncommitted → load will not access the correct data as it has not been committed to memory yet*
- *→ need to ensure load instructions get the correct data*

- - - -

***Stores & Loads with Speculation***

*→ need to ensure stores are not sent to memory until the store instruction is committed*
*→ need to ensure load instructions get the correct data*

***solution:***
*if/when the addresses of a load & all preceding uncommitted stores are known:*
- *if none of the store addresses match the load → load can proceed*
- *if the address of the load matches the address of an uncommitted store → forward the store’s data to the load*

- *add load unit & store buffer*
- *store buffer collects the uncommitted store instructions*
- *when a load instruction → waits in the load unit until its address & all preceding store addresses are known*
- *when load address & all preceding store addresses are known → proceed load instruction if no match, & forward store’s data to load if match*

- - - 

***Store-to-Load Forwarding***

- *uncommitted store & load instructions → want data to be forwarded directly from store to load (without having to wait for either instruction to be committed)*

- *the Tomasulo Scheme derives dependencies between register-register instructions*
- *the registers in use are always known at issue time*

- *loads & stores → use computed addresses*
- *which may or may not be known at issue time*

*consider:*

- *stores F0 into memory at address R3*
- *loads an address from memory into R2*
- *stores F1 into memory at address R2*
- *loads F2 from memory address R3*

- *instructions 1 & 4 ready (in store buffer & load unit) before instructions 2 & 3 have completed*
- *could forward the value from instruction 1 (stores to address R3) to instruction 4 (loads from address R3)*
- *however, if address R1 = R3 → F1 would be stored at address R3*
- *→ therefore, forwarding the value from instruction 1 to 4 would miss the value stored to R3 in instruction 3*

***solutions***
- *wait → until all preceding store addresses are known*
- *speculate (considerable performance advantage)*
- *add a forwarding predictor, to improve speculation*

***memory dependence speculation:***
- *allow a load to proceed (by either forwarding a value from some store whose value is known, or proceed by going to memory) before we know for sure which, if any, prior uncommitted store instruction writes to its address*
- *(we may know the load’s address but not all of the addresses of the prior stores, or we may not know the load’s address)*
- *use a predictor to decide whether to allow a proceed or not*

- - - 

***Design Alternatives for Out-of-Order Processor Architectures***

- *Larger Design Space Exists for Processor Architecture*

- *Register Update Units (RUU) vs Re-order Buffers (ROB)*
- *Realisation in Pentium III & Pentium 4*

***simplescalar simulator:***

- *simplescalar: simulation software of a processor microarchitecture*
- *simulates a multi-issue out-of-order design with speculative execution*
- *many aspects of the design can be controlled by parameters*
- *used a RUU*
- ***RUU: combines ROB & Reservation Stations into a single pool***

***Fetch:***
- *fetches instructions from memory*
- *obtaining the instructions that need to be executed for the program*

***I-Cache:***
- *instruction cache that stores a subset of the main memory’s instructions*
- *speeds up the process of fetching instructions by storing recently accessed/frequently accessed instructions*

***Dispatch/Issue:*** 
- *once an instruction is fetched, it is dispatched/sent to appropriate units for execution*
- *involves decoding the instruction & figuring out what resources the instruction needs (registers, execution units…)*

***Scheduler:*** 
- *determines the order in which the dispatched instructions will be executed*
- *can rearrange the order of instruction execution based on the data dependency & availability of execution units*
- *optimises for maximum throughput*

***Memory Scheduler:***
- *dedicated scheduler for memory operations (load/store scheduling/forwarding…)*

***Exec/Mem:***
- *execution/memory → where scheduled instructions are executed*
- *if the instruction involves a memory operation, it will be handled in this stage*

***D-Cache:***
- *data cache*
- *stores a subset of the main memory’s data to speed up data accesses*

***D-TLB:***
- *data translation lookaside buffer*
- *cache that store recent virtual address to physical address translations (rather than traversing the whole page table)*

***Writeback:***
- *may need to write back to a destination register or memory*

***Commit/Retire:***
- *if any speculative changes made during execution, finalised here*
- *if the instruction cannot be successfully committed (e.g. mispredicted branch) → speculative work is discarded*

- - - 

***Register Update Unit (RUU)***

- *state from commit-side registers can be transferred in parallel to issue-side registers (i.e. on a branch misprediction) to keep a consistent state*

- *instructions are issued into RRU (which unifies ROB & Reservation Stations)*
- *when an instruction is ready to be executed (operands ready & functional unit ready to execute it)*
- *→ dispatcher selects an instruction & dispatches it to a functional unit*

*advantage:*
- *as we have a common pool of instructions in the RUU → do not commit to which functional unit we are going to use*
- *in Tomasulo, we commit to the functional unit as we have to select a particular reservation station (one reservation station per functional unit)*
- *RRU defers this decision to dispatch time*

- *in Tomasulo, tags belong to the Reservation Stations/Functional Units executing that instruction*
- *with RUU → tags are RUU indexes*
- *RUU being managed as a circular buffer (like ROB) & tag used as index to that buffer*
- *when an instruction is dispatched → also include tag*
- *when the instruction completes & is broadcast to the common data bus → tag used to update issue-side registers & used to index the RUU to update the ROB entry for the instruction in question*

- *commit unit waits for the instruction at the head of the RUU queue to be completed & commits it to the commit-side registers*

- *in Tomasulo: all of the issue-side registers, reservation stations, & ROB entries have to monitor the common data bus for the tag (in case it matches the data that the unit is waiting for)*
- *in RUU: only the issue-side registers & RUU have to monitor the bus*

- *RUU can be indexed as a RAM → considerable efficiency advantage (do not need a comparator for every entry)*

- - - 

***RUU vs ROB***

- *in Tomasulo + ROB → registers & ROB entries have a tag*
- *every register, ROB entry, & reservation station needs a comparator to monitor the CDB*

- *with RUU → the tags are the ROB entry numbers*
- *so the ROB is indexed by the tag on the CDB (ROB thought of as a set of registers)*
- *at instruction issue time → ROB entry allocated for instruction in question*
- *→ dynamic remapping of registers in instruction set to ROB entries*
- *the ROB entries are a **renamed register** for the instruction’s result*

- - - 

***Pentium III & Pentium 4 (NetBurst)***

*Pentium III:*
- *to avoid copying register contents around → rather than storing register values in issue-side registers → RAT*
- *Register Alias Table (RAT) keeps track of the latest alias for logical registers → keeping pointers to ROB entries*
- *issue-side register may map to ROB entry (uncommitted), or to a RRF entry (committed)*
- *commit-side register file → RRF (retired register file)*
- *if issue-side register does not point to any uncommitted instruction, will point to RRF*
- *when instructions are committed → switch pointer from ROB to RRF (where the value has been committed to)*
- *data is copied from the ROB to the RRF*

*Pentium 4*
- *separate the allocation of physical registers*
- *when instruction is committed → data is not copied from ROB to RRF → stays in RF & Retirement RAT pointer points to location in RF*
- *committed state of the machine represented by physical registers in RF that are mapped to by logical registers in Retirement RAT*
- *need to know for each register in RF → which ROB entry corresponds to value which will be assigned to the register*
- *→ as instructions issued, allocate a ROB entry & RF register, RF pointer to ROB entry which will produce its value*
- *ROB only contains status fields*
- *when an instruction complete, RF entry is updated, ROB entry is marked as completed, & commit unit processes ROB entries in sequence & updates Retirement RAT to point to updated retired register*
- *ROB can be managed as a queue (in Pentium 4 → RF can be allocated in dynamic way, pointing Frontend RAT to dynamically allocated register)*
- *registers are not freed → when an instruction commits, Retirement RAT points to the register*
- *→ later, when value is overwritten → RF entry can be reallocated*

- - - 

***Pentium 4 NetBurst Microarchitecture***

- *instructions (which vary in length) are fetched & then decoded into microops*
- *simple instructions can be translated into one microop*
- *more complicated instructions may take more microops*
- *trace cache (instruction cache) caches the decoded microops & tries to cache predicted taken chains of microops*
- *trace cache BTB (branch predictor) tries to predict which elements of the trace cache will be the branch target*
- *when an instruction is issued → dynamically allocated register from RF*
- *→ dispatch to memory side of machine or integer/FP queue*
- *instructions then dispatched (bypassed) to available functional units*

- - - 

- *Pentium 4 runs at very high clock rate & has many pipeline stages*
- *Pentium III has slower clock rate & less pipeline stages (does more in each pipeline stage)*
- *unclear which is faster (which branch misprediction penalty is less)*

- *pipeline stages → counted by how many clock cycles a branch misprediction takes*
- *branch misprediction penalty → number of pipeline stages * clock cycle time*

- - - 

***Multiple Instructions per Clock Cycle***

- *without optimisation → compiler is not using registers*
- *all operations happening with pointers to memory*
- *→ speculative execution techniques will not work*
- *speculative execution → relies on registers to determine the data dependency structure of a program*

*with optimisation:*
- *register allocation*


- *four instructions in the loop & only references registers*
- *1.02 clock cycles per loop iteration (4 instructions executed in parallel per cycle)*
- *multiple iterations of the loop are being executed in parallel at the same time*
- *4 instructions are in flight, can be from multiple iterations of the loop*

- - - - 

***Summary - Dynamic Scheduling***

***advantages:***
- *reduced dependence on compile-time instruction scheduling (& compiler knowledge of hardware → compiler can generate code without knowing the particular hardware/processor code will be running on)*
- *handles dynamic stalls due to cache misses (which compiler cannot know about)*
- *register renaming frees architecture from constraints of the instruction set (mapping limited number of registers in instruction set to a much larger set of physical registers in the machine, see NetBurst → explicitly managed by dynamic register map allocation at issue time)*

***disadvantages:***
- *increases pipeline depth & therefore misprediction latency
- *increases power consumption & area (→ more hardware)*
- *increased complexity & risk of design error (large design space → requires more verification & validation*
- *hard to predict performance & hard to optimise count*

- - - 

