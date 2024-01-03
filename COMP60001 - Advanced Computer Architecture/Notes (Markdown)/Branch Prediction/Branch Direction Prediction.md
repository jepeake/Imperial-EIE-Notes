
- - - 

***Branch Direction Prediction***

- - - 

***Branch Prediction***

- ***→ attempt to solve control hazards that can occur in any pipelined processor***
- *→ branch instructions can occur a lot (& will arrive up to n times faster in an n-issue processor)*

<br>

***Amdahl’s Law:*** 
- → ***the overall performance improvement gained by optimising a single part of a system is limited by the fraction of time that improved part is actually used***
- → *an n-issue processor (that has been optimised to issue n instructions per cycle & has a lower CPI) will suffer a greater relative impact from control stalls*

<br>

***Dynamically Scheduled Machine:***
- *with **speculative dynamic instruction scheduling & register renaming** → can speculate many instructions at once (only limited by number of shelves on which we can put issued instructions, could be 100s)*
- *→ **predicting many branches at once** (forwarding from one speculatively-executed instruction to the next)*

- - - 

***Alternatives to Branch Prediction***

***Branch Mispredictions:***
- *→ branch mispredictions are **expensive***
- *want to **minimise the likelihood of a misprediction***

<br>

***Alternative Prediction Schemes:***
- *→ **having enough threads per core***
- → *extending the instruction set with **predication***
- → *extending the instruction set with **branch delay slots***

- - - 

***Having Enough Threads per Core***

***example:***
- *→ statically scheduled 5-stage pipelined processor with 4 threads per core*

<br>

***multithreading:***
- *fetching from 4 PCs in a round-robin fashion*
- *each thread has its own set of registers (threads independent)*
- ***multiplexing multiple threads on a single core → maximises utilisation of processor***
- *→ **provides more time to determine branch outcome without need for prediction***
- *assuming program needs to utilise multiple threads*

- *with enough multithreading → **eliminates control hazard problem***
- *but now - **performance of a single thread dominates the performance** (& single thread will not perform efficiently → only executes an instruction every 4 cycles)*

- - - -

***Predication***

- → ***avoid branch prediction by turning branches into conditionally executed instructions***

<br>

***predicate registers:***
- *processor extended with **predicate (p) registers (one-bit) which hold the outcome of conditional tests***
- ***condition is evaluated & loaded into predicate register***
- *if the predicate register has value 1 → instructions that depend on that predicate register are executed*
- *control hazard eliminated & **converted to a data hazard** (data dependence between p-register in condition & in following instruction)*
- *enables the **avoidance of conditional branches***

<br>

***branch prediction preference:***
- *branch prediction preferred when **larger loop bodies → as can jump over the loop body rather than having to run through the predicate instructions** before discovering the predicate is false*

<br>

***predication preference:***
- *predication preferred **when issuing many instructions per cycle** → as a conditional jump in a sequence of instructions will jump out of that sequence of instructions (which could be large) - leaving that sequence underutilised*
- *→ predication could **pack that sequence of instructions with predicated instructions - which would be usefully utilised if the condition is true***

- *predication preferred if it is **difficult to predict branches** → avoids frequent large misprediction penalties*
- *→ **eliminates risk of a misprediction***

- - - 

***Branch Delay Slots***

- ***→ define a branch to take place after the following instruction***
- *the following instruction has already been fetched → execute it and then take the branch*
- *→ **delaying just one instruction allows for a proper decision** & branch target address in a 5-stage pipeline (eliminating control hazards)*

<br>

***where to get instructions to fill branch delay slots?***

*can get instruction from:*
- ***before the branch instruction (useful work)***
- ***from the target address (only useful when the branch taken)***
- ***from fall through (only useful when the branch not taken)***

<br>

- *fill delay slot with instruction before the branch*
- *→ if not possible - fill with an instruction from the target address or fall through (cannot be a store instruction)*

<br>

- ***usefulness of branch delay slot → function of compiler optimisation***
- *compiler effectiveness for a single branch delay slot:*
- *→ fills about 60% of branch delay slots*
- *→ about 80% of instructions executed in branch delay slots are useful computation*
- *→ about 50% of slots are usefully filled*

<br>

***cancelling branches:***
- *to increase the utilisation of branch delay slots: **cancel branches***
- *branch delay slot instruction is executed but write-back is disabled (cancelled) if it is not supposed to be executed*
- *two variants of cancelling branch instruction: branch likely taken, & branch likely not **taken***
- ***likely taken** → speculatively executes instruction in the delay slot, & cancels if branch not taken*
- ***likely not taken** →  speculatively executes instruction in the delay slot, & cancels if branch taken*
- *→ slightly increases fraction of branch delay slots that can be filled*

