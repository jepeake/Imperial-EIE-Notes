
- - - 

***Static Scheduling for Instruction Level Parallelism***

- - - 

***Overview***

*Dynamic Scheduling*
→ *Out of Order Execution*
→ *exploiting instruction-level parallelism in hardware
→ ***hardware speculation*

*Static Scheduling*
→ *shifts complexity to compiler*
→ *changing instruction set architecture*
→ ***software speculation*

- *VLIW (Very Long Instruction Word)*
- *EPIC (Explicitly Parallel Instruction Computer)*

- - - 

***Example***

*adding scalar to a vector:*

*introduces many stalls in execution:*

*rewriting code to minimise stalls:*

→ *swapping BNEX & S.D by changing address of S.D*

*improving by loop unrolling:*

→ *4 copies of loop body*
→ *one copy of increment & test*
→ *adjust register indirect loads using offsets*

*register renaming:*

→ *each loop body uses own registers → removing RAW hazards*

*unrolled loop that minimises stalls:*

→ *3.5 cycles per iteration*

*software pipelining:*

- - - 

***Software Pipelining***

→ ***static overlapping of loop bodies***

→ *if iterations from loops are independent → then can get more ILP by taking instructions from different iterations*

***software pipelining → reorganises loops so that each iteration is made from instructions chosen from different iterations of the original loop***

→ *Tomasulo in Software*

- → *5 cycles per Iteration*

*including drain & fill phases:*

- - - 

***Changing Instruction Set***

- ***superscalar processors** →  decide on the fly how many instructions to issue in each clock cycle (capable of issuing multiple instructions per cycle)*
*problem:*
- *→ have to check for dependencies between all n pairs of instruction in a potential parallel instruction issue packet*
- → *hardware complexity of figuring out the number of instructions to issue is $O(n^2)$*
- → *leads to multiple pipeline stages between fetch & issue*

*solution*
- → *allow compiler to schedule instruction level parallelism explicitly*
- → *format the instructions into a potential issue packet (& hardware does not need to check explicitly for dependencies)*

- - - 

***VLIW***

- ***VLIW →*** *each instruction has explicit coding for multiple operations*

- *all operations the compiler puts into long instruction word independent → can be issued & executed in parallel*
- → *need compiling technique that schedules across several branches (simplifies hardware & complicates compiler)*

*examples:*

*Transmeta:*
- *four functional units in machine*
- *each 128-bit instruction has four fields telling corresponding unit what to do in this cycle*
- *x86 compatible → hardware contained software that takes x86 instructions & translates into VLIW instructions & optimised scheduling to achieve higher performance*

*Texas Instruments:*
- *8 functional units*
- *2 separate RFs (4 functional units each)*
- *can execute 8 instructions per cycle→  if fits in all 8 functional units*

- - - 

***Loop Unrolling in VLIW***

*example:*

*on a VLIW machine:*

- *each instruction packet has 5 entries*
- → *issuing instruction in parallel executes loop in fewer clock cycles*
- → *parallel instructions must be independent*

- → *unrolled 7 times to avoid delays*
- → *7 results in 9 cycles → 1.3 Clocks per Iteration*
- *Average: 2.5 operations per cycle (50% efficiency)*

- → *need more registers in VLIW*

- - - 

***Software Pipelining with Loop Unrolling in VLIW***

- *software pipelining across 9 iterations of original loop (3 iterations in each cycle & 3 cycles)*
- *in each iteration of loop:*
- *→ store, compute & load in parallel for different loop iterations*

- → *9 results in 9 cycles → 1 clock per iteration*
- *average: 3.67 instructions per cycle (73.3% utilisation)*

- → *need fewer register than VLIW*

- - - 

***EPIC***

- ***EPIC (IA-64) →*** *new ISA that exposes parallelism to the compiler*
- → *binary compatible across processor implementations*

- - - 

***Instruction Bundling in IA-64***

*EPIC/IA-64 → issues instructions in **instruction groups***

***instruction group →*** *a sequence of consecutive instructions with no register data dependencies*
→ *all instructions within a group could be executed in parallel - if sufficient hardware resources exist & if any dependencies through memory are preserved*
→ *group can be arbitrarily long (**compiler must explicitly indicate boundary between one instruction group & another**)*
→ *compiler places a **stop*** *between two instructions that belong to different groups*
→ *compiler telling hardware explicitly where parallel issue starts & stops*

*IA-64:*
→ *128-bit wide bundles of instructions*
→ *each bundle consists of a 5-bit template field & 3 instructions (41bits each)*
→ *template → marks where instructions in the bundle are dependent/independent & whether they can be issued in parallel with the next bundle*

- → *smaller code size the VLIW/larger than x86*

- → *benefit of VLIW machine with more flexibility (more binary compatible across machines)*

