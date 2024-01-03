
- - - 

***Meltdown & Spectre v2***

- - - 

***Spectre v1***

![[Pasted image 20231115202759.png|600]]

- *declare a valid array for the victim to access & a canary array whose cached-ness can be probed (to determine which allocations of canary made → & infer secret)*

- *access the valid array with an out of bounds index (speculatively executing & causing a misprediction in which the bound checking predicted to be satisfied - but is not)*

- *if an out of bounds index is selected that corresponds to the address of a character in the secret (e.g. secret - array1 → first character of secret) & the element accessed out of bounds from valid array will correspond to a character of the secret*

- *this address can then be used in the accessing of canary array

- *flushing canary array from the cache & then probing & timing accesses determines which cache lines corresponding to canary array be have been accessed by victim (fast reload times)

- *identifying fast access times statistically → can determine which elements of canary array accessed by victim → which can then be used to determine secret inferred from addresses accessed

***example with dynamically scheduled CPU with RUU:***

![[Pasted image 20231117195031.png|600]]

- *fetch & issue the victim code*
- *victim code:*
- → *speculates that p is in bounds (attacker trained branch predictor)*
- → *uses p (an address chosen specifically to access secret) to access & load secret into s*
- → *uses s to calculate address & access canary array (canary array → previously flushed by attacker)*
- → *attacker can then probe & time cache to find addresses in canary array that have been loaded & infer secret*

- *victim code instructions issued & present in RUU*

- *suppose branch instruction at head of RUU & waiting for commit*
- *while waiting:*
- → *load unit fetches s*
- → *load unit fetches r*
- → *resulting in a cache line in canary array allocated*

- *when branch instruction committed → misprediction*
- *RUU & issue-side registers flushed → but cache line allocated remains*
- → *attacker can still probe & time cache to determine secret*

→ ***most modern processors vulnerable to Spectre Variant 1***
→ ***spectre defeats all language-enforced confidentiality with no known comprehensive software mitigations***
→ ***different browser tabs cannot sun in the same address space*

- - - 

***Mapping the Kernel to the Virtual Address Space (Meltdown)***

*blue pages → user-mode mapping (virtual address space)*
*green pages → supervisor-mode mapping (mapping of the OS Kernel)*

![[Pasted image 20231117201557.png|300]]

- *performance optimisation → map the OS Kernel into every process’s virtual address space*
- *tagged as supervisor-mode access*
- *when an interrupt or system call occurs → no change to address mapping → just flip supervisor bit*

- *speculative accesses can be made to addresses in the kernel’s memory*
-  → ***spectre attack can be used to access all of the kernel’s data*

- *commonly the kernel’s virtual address space contains mapping to all of the physical memory*
- → *an attack can capture secrets from all other user processes*

***why is the invalid access to the secret data (misprediction) only detected at commit time?***

![[Pasted image 20231117202547.png|400]]

- *load unit initiates load from L1D cache (speculatively)*
- *indexing the L1D data & tag*
- *(assuming virtually indexed, physically tagged VIPT cache)*

- *looks up virtual page number in DTLB*
- *if tag matches translation → data is forwarded to the common data bus*
- *if the tag match fails → L1D cache miss → initiates L2 access*
- *index L2 with physical address & allocate cache line into L1D

- *DTLB contains metadata indicating whether address translation valid for current processor state (in user-mode are we allowed to access this particular address translation)*
- *this data passed to access validity check to determine validity of load/store* 
- *validity of load/store is then passed to corresponding RUU entry (marking as valid/invalid)*
- *commit unit checks that load/store was valid when it reaches the head of the RUU*
- ***only checked at load/store commit time***
- *may be invalid → load cannot be committed*

→ *assumes that the microarchitectural state is not observable*
→ *assumes only commit side state can be observed & checking at commit time is safe*
→ *but with canary array + flush & probe attackers can observe changes in state*

- - - 

***Address Space Randomisation***

- *modern OSs randomise the address mapping (on every boot)*

- *user-mode address-space layout randomisation common to mitigate attacks*

- *all modern OSs now also implement kernel address-space layout randomisation 
- *(attacker must guess where in user-mode mappings the supervisor-mode mappings are)*

- → *exploiting meltdown more difficult (but still possible)

- - - 

***Kernel Address Space Isolation (KPTI)***

*further mitigation:*

- *change the virtual address mapping every time kernel is entered*
- *(everytime there is a context switch between a process in user-mode & the OS in supervisor-mode)*
- → *reloading the TLB (corresponding to the mapping of the OS kernel)*