<br>

***downside of delayed branches:***
- ***additional complexity*** *→ added complexity for compilers, has to ensure instructions in delay slots are safe to execute whether the branch is taken or not (exposing microarchitectural details to the compiler)*
- ***code size*** → *filling branch delay slots may require inserting NOPs - which increases the code size (so reducing CPI but increasing Instructions/Program)*
- ***reduced efficiency in some scenarios*** *→ if useful work cannot be put into branch delay slot, filled with a NOP, which reduces efficiency*
- ***forward compatibility issues*** *→ if an architecture with branch delay slots evolves in future versions (extended pipeline/adds multiple instructions per clock cycle) → may lead to compatibility issues*

- - - 

***Direct & Indirect Branches***

***branch predictor:***
- → ***want to fetch the correct (predicted) next instruction without any stalls***
- *→ need the prediction before the preceding instruction (branch) has been decoded*

<br>

*two kinds of branch prediction:*
- ***direction prediction** (is a conditional branch taken or not)*
- ***target prediction** (what is the target address of the branch instruction)*

<br>

- ***target prediction needed for indirect branches** (e.g. indirect branch to a target address stored in a register) & determining target of conditional branch once predicted taken*
- *most common cases of indirect branch → return addresses (take return address from the stack & indirect branch to that address)*

<br>

- ***direct branch** → provides the address of the target instruction directly as part of the branch instruction*
- ***indirect branch** → target address is not specified directly within the branch instruction → instruction specifies a register or memory location that contains the target address*

- - - 

***Branch Prediction Schemes***

***Takenness (Direction):***
- *1-bit Branch History Table (1-bit BHT)*
- *2-bit Branch History Table (2-bit BHT)*
- *Correlating Branch History Table (Correlating BHT)*
- *Tournament Branch Predictor*

***Target**:*
- *Branch Target Buffer (BTB)*
- *Return Address Predictors (RAP)*

- - - 

***Branch History Table (BHT)***

***1-bit BHT:***

- ***lower bits of PC address index a table of 1-bit values***
- *says whether or not the branch was taken last time*
- *no address check (saves HW - but as we are only using k low-order bits of PC → may be a different branch instruction)*

- ***when a branch instruction is executed** → update the table value with the corresponding PC index to 0/1 depending on whether the branch was taken or not taken*
- ***when we encounter that branch again** → index the branch history table, & predict based upon whether the branch was taken/not taken last time*

<br>

***problems:***

***aliasing:***
- *aliasing → possible mispredictions if 2 different branch instructions map to the same Branch History Table Entry*

***loops:***
- *in a loop → a 1-bit BHT will cause 2 mispredictions*

- *on the first pass through the loop, Blt R1,L1 will be updated as taken in the BHT → and will be repeatedly taken for iterations of L1*
- *on L1 exit, when the loop falls through, the branch will not be taken (and mispredicted as taken), & the BHT entry will be updated to not taken*
- *then, on the second pass through L2, Blt R1,L1 now has not taken in the BHT, so prediction will be incorrect (a second misprediction)*
- *→ only 80% accuracy even if the loop’s branch is taken 90% of the time*

- - - 

***Dynamic Branch Prediction***

→ ***2-bit scheme where change prediction only if get misprediction twice***

<br>

- ***2-bits → encoding a number 0-3 (4 states)***
- *when we encounter a **not taken branch → decrement the counter***
- *when we encounter a **taken branch → increment the counter***
- ***state 0,1 → predict taken***
- ***state 2,3 → predict not taken***

<br>

***loops:***
- *→ fixes the loop case for a 1-bit BHT*
- ***→ works well for loop intensive applications***

- - - -

***2-bit Branch History Table***

- *when we encounter a branch → **update the state in the BHT for that branch index***
- ***incrementing if taken & decrementing if not taken***

- - - 

***BHT Benchmarks***

→ ***does increasing the number of BHT bits improve performance?***

- *2-bit predictor → often very good - sometimes awful*
- *1-bit usually worse*
- *3-bit not usefully better*

<br>

**→ *does making the BHT capacity greater improve performance?***

- ***increasing the BHT capacity → more entries in the BHT table → prevents aliasing***
- *any program with many branches should benefit from increased BHT capacity*
- *→ however, **little evidence that BHT capacity is an issue** (4096 vs unlimited entries → little performance difference)*

- - - 

***Saturating Counter***

- ***n-bit BHT predictor essentially based on a n-bit saturating (no wraparound) counter → taken increments, not-taken decrements***
- *predict taken if most significant bit is set*

<br>

***bimodal:***
- *most branches are highly biased → either almost always taken - or almost always not taken*
- *works badly for branches which are not biased*