- ***;; → stop bit*** *(marks the end of a sequence of bundles that can be issued in parallel)*
- → *if not stop bit between instructions - can be issued at the same time*

- - - 

***Hardware Support for Exposing More Parallelism to Compiler***

*to help trace scheduling & software pipelining - Itanium instruction set includes several mechanisms:*
- ***register stack***
- ***predicated execution***
- ***speculative non-faulting load instructions***
- ***rotating register frame***

- - - 

***Register Stack***

→ ***general purpose registers are configured to help accelerate procedure calls using a register stack***

→ *maintaining a logical mapping between logic registers included in instructions & physical registers in machine*
→ *explicitly changing logical mapping via a function call*

- *registers 32-128 used as a register stack & each procedure is allocated a set of registers*
- → *the new register stack frame is created for a called procedure by renaming registers in hardware*
- → *a special register called the **current frame pointer (CFM)** points to the set of registers used by a given procedure*
- → *registers 0-31 are always accessible & addressed 0-31*

- *main calls function f with parameters a,b,c*
- → *a,b,c put into logical registers of main & call made*
- → *call instruction transfers control to f & f has its own logical register mapping for a,b,c → mapping to the same physical registers (such that f can see the parameters a,b,c passed from main)*
- → *same for f calling g*

- → *exposes compiler to logical to logical to physical register mappings → can be manipulated*
- → *explicit dynamic mapping can be controlled in stacks rather than individually (as done in dynamically scheduled processors with register renaming)*

- - - 

***Predication***

- *conditions can be computed & stashed in a predicate register*
- → *used to control whether ana instruction executed*
- → *almost all instructions can be predicated*

***prediction means:***
- → *compiler can move instructions across conditional branches*
- → *can pack parallel issue groups*
- → *may also eliminate some conditional branches completely*
- → *avoids branch prediction & misprediction*

- *when a branch would break a parallel issue packet → move instructions & predicate them*

*instruction sequence into two instruction packets:*

*moving instructions:*
→ *first compute the predicate*
→ *predicate C & D (executed if branch taken)*
→ *then - branch instruction to L*

→ *in the next packet - predicate E,F,H (executed if branch not taken)*
→ *& pack packet with H,I,J*

- - - 

***Load Instruction Variants (IA64)***

- *IA64 has several different mechanisms to **enable the compiler to schedule loads***
*(e.g. if a load instruction a cache miss - need to schedule other instructions → allow compiler to do this statically & explictly)*

- *ld.s → speculative non faulting*
- *ld.a → speculative advanced (checks for aliasing stores)*

- *register values marked “NaT” (not a thing)*
- → *if speculation invalid*

- *advanced load address table (ALAT)*
- → *tracks stores to addresses of advanced loads*

- - - 

***Speculative Non-Faulting Load***

*example: RAW hazard after conditional branch*
→ *unsafe*
→ *want to move load instruction earlier to allow more instructions to be used before read (otherwise stalls waiting for read)*

- → *ld.s fetches speculatively from memory*
- → *i.e. any exception (i.e invalid access) due to ld.s is suppressed*

- → *check instruction on conditional path*
- → *if ld.s r did not cause an exception (valid access to r1) then chk.s r is a NOP & safely uses r1*
- → *else takes a branch to some compensation code that explicitly loads at this point (r1 loaded speculatively was not a valid access)

- ***compiler managing consistent state at which exception should take place***

*example:*

- → *move not only load but read earlier also (useful instructions between load, use, and check)* 
- → *every register has a NaT bit indicating whether the data in the register is valid*

- → *speculatively loaded data can be consumed prior to check*
- → *speculation status is propagated with speculated data via NaT bit (if speculative instruction causes an exception → NaT bit indicates this)*
- → *any instruction that uses a speculative result also becomes speculative*
- → *chk.s checks entire dataflow sequence for exceptions* (*checking NaT bit to indicate if valid*)
- → *if not valid → execute load & use instructions accordingly*

- - - 

***Speculative Advanced Load***

→ *want to move load instruction earlier & put useful work between load & use*
→ ***but there is a store instruction that may store to same address as load***

- → *ld.a starts the monitoring of any store to the same address as the advanced load*
- → *if no aliasing has occurred since ld.a → ld.c is a NOP*
- → *if aliasing has occurred → ld.c re-loads from memory*

- *ld.a → allocate x into Advanced Load Address Table (ALAT)*
- → *when a store is executed - remove any matching entry from the ALAT*
- → *ld.c → checks if x is still in the ALAT (if still in the ALAT → load was valid → NOP)*

- - - 

***IA-64 Registers***

- *both integer & FP registers support register rotation for registers 32-128*
- ***register rotation*** → *designed to ease the task of register allocation in software pipelined loops*
- → *when combined with predication - possible to avoid the need for unrolling & for separate prologue & epilogue code for software pipelined loop*
- → *makes software pipelining usable for loops with smaller number of iterations - where overheads would traditionally negate many of the advantages*

