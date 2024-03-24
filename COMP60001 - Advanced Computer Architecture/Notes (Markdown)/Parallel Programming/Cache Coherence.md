
- - - 

***Cache Coherence***

- - - 

***Parallel Programming Overview***

→ ***power considerations & multicore***

→ *shared-memory parallel programming*
	→ *dynamic load-balancing*
	→ *distributed-memory parallel programming (why harder to program but more robust performance)*

→ ***cache coherency***
	*→ **design space of snooping protocols based on broadcasting invalidations & requests***

→ ***atomic operations & locks***

→ *sequential consistency*

- - - 

***Implementing Shared Memory - Multiple Caches***

- → *suppose processor 0 loads from memory location X*
- → *X is fetched from main memory & allocated into processor 0’s cache(s)*
- → *suppose processor 1 loads from memory location X*
-  *→ X is fetched from main memory & allocated into processor 1’s cache(s) too*
- → *suppose processor 0 stores to memory location X & processor 0’s cached copy of X is updates*
- → ***processor 1 continues to use old value of X***
- → *suppose processor 2 loads memory location X → **does not not whether to get X from main memory or processor 0 or processor 1***

***issues:***
- → *how to find the latest version of the cache line?*
- → *how to know when to use cached copy & when to look for a more up-to-date version?*

- - - 

***Cache Coherency***

→ ***when multiple processors with their own caches access & modify the same memory location - cache coherence ensures that changes in one cache are propagated to others - maintaining a consistent view of memory across all caches*

***Goal (Sequential Consistency):***
→ *the result of any execution is the same as if the operations of all the processors were executed in some sequential order & the operations of each individual processor appear in this sequence in the order specified by its program*

***Advantages:***
- ***performance** → efficient cache coherence protocols enhance performance of multicore systems by reducing the latency of memory operations (if caches copies are correctly updated - do not need to go to main memory as often)*
- ***consistency** → ensures all processors have a consistent view of memory - giving correctness*

***Challenges:***
- ***complexity** → implementing efficient cache coherence protocols is complex & may incur additional hardware costs*
- ***scalability** → as the number of cores in a multicore system increases - coherence may become a significant bottleneck*

- - - 

***Updation***

***Updation:***
→ *when a store to address X occurs - **update** all remote cached copies*

***approaches:***
- → *broadcast every store to every remote cache*
- → *keep a list of which remote cache holds the cache line*
- → *keep a note of whether there are any remote cache copies of this line (indicated by a shared bit per cache line)*

***advantages:***
- → ***reduces cache misses** as other processors always have most recent data in cache*
- → *can **improve performance** when shared data is frequently read but infrequently written*

***problems:***
- → *if cache line is several words long - each update to each word in the line leads to a broadcast **(higher network load)***
- → *if a processor wants to keep data that other processors will not access - broadcasting updates will occur indefinitely even if that data is not used by other processors - making updates **inefficient***

- - - 

***Invalidation***

***Invalidation:***
→ *when a store to address X occurs - **invalidate** all remote cached copies*

***approach***
- → *when a cache line is modified by one processor - message broadcast to all other caches holding that line to invalidate their copies*
- → *subsequent attempts by other processors to read this data will result in a cache miss - forcing them to fetch the updated data from memory*
- → *after the first write to a cache line (& invalidating other copies) → can repeatedly write to that line without having to broadcast again*

***advantages:***
- → ***reduces traffic caused by write operations** as only invalidation messages are sent - not data itself*
- → ***simplifies** the consistency model as only one valid copy of any writeable data exists at any time*
- → ***often better than update*** *(unless the other processors need the data as soon as possible)*

***problems:***
- → *can lead to **increased memory traffic due to cache misses** when other processors attempt to access the invalidated data*
- → *may cause a **performance penalty in read-heavy workloads** where multiple processors frequently read the same data*

- - - 

***Updation vs Invalidation***

***Workload Dependency:***
-  ***Updation** → preferred where read operations on shared data are frequent*
-  ***Invalidation** → preferred where write operations are frequent & multiple processors do not frequently read the same data*

- - - 

***Snooping***

→ ***all snooping cache controllers monitor (snoop on) all bus transactions to determine whether they have a copy of a block of data that is being read or written by another processor (by checking tags against tags of its cache)***

***features:***
- ***shared bus architecture** → snooping protocols used in systems where all processors are connected to a common bus (used for both data transfers & coherence communication)*
- ***broadcast-based** → whenever a processor issues a memory operation - this operations is broadcasted on the shared bus*
- ***active cache management** → each cache controller actively listens to all the traffic on the bus & takes appropriate actions based on the coherence protocol (like updation or invalidation)*

***protocol:***
- ***write** → when a processor writes to a memory location - the cache controller for that processor broadcasts this information onto the bus - the other caches snoop the bus & if they have a copy of that data - they either invalidate it or update it*
- ***read** → when a processor reads from a memory location - if another cache has a modified version of the data - data can be provided directly over the bus*