- *often called **bimodal predictor***
- *→ **exploits highly biased property***

- - - 

***Bias***

- ***bimodal distribution***

- *most branches are untaken < 1% of the time or > 99% of the time*
- → ***exploited by saturating counter***

- - - 

***Local History***

- → ***the bimodal predictor uses the BHT to record local history***
- ***local history** → prediction information used to predict a particular branch is determined only by its memory address*

*consider:*

```
if(C1) then
	S1;
endif
if(C2) then
	S2;
endif
if(C3) then
	S3;
endif
```

- *very likely that that condition C2 is correlated with C1 & C3 is correlated with C1 & C2*
- *can we use correlation in a better branch predictor? → global history*

- - - 

***Global History → Correlated Branch History Table***

***global history**: the taken/not-taken history for all previously executed branches*

- *idea: **use global history to improve branch prediction***
- *compromise: use m most recently-executed branches*
- *implementation: keep an m-bit branch history register (BHR)*
- ***BHR → a shift register recording taken/non-taken direction of the last m branches***
- *when we encounter a branch, discover taken/not-taken, & push a 0/1 into the shift register*

- - - 

***gselect Branch History Table***

→ ***type of correlated BHT***

- ***use global history to select from a number of branch history tables***
- *→ according to the behaviour of the last m branches (exploits correlation between branches)*

<br>

***several BHTs:***
- *several BHTs **indexed by the same PC k low-order bits***
- ***BHT selected using the Branch History Register***
- *Branch History Register → contains the global history*

<br>

***updating entries:***
- *when a branch instruction has been completed → you know **whether the branch was taken/not taken & the branch history***
- *use the **branch history to select which table to use***
- *& **use the PC of that branch to select which entry to use***
- ***update that entry with the takenness of the new branch***

<br>

- *different histories lead to the updation of different BHTs for that branch’s PC index*
- *→ e**ach distinct m-bit branch history (in this case 2-bit) updates a different BHT (in this case 4)***

***predicting:***
- *when there is a new branch to predict*
- ***select index based on PC low-order bits***
- ***look at branch history to select the BHT***
- *& predict based on that BHT entry*

- ***gselect** → concatenates PC address bits & branch history register bits, to exploit the correlation between successive branches, & determine where prediction will be stored & found*

- - - 

***How many bits of branch history should be used?***

***branch prediction accuracy vs number of global bits used for indexing:***

<br>

- *code assumed to contain lots of correlated branches*
- *& for a 4k entry branch predictor*
- *keep the total capacity constant, but vary how many address bits come from the branch history register vs PC (i.e vary the number of BHTs vs number of entries in each BHT)*
- *4k entries → 12bits available*
- *if 2 bits used for global history, 10 bits available for PC in indexing*
- ***optimum choice: 6 bits for branch history & 6 bits from PC***

- - - 

***Variations to Global History (Correlated BHTs)***

- ***gselect** → many combinations of n & m*
- ***global** → use only the global history to index the BHT, ignore the PC of the branch being predicted*
- ***gshare** → arrange bimodal predictors in a single BHT, but construt its index by XORing (rather than concatenation) low-order PC address bits with global branch history shift register → claimed to reduce conflicts*
- ***Per-address Two-level Adaptive using Per-address pattern history (PAp)** → for each branch, keep a k-bit shift register recording its history, & use this to index a BHF for this branch*

- *this optimum choice is **program dependent***

- - - 

***Re-evaluating Correlation***

- *several of the SPEC benchmarks have less than a dozen branches responsible for 90% of taken branches*
- *→ may not represent real programs + OS (more like gcc & much larger) → more sensitive to branch predictor capacity*
- *→ perhaps real program benefit less from global history*
- *→ **benchmarks do not test branch predictor capacity***

- ***must be careful about appropriateness of benchmarks (particularly outlier benchmarks)***

- - - 

***Tournament Predictors***

- ***uses 2 predictors***
- *→ **one based on global information***
- *→ **another based on local information***

- *combined with a **selector** (which is driven by a predictor to predict the correct predictor to use)
- *→ hopes to select the right predictor for the right branch*

- - - 

***Summary - Branch Direction Prediction***

***Prediction Alternatives:***
- → ***Fine Grained Multithreading with Enough Threads***
- → ***Predicated Execution***
- → ***Branch Delay Slots***

→ ***Branch Takenness & Branch Target***

***Takenness:***
- → ***BHT:*** *2-bits for Loop Accuracy*
- ***→ saturating counter (bimodal) scheme** → handles highly-biased branches well***
- → ***correlating BHTs (global history) →** assumes recently executed branches correlated with next branch*
- → ***tournament predictor →** tries two or more competitive solutions & pick between them*

- - - 