- - - 

***Register Rotation***

*consider loop for copying data as one issue packet:*

- ***br.ctop L1*** → *rotates the general registers*

- → *value stored into r35 & read into r37 in two iterations (& two rotations) later*
- → *reading r37 reads the value that was assigned into r35*
- *→ register rotation eliminates a dependence between load & store instructions & allows the loop to execute in one cycle*

- → ***logical to physical register mapping is shifted by 1 each time the branch (br.ctop) is executed***

- - - 

***Example - Register Rotation***

- *loop instructions all in one issue packet & will be executed in a single clock cycle*

***predicate mechanism:***
- *predicate mechanism activates successive stages of the three-stage software pipeline - to fill (successively activating stage) & drain (successively deactivating stages) on start-up & loop termination*
- *when software pipeline full → all predicate bits are 1*

***br.ctop:***
- *software pipeline branch that rotates registers*
- → *logical to physical register mappings of general registers (r32, r33, r34, r35) shifted by 1 for each br.ctop execution*
- → *br.ctop rotates the predicate registers & injects a 1 into p16 to start filling pipeline one stage at a time*
- → *when loop trip count is reached → br.ctop injects 0 into p16 - disabling one stage at a time - then finally falls through*

***rotation:***
- *as registers shifted by 1 for every iteration*
-  → *add instruction adds 1 to the register loaded on the previous iteration*
- → *store instruction stores the result of the add instruction from the previous iteration*

***fill:***
- *on first iteration of loop - only predicate register p16 is 1 & only load instruction executes (loading x1 into r32)
- *add & store do not execute*
- *bt.ctop activates the next predicate register (sets to 1) & rotates general registers logical-physical mapping (shifting logical registers by 1 with respect to physical registers)*
- *physical register 32 → logical register 33*
- *33 → 34 … 38→ 39 … 39 → 32*
 - *on second iteration of loop - loads into logical register 32 (physical register 39)*
 - *adds 1 to logical register 33 (physical register 32) & result into logical register 34 (physical register 33)*
 - *load instruction does not execute (predicate 0)*
 - *br.ctop activates the final predicate register & rotates general register logical-physical mappings*
- *third iteration loads into logical r32 (physical 38)*
- *adds 1 to logical 33 (physical 39 & value loaded into r32 in previous iteration)*
- *stores logical r35 into memory (result of add instruction from previous iteration)*

***operation:***
- *all predicate bits are set to 1*
- → *all software pipeline stages are active*
- → *pipeline is full*
- → *continues execution of loop*
- → *loading in a value - adding 1 to that value an iteration later - storing that result an iteration later*

***drain:***
- *propagate 0s through the predicate registers (starting with first loop instruction)*
- *final iterations complete & software pipeline drained*

- *only one copy of the loop needed (no extra copies needed for fill & drain stages)*

- - - 

***Itanium***

***IBM Power 4:***
- → *dynamically scheduled processor*
- → *multiple pipeline stages for instruction crack & group formation*
- → *finding which instructions in a packet can be issued at the same time (no dependencies)*
- → *much more complicated pipeline required before instructions can be issued as a group*

***Itanium II:***
- → *pipeline introduces a Register Rotation Pipeline Stage*
- → *much shallower, simpler pipeline*
- → *lower branch misprediction penalty etc.*

***benchmark results:***
- → *for floating point operations (predictable memory accesses)*
- → *compiler can make good choices about how to pack instruction packets across chains of conditionals (using predicates)*
- → *itanium performs well*
- → *for integer operations (hard to predict branches & ambiguous memory accesses)*
- → *dominated by dynamically scheduled processors*
- → ***itanium (statically scheduled processor) performs well when there is more predictable memory accesses in a program & compiler can perform static analysis more easily

***static analysis:***
- *process of analysing a program without execution*

- - - 

***Summary***

*Dynamic Scheduling → HW Speculation*
*Static Scheduling → SW Speculation*

***HW vs SW Speculation:***
- *to speculate extensively → must be able to disambiguate memory references*
- → *easier in HW than SW (for code with pointers)*

***SW Speculation:***
- → *compiler-based approach*
- → *requires instruction set support*
- → *works well when control flow is predictable*
- → *complex to maintain precise exceptions at correct program point*

***HW Speculation:***
- *→ works better when control flow is unpredictable*
- → *& when HW-based branch prediction is superior to SW-based branch prediction at compile-time*
- *→ maintains precise exception model even for speculated instructions*
- → *does not require compensation of bookkeeping code (code that manage the overhead/administration required for the program rather than execution of core logic)*
- → *with dynamic scheduling does not require different code sequences to achieve good performance for implementations of an architecture (generalises well to all implementations)*

- - - 





