
- - - 

***Branch Target Prediction***

- - - 

***Direct & Indirect Branches***

***branch predictor:***
- → ***want to fetch the correct (predicted) next instruction without any stalls***
- *→ need the prediction before the preceding instruction (branch) has been decoded*

*two kinds of branch prediction:*
- ***direction prediction** (is a conditional branch taken or not)*
- ***target prediction** (what is the target address of the branch instruction)*

- ***target prediction needed for indirect branches** (e.g. indirect branch to a target address stored in a register) & determining target of conditional branch once predicted taken*
- *most common cases of indirect branch → return addresses (take return address from the stack & indirect branch to that address)*
- → *indirect jumps*
- → *indirect function calls*

- ***direct branch** → provides the address of the target instruction directly as part of the branch instruction*
- ***indirect branch** → target address is not specified directly within the branch instruction → instruction specifies a register or memory location that contains the target address

- - - 

***Branch Target Buffer (BTB)***

- → ***given the current PC - tells you what the next PC should be (while fetching current branch instruction → predicting what next instruction should be)***
- → *in the instruction fetch stage of the processor (in parallel with instruction fetch)*

![[Pasted image 20240103162351.png|400]]

***predicted PC:***
- *→ the predicted PC is the **address to which the branch instruction transferred control to the last time it was taken***
- *if the processor predicts that the branch will be taken → it will start fetching instructions from this predicted target address*
- *if the processor predicted the branch will be not taken → it will increment the PC normally*

***index & tag:***
- → *BTB indexed with **low-order PC address bits***
- → *BTB tagged with **high-order PC address bits** (hit?)*

***cache of branch target addresses:***
- → *BTB acts as a cache of branch target addresses accessed in parallel with I-cache in the fetch stage*
- → *updated only by taken branches*
- → *direction predictor determines if BTB uses*

![[Pasted image 20240103163038.png|400]]

- - - 

***5-Stage Pipeline***

![[Pasted image 20231101195252.png|400]]

- *→ extend with BTB*

![[Pasted image 20231101195309.png|400]]

- ***BTB in parallel with instruction fetch from I-cache***
- *→ fetches next PC prediction*
- → *if tag matches (hit), can override PC+4*

- ***must later check if prediction was correct in decode stage***
- *→ if misprediction → override the PC for current fetch & disable the MEM & WB for mispredicted instruction (mispredicted instruction cannot update memory/registers)*
- → *go back & fetch correct branch target*
- → *branch misprediction penalty (only 1 instruction in simple 5-stage pipeline)*

- - - 

***Combining BTB with Direction Prediction***

- *for conditional branches*

![[Pasted image 20231101195636.png|400]]

- ***simultaneously check BTB & determine predicted takenness of branch***
- *→ if BTB hit → target address = predicted PC of conditional branch*

- ***in parallel → determine takenness prediction***
- → *if taken → use target from BTB*
- → *otherwise, increment PC normally*

![[Pasted image 20231101200046.png|400]]

- ***BTB & direction predictor accessed in parallel to instruction cache → in IF stage***
- *if predicted taken → updates next PC to BTB predicted PC*

- *→ **bigger slower predictor** can also be used in combination (has much better prediction performance) with a small, faster predictor*
- → *if slower prediction (e.g. tournament predictor) differs → update prediction target & disable MEM & WB for first prediction*

- *can get an improved prediction on the 2nd cycle → more important for more complex (dynamically scheduled) designs*
- → *lose only one cycle if new prediction differs, rather than full misprediction penalty*

- - - 

***when should the branch prediction (BTB) be updated in the dynamically scheduled processor?***

![[Pasted image 20231101202213.png|400]]

- *when the branch outcome is known?*
- *when the branch is committed?*

- *when branch outcome is known → updates prediction as quickly as possible*
- *→ if iterating over a loop, all instructions in the loop may be speculatively executed & not yet committed → need a good prediction for those speculatively executed instructions
- *but → if discovers a misprediction → misupdated the branch prediction (leads to a further misprediction, as branch predictor state is wrong)*