- *slightly improved using address space identifiers*

- *significantly improves mitigation against spectre side-channel attack reading OS memory*
- *substantial performance penalty for some applications (2-30% slowdown)*
- → *more substantial for applications with more context switches*

- - - -

***How can the Kernel be read with KPTI Mitigation?***

- *how can we access the data that really is in a different address space?*
- → *trick the victim into accessing the wanted data (creating an observable microarchitectural state change → side channel)*

- *suppose the OS kernel includes:*
![[Pasted image 20231117214849.png|250]]
- → *called a gadget*
- → *suppose p points to secret & we know address of B*

- → *need to persuade the kernel to jump to label (+ attacker can construct a side channel)*
- → *train branch predictor to mispredict a branch target as label*

- → *cannot access B directly (part of the victim’s address space) → but can access data that conflicts with B in the cache)*
- → *if enough allocations into cache → can displace B array from cache (flush)*
- *& then measure access times those conflicting addresses to infer if B has been reloaded (those addresses will load slow if replaced by B by victim & fast if not replaced by B)*

***tricking the branch predictor (BTB):***

- *system call in an OS → invoked with sysenter instruction*
- *a register can be set to hold the id of the particular system call*

![[Pasted image 20231117215519.png|400]]

- *the kernel is entered at a standard entry address*
- *looks up the system call handler in a table according to the id of the system call*
- *that handler is called*
- *i.e. calling of the handler→ indirect function call*
- → ***predicted by the BTB*

![[Pasted image 20231117215531.png|400]]

- → *perhaps the BTB can be primed to jump to the gadget code*

- - - 

***Spectre Variant 2***

- *finds a gadget in victim’s code space (or puts it there)*
- *trains branch predictor so that it will cause a speculative branch to the gadget when the system call is executed*
- *observes a microarchitectural/cache side channel from the speculatively executed gadget (e.g. using Spectre v1)*
- *steals secret*

- - - 

***Mitigating Spectre v2***

- ***block microarchitecture side-channels & cache side-channels***
- → *changing microarchitecture configuration*
- → *(difficult to ensure all side-channels blocked)*

- ***prevent attacker from cache probing***
- → *adding noise to timers & prevent attacker from receiving accurate timing after probing*
- → *(attacker may overcome this noise)*

- ***prevent the attacker from tricking the branch predictor***
- *→ add an instruction to block use of branch prediction*
- *→ find places where blocking instruction should be used (i.e. all places where language based security needed → difficult) (could be placed at kernel-entry code)
- → *(performance cost due to no branch prediction)*

- ***block branch predictor contention***
- → *maintain separate predictions for each thread in each protection domain*
- *→ prevents attacker tricking the branch predictor*

- - - 

***Retpolines***

- → *Spectre Mitigation*

- *branch prediction + return address stack*
- → *used to prevent tricking branch predictor*

***retpoline:*** *implements an indirect branch using a return instruction*
→ *fixes return address stack to ensure a fixed branch prediction target*
→ *ensures safe control transfer to a target address by performing a function call, modifying the return address, & then returning*
→ ***attacker cannot modify prediction target***

![[Pasted image 20231121201835.png|600]]

→ *RP2 called (jump to RP2) & address of RP1 pushed to return address stack*
→ *overwrite top of the return address stack with desired jump target*
→ *return*
→ *returns to desired target*

*(RAS Predictor predicts that will return to caller → address RP1 → misprediction → will actually return to desired jump target)*

- - - 

***Spectre v1 vs Meltdown vs Spectre v2***

***Spectre v1:***
→ *defeats bounds checking within a single process’s virtual address space*
→ *uses branch takenness misprediction to expose a sidechannel due to speculative instruction execution*
→ *no good mitigation*

***Meltdown***
→ *extends spectre v1 to access any data currently in the process’s virtual address space → even if marked supervisor only*
→ *exploits a common defect in processor design → where the check for sufficient privilege is performed at commit time*
→ *works as OSs would try to avoid having to change the page table when handling a system call or interrupt*
→ *removing this optimisation avoids the problem*

***Spectre v2:***
→ *uses jump target prediction to choose which code is executed in the address space of another process*
→ *by choosing (or more likely finding a way to insert) suitable code → a speculative execution sidechannel can be exploited so the attacker can read the OSs data*
→ *this can be mitigated by preventing the attacker from being able to influence the victim’s branch target prediction (e.g. retpolines)*

- - - 