***advantages:***
- ***simplicity** → snooping protocols simple & easy to implement*
- ***immediate coherence** → provide immediate coherence actions as all caches have direct access to visibility of all memory operations*
- ***effective in small systems** → efficient when bus contention is not a significant issue*

***disadvantages:***
- ***scalability** → as processor numbers increase - bus can become a bottleneck*
- ***bus traffic** → as all transactions need to be broadcasted - can lead to high bus traffic (particularly in write-intensive workloads)*

***uses:***
- → *small to medium sized multicore systems*
- → *systems where latency of coherence actions critical*
- → *systems that can tolerate bus bandwidth limitations imposed by the snooping mechanism*

- - - 

***The Berkley Protocol***

***directory-based protocol:***
→ *keeps track of the state of each block of memory & which caches contain copies of those blocks*
→ *keeps bus & cache controllers*
→ *invalidation protocol*

***states:***
*each cache line can be in one of four states (2 bits):*
- ***INVALID*** → *data in cache line never loaded or invalidated due to other processors updating*

- ***VALID** → clean, potentially shared, unowned
- → *this data is read-only & cannot be supplied to other processors
- → *unmodified & reflects the current value in main memory

- ***DIRTY** → modified, only copy, owned 
- → *processor has exclusive write access & can supply to other processors
- → *changes have not been written back to main memory (value in main memory invalid)*

- ***SHARED-DIRTY**→ modified, possibly shared, owned
- → *changes not written back to main memory & shared between processors*

***read hit:***
→ *attempts to read data in cache in VALID, DIRTY, or SHARED-DIRTY state
→ *read that data from the cache

***read miss:***
→ *attempts to read data not present in cache or in INVALID state*
→ *broadcast request on the bus*
→ *if another cache has the line in SHARED-DIRTY or DIRTY*
	→ *supplies it*
	→ *sets its lines’ state to SHARED-DIRTY*
	→ *set our copy to VALID*
→ *otherwise line comes from memory & state of line set to VALID*

***write hit:***
→ *attempts to write to data that is already present in cache*
→ *if line is DIRTY - data in the cache written to*
→ *if VALID or SHARED-DIRTY (not exclusive write access which is wanted)*
	→ *an invalidation is sent to other cached copies & local state set to DIRTY*

***write miss:***
→ *attempts to write to data that is not in the cache*
→ *depending on the write policy - cache controller either fetches the data into the cache from main memory, updates it, & marks as dirty (WB) or writes data directly to main memory (WT)*
→ *all other copies set to INVALID & line in requesting cache set to DIRTY*

→ ***states needed for ensuring all processors in the system have a consistent view of memory (reduce unnecessary memory accesses & maintain data integrity across caches)*

***advantages:***
- ***scalability** → typically more scalable than snooping protocols as they don’t rely on broadcasting over a bus (which can bottleneck large systems)*

***limitations:***
- ***complexity** → directory-based systems typically more complex to implement rather than snooping protocols*

- - - 

***Berkley Protocol - State Transitions***

- - - 

***Snooping Cache Controller in Berkley Protocol***

***cache controller:***
→ *manages state transitions & snoops bus traffic to detect actions that may affect cache lines it controls*

***transitions:***
*triggered by:*
→ ***bus** (bus invalidate, bus write miss, bus read miss)*
→ ***CPU** (read hit, head miss, write hit, write miss)*

***on every bus transaction - controller looks up the directory (cache line state) for the specified address:***
- → *if the cache controller’s processor holds the data & is DIRTY -  given a bus read miss the controller will respond by providing the data to the CPU*
- → *if the cache controller’s processor holds the data & is INVALID → another CPU will have the cache line in the SHARED-DIRTY state & the controller must request the current data from that CPU*

- - - 

***Design Space of Snooping Cache Protocols***

- - -

***Implementing Snooping Caches***

***snooping mechanism:***
- → *processors continuously monitor (snoop) the bus to detect if any transactions involve memory addresses in their cache*
- → *if there is a match with the cache tag - the cache controller with either invalidate or update the corresponding cache line*

***bus:***
- → *all processors must have access to the shared bus - used to broadcast memory addresses & data for cache coherence*
- → *each bus transaction involves checking cache tags & can lead to **contention** between bus transactions & CPU access to the cache*

***solving bus contention:***
- ***duplicate set of tags for L1 caches** → allowing checks in parallel with CPU operations*
- → *while one set of tags is used for CPU to check while performing computations - another set can be used to respond to snooping operations - thereby decoupling the CPUs access to the cache from the snooping process*

- ***use the L2 cache to filter invalidations*** → *if there is multi-level inclusion & if a block is present in the L1 cache it is also present in the L2 cache - then L2 cache can be used to handle coherence checks for data in the L1 cache*
- → *if the L2 cache does not contain a tag for a particular block - it is not necessary to check the L1 cache as the block cannot be there - reducing the number of snooping checks required at the L1 level*

***cache inclusivity:***
- → *if a block is present in L1 cache - must also be present in L2 cache*
- → *contains cache design in terms of block size & associativity as L2 must be large enough to contain all blocks that may be in L1*

- - -