- → ***update the BTB when a branch is committed***

- - - 

***Return Addresses***

***Return Address → location directly after a subroutine (JSR) is called***

- *type of indirect jump (target → point in program to return to)*
- *target specified in memory on the stack (or in a special register on MIPS)* 
- → *should be somewhat predictable*

![[Pasted image 20231101202632.png|400]]

- *returns the to next instruction after the first JSR*
- *this is set to the predicted PC of the return instruction in the BTB*
- *next time returns from the function (when called from somewhere else) → will return to the instruction after the first call, based on the value in the BTB → incorrect target for new call*

![[Pasted image 20231101203141.png|400]]

*deep call stack:*

![[Pasted image 20231101203331.png|400]]

- ***stack → should be easy to predict***
- → ***add a Return Address Predictor - containing a stack of the return addresses***

- - - 

***Return Address Predictor***

- *extend branch address predictor → **+ return address predictor***

![[Pasted image 20231101203712.png|400]]

- *when a JSR instruction is decoded → **pushes the return address (address of the instruction immediately following the call) onto the RAP stack***

- ***the stack in the RAP keeps track of the return addresses for JSR instructions***
- → *the stack is updated on every JSR & return instruction*
- → *when a JSR instruction is decoded → the return address is pushed onto the stack*
- → *when a return instruction is decoded → the address at the top of the stack is popped*
- → *popping & pushing from the RAP is done after the decode stage, when mispredictions are detected (so misprediction does not impact the RAP stack)*

- ***when the fetch stage encounters a return instruction → uses the RAP to predict the return address (rather than the BTB)***
- → *does this by indexing the BTB with the current PC*
- → *if the current PC corresponds to a predicted PC which is a return address→ address at top of RAP used as the predicted PC rather than the value in the BTB

- *→ if the return prediction is incorrect (misprediction)  → the incorrect speculative state is flushed from the pipeline (MEM & WB disabled) & the correct PC is fetched*
- → *in general, less costly for a return instruction due to the high accuracy of return address predictions due to the stack like nature of calls & returns*

- - - 

***how could the RAP mispredict a branch target?***

![[Pasted image 20231101204125.png|400]]

- → ***call stack may be deeper than the RAP stack→ & returns empty***

- *RAP prediction may be wrong*
- → ***return address on call stack may be overwritten **(RAP no longer matches call stack)*
- → ***stack pointer may be changed** (e.g. switched to another thread with a different stack pointer)*

- - - 

***when should the branch prediction be updated in the dynamically scheduled processor?***

![[Pasted image 20231102204231.png|400]]

- → *BTB updated when a branch is committed*

- *→ if we wait for a branch to be committed to the BTB before updating the RAP stack → may lack a prediction for the return from a function*

*example:*
- *IF(C) is a conditional branch, followed by JSR H*
- → *assume we wait for the branch to be committed to the BTB before updating the RAP stack*
- → *if branch is predicted taken → JSR H will speculatively execute (not yet updating the RAP stack)*
- → *H may return before the branch has been committed → but there is no prediction in the RAP for the return instruction to predict the return address*

- *or → if we are not waiting for branch to be committed (updating BTB as soon as branch outcome in known) → may mispredict the conditional branch IF(C)*
- → *JSR speculatively executing & updates the RAP stack*
- → *branch misprediction → RAP stack contains an incorrect return address → which will be used on the next return*

- - - 

***Branch Prediction & Multi-Issue***

- *in many real-world processors → multiple instructions are fetched, issued, & dispatched per cycle*
- *- may encounter two or more branches in a packet of instructions*

- *→ **all the BTB needs to predict is the next instruction to fetch → does not matter which branch is responsible***

- → *commonly, **a bigger slower branch predictor may later resteer the processor if it has a better prediction that should over-ride the BTB***

- - - 

***Summary***

![[Pasted image 20231102213722.png|400]]

- - - 

