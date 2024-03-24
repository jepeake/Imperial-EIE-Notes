
- - - 

***Sidechannel Vulnerabilities & Spectre v1***

- - - 

***Overview***

***Side-Channels:***
→ *what can we infer about another thread by observing its effect on the system state?*
→ *through what channels?*

- *how can we trigger exposure of private data?*
- *how can we block side-channels?*

- - - 

***Exfiltration***

- *suppose we control thread A (running on core 1)*
- *thread B is encrypting a message using a secret key - executing code on core 2 that we know but do not control*

- *how can we program thread A to learn something (perhaps statistically) about B - perhaps the message?*

*suppose that thread B’s encryption algorithm is simple:*

```cpp
for(i=0;i<N;++i){
	C[i] = code[P[i]];
}
```

- → *iterates through a private message P & generates coded representation of it C*

- *how can we program thread A to learn something (perhaps statistically) about P?*

- - - 

***Prime & Probe***

*technique that detects the eviction of the attacker’s working set by the victim:*
- → *attacker first primes the (shared) cache by filling one or more sets with its own lines*
- *→ once the victim has executed, the attacker probes by timing accesses to its previously-loaded lines, to see if any were evicted*
- → *if so, the victim must have touched an address that maps to the same set*

*e.g. in example above → by observing victim’s memory accesses, could infer what accesses to the code function are being made, & hence infer P*

- - - 

***Evict & Time***

*technique that uses the targeted eviction of lines, together with the overall execution time measurement*
- → *the attack first causes the victim to run, preloading its working set (all addresses that it accesses), & establishing a baseline execution time*
- → *the attacker then evicts a line of interest (e.g. corresponding to a character in the private message), and runs the victim again*
- → *a variation in execution time indicates that the line of interest was accessed*

- - - 

***Flush & Reload***

*inverse of prime & probe → technique that relies on the existence of shared virtual memory (such as shared libraries or page deduplication), & the ability to flush by virtual addresses*

- → *the attacker first flushes a shared line of interest (by using dedicated instructions or by eviction through contention → loading enough data with carefully chosen addresses that ensure eviction of line of interest)*
- → *once the victim has executed, the attacker then reloads the evicted line by touching it, measuring the time taken*
- → *a fast reload indicates that the victim touched this line (cache hit → victim already reloaded it), while a slow reload indicates that it didn’t (cache miss → victim did not reload into cache)*

→ *can observe the accesses taken by a piece of victim code*

- - - 

***Side Channels - Shared State***

- *for a side channel to be exploited - need to identify state that is affected by execution and shared between attacker & victim*
- *if they share a single core → L1I, L1D, L2, TLB, branch predictor, physical rename registers, dispatch ports…*
- *separate cores → may share caches, interconnect…*

- *different packages (chips) → may share L3 Cache*
- *Memory Controllers may be shared*
- *Interconnects → may be shared across whole system*

- → *for side-channel attacks to work → must be some shared state between threads that are running (point of contention)*
- → *such that can trigger execution of victim code*

- - - 

***How can we trigger co-located execution of the victim?***

- *System Call (e.g. to execute code in the OS which may have access to privileged data)*
- *Release a Lock (e.g. that victim process is waiting on)*
- *SMT (simultaneous multi-threading) - threads co-scheduled on same core*
- *Call it as a Function*

*Why is calling a function interesting?*
→ *Language-Based Security (runtime system of programming language protects threads that belong to different security domains)*
→ *Victim may be an object with secret state & a public access method (e.g. simply call public access method to trigger execution of victim thread)*

- - - 

***Language-Based Security: Bounds Checking***

- *if there is an array in the program → compiler implements such that subscript i is first checked to be within the bound of the array, before computing the address & accessing the array*
- → *such that an out of bounds index cannot access the array*

- - - 

***Side-Channels in Speculative Execution***

→ *side channel attacks can exploit the side effects of speculative execution (even if the execution is later undone as the condition is unmet)*

- *in a speculatively executed machine, suppose that the bounds check is predicted satisfied*
- *such that the subsequent code is speculatively executed*
- *but i is out of bounds*

- *so p points to a victim web page’s secret s (e.g. data on page B above)*
- *can speculatively use s as an index into an array that we do have access to (B → canary array)*
- *& then use timing to determine whether the cache line on which B[s] falls has been allocated as a side-effect of speculative execution (flushing & reloading B - if B reloads quickly → cache line has been allocated as a side-effect of speculative execution)*
- *determines which allocation took place from B & therefore can infer the secret s (that was used to index B)*
- → *can do all this before the speculative execution is undone (if statement evaluates to not satisfied)*

- → *need to make sure branch predictor predicts will be in bounds (running code a few times where in bounds is satisfied) & then making branch predictor mispredict on an out of bounds index*

- ***Spectre Variant 1*

- - - 

***Spectre Variant 1***

- *declare an array1 to be accessed out of bounds (during speculative execution)*
- *declare a canary array (array2) whose cached-ness can be probed to determine which allocation took place (previously array B) & therefore determine the secret*

- *execute a victim function that includes a bound-checking conditional & accesses the canary array using data from array 1*
- *if this function is indexed out of bounds (speculatively executing the if statement) → canary array is indexed using data that is out of bounds (outside of array 1)*

- *if x is out of bounds & carefully chosen to be the index which leads to the first character of the secret (address of secret - address of array 1)*
- *array2 is accessed using the first character of the secret * 512 → hitting a specific cache line*
- *can detect which cache line hit → from which we can determine the secret*

- *flush array 2 from the cache to prepare for reloading & timing accesses, to determine which cache lines (in array2) hit → inferring the secret data (looking at the address of array 2 which is a hit & working backwards to determine the secret value that must have been used to access that element in array2)*
- *train the branch predictor to speculatively predict the if statement as true - & then when an out of bounds index is then passed to it (mispredicts)*

- *reload the cache & time accesses (fast access → corresponds to secret data)*

- → *statistical analysis to find outlier access times (find fast accesses & corresponding secret data)*

- - - 

***How bad is this?***

- *different browser tabs should not run in the same address space bounds checking not enough to protect secrets from side-channel attacks)*

- - - 

